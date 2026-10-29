# 2. Architecture

Un seul processus web, une base SQLite, un ordonnanceur déclenché par systemd, un bot Discord, et des agents exécutés
par Claude Code. Pas de microservice, pas de file de messages, pas de dépendance payante.

```
navigateur ──(tunnel SSH)──▶ desk serve (FastAPI + Jinja, 127.0.0.1)  ─┐
                                                                      │
timer systemd (5 min) ──▶ desk scheduler tick ───────────────────────┤
                              │ au plus un agent IA par passage       ├──▶ SQLite (WAL)
                              └──▶ claude -p --agent noryx-…          │
                                      │ sortie JSON                   │
                                      └──▶ validation stricte ──▶ application (droits « agent ») ──┘
                                                                      │
bot Discord (connexion sortante) ◀────── file de publications ────────┘
   salons, panneau de contrôle, boutons d'approbation, assistant (claude -p, sans outils)
```

## Technologies

| Couche | Choix | Pourquoi |
|---|---|---|
| Langage | Python ≥ 3.12 | Cohérent avec les autres outils Noryx |
| Web | FastAPI + gabarits Jinja, rendu serveur, JS natif sans CDN | Chaque action marche sans JavaScript ; CSP stricte |
| Base | SQLite en WAL, migrations SQL numérotées | Sauvegarde cohérente par l'API de sauvegarde SQLite ; une machine, une petite équipe |
| Validation | pydantic (`extra="forbid"`) | Toute sortie d'agent et tout réglage sont validés strictement |
| Montants | `Decimal` | Exactitude au centime |
| PDF | reportlab | Devis |
| Ordonnanceur | timer systemd utilisateur | Rien ne tourne entre deux passages ; reprise naturelle après redémarrage |
| IA | Claude Code (`claude -p`) sur abonnement | Aucun coût à l'usage |
| Discord | discord.py, connexion sortante | Aucun port ouvert |

## Arborescence du code

| Chemin | Contenu |
|---|---|
| `desk/config.py` | Configuration technique (TOML + variables `DESK_*`), politique d'écoute réseau |
| `desk/db.py` | Connexion, migrations, audit, incidents, sauvegarde |
| `desk/migrations/000N_*.sql` | Une migration par évolution de schéma |
| `desk/settings.py` | Réglages métier en base, validés ; contrôle des mentions obligatoires |
| `desk/permissions.py` | Cinq niveaux d'autorisation et plafond des agents |
| `desk/money.py` | Calculs de devis |
| `desk/services/` | Métier : `crm`, `quotes`, `mail`, `compliance`, `tasks`, `estimates`, `inbox`, `reports`, `notify` (Discord), `panel` (panneau de contrôle), `assistant` |
| `desk/scheduler.py` | Tâches récurrentes, verrous, tentatives, priorité des agents |
| `desk/agents/registry.py` | Définition des agents : consigne de tâche, contexte, travail en attente, relances |
| `desk/agents/schemas.py` | Schémas de sortie |
| `desk/agents/runner.py` | Exécution par Claude Code, contrôles de facturation et de quota |
| `desk/agents/ingest.py` | Application des sorties à la base, avec les droits de l'agent |
| `desk/discord_bot.py` | Le bot : salons, publication, boutons, panneau, assistant |
| `desk/web/` | Routes, gabarits, CSS et JS locaux |
| `.claude/agents/`, `.claude/skills/` | Sous-agents et procédures Claude Code |
| `deploy/systemd/` | Services et timers |
| `scripts/` | Installation, déploiement, cadence, sauvegarde, restauration, mise à jour |
| `tests/` | Suite automatisée (faux exécutable `claude`) |

## Données principales

Utilisateurs et sessions · réglages · offres · entreprises (prospects/clients) · contacts · sources · constats (faits
vérifiés, hypothèses, informations manquantes) · opportunités · interactions · tâches · devis, lignes et versions ·
estimations · messages et tentatives d'envoi · e-mails reçus · exclusions · demandes RGPD · registre de conformité ·
factures · exécutions d'agents et de tâches · verrous · incidents · rapports · publications Discord · candidats
examinés par la prospection · journal d'audit.

- Prospect : identifiant `PRS-XXXXXX`, source et date de collecte, dernière vérification, niveau de confiance, score
  justifié ligne à ligne, prochaine action, historique.
- Montants stockés en texte décimal exact ; totaux toujours recalculés à partir des lignes.

## Sécurité de l'application

| Mesure | Détail |
|---|---|
| Écoute locale | `127.0.0.1` uniquement ; refus de démarrer sur une autre adresse sans `allow_network = true`. Accès distant par tunnel SSH ou proxy HTTPS. |
| En-têtes | Contrôle de `Host` (anti DNS rebinding) et d'`Origin` des POST ; CSP stricte sans script ni style en ligne ; `X-Frame-Options: DENY` ; `no-store` |
| Authentification | Obligatoire partout ; mots de passe scrypt ; jeton de session aléatoire stocké haché ; cookie `HttpOnly`, `SameSite=Strict` |
| Limitation | 8 échecs de connexion en 15 min par adresse |
| CSRF | Jeton dans chaque formulaire et en-tête `X-CSRF` |
| Rôles | Admin (réglages, prix, comptes, envoi) / membre (préparation, validation) |
| Agents | Plafonnés à « modification de données internes » quel que soit le réglage |
| Secrets | Hors base, fichiers en 600 dans un dossier en 700 (mots de passe SMTP et IMAP, jeton Discord) |
