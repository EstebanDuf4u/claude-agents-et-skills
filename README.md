# Agents et skills Claude Code — guide complet et cas réel

Ce dépôt explique, en français, comment fonctionnent les **sous-agents**, les **skills** et l'exécution
**non interactive** de Claude Code, puis montre une mise en œuvre réelle : **Noryx Business Desk**, un outil
commercial interne dans lequel une douzaine d'agents prospectent, qualifient, rédigent et estiment, sans jamais agir
seuls à l'extérieur.

- **Partie 1 — Comprendre** : les briques de Claude Code, indépendamment de tout projet.
- **Partie 2 — Le cas Noryx Business Desk** : l'architecture complète, chaque agent, chaque garde-fou, et le journal
  de ce qui a été construit, corrigé et pourquoi.
- **Exemples** : des fichiers prêts à copier (agent, skill, réglages, exécuteur minimal).

> Les informations sur Claude Code correspondent à la documentation officielle et au comportement constaté avec la
> version 2.1.312 (octobre 2026). Les fonctions évoluent vite : en cas de doute, la référence est
> [code.claude.com/docs](https://code.claude.com/docs).

## Sommaire

### Partie 1 — Comprendre

| # | Chapitre | En une phrase |
|---|---|---|
| 1 | [Claude Code en bref](partie-1-concepts/01-claude-code-en-bref.md) | Une boucle d'agent avec des outils, dans votre dossier |
| 2 | [CLAUDE.md et mémoire](partie-1-concepts/02-claude-md-et-memoire.md) | Les consignes permanentes d'un projet |
| 3 | [Les sous-agents](partie-1-concepts/03-sous-agents.md) | Des spécialistes avec leur propre contexte et leurs propres outils |
| 4 | [Les skills](partie-1-concepts/04-skills.md) | Des procédures chargées seulement quand elles servent |
| 5 | [Permissions et réglages](partie-1-concepts/05-permissions-et-reglages.md) | Ce que Claude peut faire sans demander |
| 6 | [Le mode non interactif (`claude -p`)](partie-1-concepts/06-mode-non-interactif.md) | Utiliser Claude Code comme un programme |
| 7 | [Hooks, plugins, MCP et channels](partie-1-concepts/07-hooks-plugins-mcp-channels.md) | Étendre et piloter à distance |
| 8 | [Sécurité des agents](partie-1-concepts/08-securite-des-agents.md) | Injection, plafonds de droits, validation humaine |
| 9 | [Bonnes pratiques](partie-1-concepts/09-bonnes-pratiques.md) | Ce qui marche, ce qui casse |

### Partie 2 — Le cas Noryx Business Desk

| # | Chapitre | Contenu |
|---|---|---|
| 1 | [Vue d'ensemble](partie-2-noryx-business-desk/01-vue-d-ensemble.md) | Objectif, principes, parcours d'un prospect |
| 2 | [Architecture](partie-2-noryx-business-desk/02-architecture.md) | Composants, données, sécurité |
| 3 | [L'équipe d'agents](partie-2-noryx-business-desk/03-equipe-d-agents.md) | Les 11 agents, leurs entrées, sorties et interdits |
| 4 | [Les skills du projet](partie-2-noryx-business-desk/04-skills-du-projet.md) | Les 6 procédures interactives |
| 5 | [Exécuter un agent sans facturation API](partie-2-noryx-business-desk/05-executer-un-agent.md) | L'exécuteur, ses contrôles, les relances en session |
| 6 | [De la sortie de l'agent à la base](partie-2-noryx-business-desk/06-de-la-sortie-a-la-base.md) | Schémas stricts, application, plafond de droits |
| 7 | [Ordonnanceur et cadences](partie-2-noryx-business-desk/07-ordonnanceur-et-cadences.md) | Qui tourne quand, et pourquoi dans cet ordre |
| 8 | [Pipeline commercial](partie-2-noryx-business-desk/08-pipeline-commercial.md) | Étapes, transitions, passages de relais |
| 9 | [Prospection](partie-2-noryx-business-desk/09-prospection.md) | Sources, règles sur les adresses, mémoire, persévérance |
| 10 | [Envoi des e-mails et délivrabilité](partie-2-noryx-business-desk/10-envoi-et-delivrabilite.md) | Validation, contrôles, SPF/DKIM/DMARC/rDNS, listes noires |
| 11 | [Discord : panneau de contrôle et assistant](partie-2-noryx-business-desk/11-discord.md) | Salons, panneau, assistant conversationnel |
| 12 | [Conformité et RGPD](partie-2-noryx-business-desk/12-conformite.md) | Prospection B2B, registre, droits des personnes |
| 13 | [Déploiement et exploitation](partie-2-noryx-business-desk/13-deploiement-et-exploitation.md) | Serveur, systemd, sauvegardes, mises à jour |
| 14 | [Tests](partie-2-noryx-business-desk/14-tests.md) | Faux exécutable `claude`, ce qui est couvert |
| 15 | [Journal des évolutions](partie-2-noryx-business-desk/15-journal-des-evolutions.md) | Ce qui a été construit, les problèmes rencontrés, les corrections |
| 16 | [Limites et pistes](partie-2-noryx-business-desk/16-limites-et-pistes.md) | Ce qui reste imparfait |

### Exemples

| Fichier | Rôle |
|---|---|
| [`exemples/agents/redacteur.md`](exemples/agents/redacteur.md) | Sous-agent complet, avec règles communes |
| [`exemples/skills/point-du-jour/SKILL.md`](exemples/skills/point-du-jour/SKILL.md) | Skill de procédure |
| [`exemples/settings.json`](exemples/settings.json) | Permissions d'un projet à agents |
| [`exemples/executeur_minimal.py`](exemples/executeur_minimal.py) | Lancer un agent en `claude -p` avec sortie validée |
| [`exemples/CLAUDE.md`](exemples/CLAUDE.md) | Consignes de projet |

## Ce que ce dépôt n'est pas

- Ce n'est pas le code de Noryx Business Desk : les extraits sont reproduits quand ils éclairent un mécanisme.
- Ce n'est pas un avis juridique : les passages sur la prospection et le RGPD décrivent des garde-fous techniques et
  signalent ce qui doit être validé par un professionnel.
- Aucune donnée réelle n'y figure : pas de secret, d'adresse IP, d'identifiant, de nom de prospect ni d'adresse
  e-mail réelle. Les exemples utilisent des domaines `.invalid` et des valeurs fictives.
