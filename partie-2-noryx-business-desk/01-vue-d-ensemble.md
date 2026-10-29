# 1. Vue d'ensemble

**Noryx Business Desk** est l'outil commercial interne de Noryx, un studio technologique indépendant. Il gère les
prospects, le pipeline, les devis, les relances et les e-mails, et s'appuie sur une équipe d'agents Claude Code qui
**préparent** le travail : recherche de prospects, qualification, rédaction, estimation de charge, tri de la boîte
mail.

## Les principes non négociables

| Principe | Mise en œuvre |
|---|---|
| **Aucune facturation API** | Les agents passent uniquement par `claude -p` connecté à l'abonnement. Toute configuration qui ferait basculer vers une facturation à l'usage bloque l'IA. |
| **Aucune action externe sans validation humaine** | Les agents créent des fiches, des tâches et des brouillons. Seule une personne approuve, envoie, émet ou déclare un paiement. |
| **Rien d'inventé** | Pas de mention légale, de tarif, de référence client, de témoignage ni de statistique fabriqués. Un champ inconnu reste vide et l'interface le signale. |
| **Chiffres réels uniquement** | Les indicateurs sont calculés à partir des données enregistrées. Les fiches de test sont exclues. Un ratio sans base est « non calculable ». |
| **Calculs exacts** | Tous les montants passent par un module `Decimal` (arrondi au centime, demi vers le haut). Jamais de flottant. |
| **Contenus importés = données** | Pages web, e-mails et notes ne sont jamais traités comme des instructions. |

## Le parcours d'un prospect

```
 Recherche (Prospector)          ──▶ fiche « À qualifier », sources datées, contact publié
   │                                 🎯 annonce dans #assistant, fiche dans #prospects
   ▼
 Qualification (Qualifier)       ──▶ faits / hypothèses / manques, priorité, opportunité
   │                                 e-mail publié → « À contacter » ; téléphone seul → tâche « Appeler »
   ▼
 Premier message (Copywriter)    ──▶ e-mail « en attente de validation » dans #validation
   │
   ▼
 Validation humaine              ──▶ la personne lit le message en entier et l'approuve (Desk ou Discord)
   │
   ▼
 Envoi (SMTP)                    ──▶ contrôles rejoués au dernier moment ; fiche « Contacté »
   │
   ▼
 Relances (Follow-up)            ──▶ J+7 après un premier contact, J+5 et J+12 après un devis
   │
   ▼
 Réponse reçue (Inbox Triage)    ──▶ demande de projet → estimation → devis brouillon → relu et émis par vous
```

## Ce qui tourne tout seul, et ce qui attend une personne

| Automatique | Toujours humain |
|---|---|
| Recherche, qualification, rédaction des brouillons | Approuver et envoyer un e-mail |
| Relève de la boîte mail (lecture seule) et tri | Émettre un devis, modifier un prix |
| Estimation de charge et contre-expertise | Passer un prospect en « Contacté », « Gagné », « Perdu »… |
| Détection des relances, contrôles, sauvegardes, rapports | Déclarer un paiement |
| Compte rendu de la nuit, annonces Discord | Modifier les réglages d'envoi et de conformité |

## Les chiffres d'une journée type (29 octobre 2026)

En cadence intense, entre 12 h et 20 h :

- 27 nouveaux prospects, tous confirmés au registre des entreprises : 25 avec une adresse e-mail publiée par
  l'entreprise, 2 avec un téléphone seul ;
- 29 fiches passées « À contacter » après correction du passage de relais ;
- 25 exécutions d'agents sur un plafond de 120, aucun incident.

La [partie 2, chapitre 15](15-journal-des-evolutions.md) raconte comment on en est arrivé là, problèmes compris.
