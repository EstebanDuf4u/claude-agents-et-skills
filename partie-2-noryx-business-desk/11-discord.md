# 11. Discord : panneau de contrôle et assistant

Le bot Discord est le **poste de pilotage à distance** du Desk. Il tourne comme un service systemd
(`desk discord run`), avec une connexion **sortante** vers Discord : aucun port n'est ouvert. Toute la logique est dans
`desk/services/notify.py`, `panel.py` et `assistant.py`, testable sans Discord ; `desk/discord_bot.py` ne fait que
relier Discord à cette logique.

## Les garanties

- Seuls les **identifiants Discord autorisés** (réglages) peuvent agir. Chaque action est faite au nom du compte du
  Desk associé au bot, avec son rôle, et journalisée « via Discord ».
- Les salons sont dans une catégorie **privée** : invisibles pour les autres membres du serveur.
- Un bouton d'approbation porte l'**empreinte** du contenu publié : si le message a changé, l'action est refusée.
- **Rien n'est publié rétroactivement** : seuls les éléments apparus après la mise en service sont annoncés.
- Le bot crée lui-même la catégorie et les salons au démarrage, et synchronise les permissions.

## Les salons

| Salon | Contenu |
|---|---|
| **#tableau-de-bord** | Le panneau de contrôle, tenu à jour (voir ci-dessous) |
| #validation | Chaque e-mail à valider, en entier, avec **Approuver et envoyer**, **Approuver seulement**, **Annuler** |
| #prospects | Chaque nouveau prospect (lieu, secteur, site, score, contact) et le compte rendu 🔎 de chaque recherche |
| #boite-de-reception | Demandes de projet détectées dans la boîte mail |
| #estimations | Charge, prix calculé, fourchette |
| #rapports | Rapport quotidien |
| #alertes | Incidents (quota, connexion, tâche en échec) |
| #assistant | Conversation ; 🎯 chaque entreprise trouvée ; 🌅 compte rendu de la nuit à 8 h |
| #claude-code | Comptes rendus des sessions de développement Claude Code |

## Le panneau de contrôle (#tableau-de-bord)

Quatre messages **réédités sur place**, jamais de nouveaux messages. Les heures utilisent la syntaxe
`<t:horodatage:R>` de Discord : chaque client les affiche dans son fuseau et les met à jour lui-même (« il y a
5 minutes »). Le bot ne réédite donc un message que lorsque son contenu change réellement.

1. **🧭 État général**
   - automatisation en marche ou en pause, cadence (☀️ / ⚡ / 🌙), plage horaire de l'IA ;
   - exécutions IA du jour sur le plafond, **prochaine exécution IA possible** et ce qui la retarde (quota du jour,
     délai entre deux exécutions, pause après quota, plage horaire) ;
   - état de l'envoi des e-mails (actif, ou les raisons), envois du jour ; mode simulation ;
   - tous les délais : entre deux exécutions IA, pause après quota, relances (J+7, J+5/J+12), nombre maximal de
     relances, délai entre deux contacts, seuil d'inactivité.
   - Boutons : ⏸️ tout mettre en pause / ▶️ reprendre, ☀️ normale, ⚡ intense, 🌙 nuit, 🔄 actualiser.
2. **🤖 Agents IA** (9) et 3. **⚙️ Tâches automatiques** (8) : pour chacun, un état (🟢 actif, ⚪ inactif, 🔄 en cours,
   ⏸️ en pause, 🔴 dernier passage en échec), la fréquence, le travail en attente, le dernier passage (✅ ❌ ⚠️ ⏭️) et
   le **prochain passage estimé**.
   - Menu « Piloter un agent… » : ouvre une **fiche privée** (visible de vous seul) avec l'historique des 6 derniers
     passages, **Activer / Désactiver**, **Lancer maintenant** et le choix de l'intervalle (15 min à 1 semaine). La
     fiche se met à jour sur place après chaque action.
4. **📈 Activité** : pipeline par étape, e-mails à valider, tâches du jour et en retard, devis en attente, incidents
   ouverts, dernières exécutions IA.

### Le calcul du prochain passage

Il reproduit les règles réelles de l'ordonnanceur et de l'exécuteur :

```
dû = dernier passage + intervalle      (10 min × tentative après un échec ; 30 min au plus après « non exécuté »)
si agent IA : dû = max(dû, première heure permise)
    où première heure permise = max(maintenant, demain si quota du jour atteint,
                                    dernière exécution + délai minimal, incident quota + pause)
    puis décalée au début de la plage horaire si besoin
si rien à traiter : « rien à traiter (vérifié toutes les X) »
si dû ≤ maintenant + 5 min : « au prochain passage »
```

### Ce qui se pilote, et ce qui ne se pilote pas

| Depuis Discord | Seulement dans le Desk |
|---|---|
| Pause / reprise, cadence | Activer l'envoi des e-mails, mode simulation |
| Activer, désactiver, lancer un agent ou une tâche, changer son intervalle | Délais de relance, limites d'envoi |
| Approuver, envoyer, annuler un e-mail (#validation) | Prix, offres, comptes, conformité |

Toute modification est réservée au rôle **admin**. À 8 h et 22 h, la cadence redéfinit l'intervalle de la prospection,
de la qualification et des premiers messages, et les réactive : la fiche de ces agents le signale.

## L'assistant (#assistant)

On lui écrit librement : « fais-moi le point », « qu'ont trouvé les agents cette nuit ? », « quels agents sont
actifs ? », « lance une recherche sur les restaurants de Dax », « passe en cadence normale ».

- La réponse est rédigée par Claude via `claude -p` sur l'abonnement, **sans aucun outil**, à partir d'un
  **instantané** du Desk : état, agents et tâches (actif, intervalle, dernier et prochain passage), envoi, prospects
  récents, messages, exécutions récentes (avec les notes de la prospection), incidents, demandes reçues.
- L'instantané est encadré comme données non fiables ; seul le message de la personne est une demande.
- Les **seules actions** possibles, sur demande explicite, sont dans le schéma de sortie : `prospection` (avec une
  consigne), `cadence`, `pause`, `reprise`, `evolution` (une demande de modification du logiciel, notée comme tâche à
  traiter en session de développement). Elles sont exécutées par le code, au nom de la personne, avec ses droits.
- Jamais d'envoi, d'approbation ni de modification d'e-mail, de devis ou de réglage depuis ce salon.
- Un historique des 6 derniers échanges est conservé pour le fil de la conversation.
- Sans l'option « Message Content Intent » du portail développeur, il faut mentionner le bot (@nom) pour lui parler.

## Les commandes

`/panneau` (republier le panneau) · `/etat` · `/a-valider` (republier la file) · `/prospection [consigne]` ·
`/relever` (relève de la boîte mail) · `/pause` · `/reprise` · `/cadence intense|normale|nuit`. Les réponses ne sont
visibles que par vous.

## Les comptes rendus de Claude Code (#claude-code)

Les sessions de développement publient leur compte rendu avec :

```bash
desk discord note "Titre" < compte-rendu.md
```

La commande enregistre le texte et le bot le publie dans #claude-code dans la minute. La consigne est inscrite dans le
`CLAUDE.md` du projet, pour que chaque session le fasse en fin d'étape.

## La file de publication

Le bot passe toutes les minutes : il ajoute à la file les éléments nouveaux (messages en attente, prospects, demandes,
estimations, incidents, rapports, comptes rendus de recherche et de nuit, notes Claude Code), puis publie chaque
élément une seule fois (table `discord_posts`, unique par type et identifiant). Un élément disparu entre-temps est
ignoré ; une publication en erreur est marquée et n'est pas renvoyée en boucle.

> Piège évité lors de l'ajout des annonces 🎯 : ajouter un nouveau type d'annonce aurait publié d'un coup toutes les
> entreprises existantes. Avant le déploiement, les entreprises existantes ont été marquées « déjà annoncées ».

## Sécurité

- Les contenus publiés transitent par Discord (entreprise américaine) : noms d'entreprises, adresses e-mail publiées,
  textes des e-mails. À mentionner dans le registre de conformité ; ne pas y faire figurer de données sensibles.
- Jeton compromis : le régénérer dans le portail développeur, puis remplacer le fichier du jeton (droits 600).
