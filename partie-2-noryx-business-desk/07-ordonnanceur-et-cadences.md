# 7. Ordonnanceur et cadences

## Le principe

Un **timer systemd utilisateur** lance `desk scheduler tick` **toutes les 5 minutes** (`OnCalendar=*:0/5`,
`Persistent=true`). Rien ne tourne entre deux passages. À chaque passage, l'ordonnanceur exécute les tâches **dues**.

Garanties :

- un verrou de fichier empêche deux passages simultanés ;
- chaque tâche a un verrou en base avec expiration (protection contre les doubles exécutions, y compris un lancement
  manuel concurrent) ; une exécution orpheline est marquée « interrompue » ;
- une tâche en échec est retentée (attente de 10 min × n° de tentative) jusqu'à son nombre maximal de tentatives,
  puis un incident est enregistré ;
- une tâche IA « non exécutée » (quota, plage horaire, délai minimal) est retentée au plus tôt 30 min plus tard ;
- **pause globale** : aucune tâche automatique.

## Les tâches

| Tâche | Rôle | Intervalle par défaut |
|---|---|---|
| `inbox_fetch` | Relève IMAP en lecture seule | 15 min |
| `followup_scan` | Détection des relances et des actions manquantes | 4 h |
| `failure_check` | Contrôle des erreurs et envois bloqués | 1 h |
| `backup` | Sauvegarde de la base (14 conservées) | 1 jour |
| `daily_report` | Rapport commercial quotidien | 1 jour |
| `pipeline_summary` | Synthèse du pipeline | 1 jour |
| `data_check` | Fraîcheur des données (revérification à 90 j), échéances RGPD | 1 jour |
| `retention_check` | Durées de conservation (36 mois) | 1 semaine |
| `ai_inbox_triage` | Tri des e-mails reçus | 30 min |
| `ai_first_contact` | Premiers messages | selon cadence |
| `ai_followup_drafts` | Relances | 1 jour |
| `ai_qualification` | Qualification | selon cadence |
| `ai_estimates`, `ai_estimate_review` | Estimation et contre-expertise | 30 min |
| `ai_quote_drafts` | Devis brouillons | 1 jour |
| `ai_daily_brief` | Synthèse du Sales Manager | 1 jour |
| `ai_prospect_research` | Recherche de prospects | selon cadence |

Une tâche IA dont l'agent n'a **rien à traiter** se termine sans appeler Claude (« rien à traiter : Claude n'a pas
été appelé ») : aucune consommation.

## Les cadences

Le script `scripts/cadence.py` applique un profil aux réglages de l'IA et de trois tâches clés. Deux timers le lancent
automatiquement : **8 h → normale**, **22 h → nuit**. Le mode **intense** se déclenche à la demande.

| Cadence | Plage IA | Délai entre deux exécutions | Prospection | Qualification | Premiers messages | Pause après quota |
|---|---|---|---|---|---|---|
| ☀️ normale | 08 h – 22 h | 20 min | 2 h | 1 h | 1 h | 3 h |
| 🌙 nuit | 24 h/24 | 5 min | 20 min | 15 min | 15 min | 30 min |
| ⚡ intense | 24 h/24 | 5 min | 20 min | 15 min | 15 min | 30 min |

- Le plafond reste de 120 exécutions par jour.
- La cadence **réactive** ces trois tâches et redéfinit leur intervalle : un intervalle changé à la main est
  remplacé au prochain changement de cadence.
- Au passage en cadence normale, un **compte rendu de la nuit** est enregistré (prospects trouvés, e-mails préparés,
  exécutions, échecs, quotas atteints) et publié dans #assistant.
- Bascule manuelle : bouton du panneau Discord, commande `/cadence`, demande à l'assistant, ou
  `python scripts/cadence.py nuit|intense|normale`.

## Priorité entre les agents

Le délai minimal entre deux exécutions IA (5 min la nuit) et la fréquence du timer (5 min) font qu'**un seul agent
peut réellement tourner par passage**. L'ordre de passage décide donc de qui travaille.

Problème constaté : les tâches étaient parcourues dans leur ordre de déclaration, la prospection en premier. Elle
prenait le créneau ; l'agent de rédaction, plus loin dans la liste, tombait toujours sur « intervalle minimal », était
reporté de 15 min, et la prospection reprenait le créneau suivant. Résultat : 31 fiches « À contacter » et aucun
premier message rédigé.

Correction (prête dans le code, voir [journal](15-journal-des-evolutions.md)) :

```python
# Au plus une exécution IA par passage : l'aval du pipeline d'abord, pour traiter ce qui existe avant d'en produire
# davantage. Une tâche sans travail ne consomme pas le créneau.
AI_PRIORITY = ["ai_inbox_triage", "ai_first_contact", "ai_followup_drafts", "ai_qualification", "ai_estimates",
               "ai_estimate_review", "ai_quote_drafts", "ai_daily_brief", "ai_prospect_research"]
```

- Les tâches locales passent d'abord, puis les tâches IA dans cet ordre.
- Dès qu'une tâche IA a réellement appelé Claude (ou a échoué), les autres tâches IA attendent le passage suivant,
  sans être marquées « non exécutées ».
- Une tâche IA sans travail ne prend pas le créneau.

## Commandes

```bash
./bin/desk scheduler status          # dernière exécution de chaque tâche
./bin/desk scheduler tick            # un passage
./bin/desk scheduler run <tâche>     # une tâche, tout de suite (les contrôles de l'IA s'appliquent)
systemctl --user list-timers | grep noryx
journalctl --user -u noryx-desk-scheduler.service -n 100
```
