# 1. Claude Code en bref

Claude Code est un **agent** : un modèle de langage placé dans une boucle, avec des outils. À chaque tour, le modèle
lit la conversation, décide d'appeler un outil (lire un fichier, lancer une commande, chercher sur le web…), reçoit
le résultat, et recommence jusqu'à pouvoir répondre.

```
vous ──▶ message ──▶ modèle ──▶ appel d'outil ──▶ résultat ──▶ modèle ──▶ … ──▶ réponse
                         ▲                                         │
                         └──────────── contexte (fenêtre) ─────────┘
```

## Les notions à connaître

| Notion | Ce que c'est |
|---|---|
| **Session** | Une conversation, avec son historique. Elle est enregistrée sur la machine (`~/.claude/projects/…`) et peut être reprise. |
| **Contexte** | Tout ce que le modèle « voit » à un instant : consignes, messages, résultats d'outils. Sa taille est limitée ; au-delà, la conversation est résumée (*compaction*). |
| **Tour** (*turn*) | Un aller-retour modèle → outil → modèle. Les exécutions automatisées en limitent le nombre (`--max-turns`). |
| **Outils** | `Read`, `Write`, `Edit`, `Bash`, `Grep`, `Glob`, `WebSearch`, `WebFetch`, `Agent` (déléguer à un sous-agent), `Skill` (charger une procédure)… |
| **Dossier de travail** | Le dossier où Claude Code est lancé. Ses fichiers `.claude/` et `CLAUDE.md` configurent le projet. |
| **Permissions** | Les règles qui décident si un outil peut s'exécuter sans vous demander. Voir [chapitre 5](05-permissions-et-reglages.md). |

## Les briques de personnalisation

| Brique | Fichier | Chargée quand | Sert à |
|---|---|---|---|
| `CLAUDE.md` | racine du projet, `~/.claude/CLAUDE.md` | toujours, au démarrage | règles permanentes du projet |
| Sous-agent | `.claude/agents/<nom>.md` | quand Claude délègue, ou avec `--agent` | un spécialiste isolé, avec ses outils |
| Skill | `.claude/skills/<nom>/SKILL.md` | quand la demande correspond à sa description, ou `/nom` | une procédure, chargée à la demande |
| Réglages | `.claude/settings.json` | toujours | permissions, hooks, variables |
| Hook | dans les réglages | sur un événement (avant un outil, à la fin…) | automatismes garantis, exécutés par Claude Code et non par le modèle |
| Plugin | installé par `/plugin` | selon le plugin | paquet de skills, agents, hooks, serveurs MCP |
| Serveur MCP | `.mcp.json` | au démarrage | outils supplémentaires (bases, API…) |

La différence essentielle : **un `CLAUDE.md` et un hook sont toujours actifs ; un skill et un sous-agent ne coûtent
du contexte que lorsqu'ils servent.**

## Deux façons de l'utiliser

1. **Interactive** : `claude` dans un terminal. Vous dialoguez, vous approuvez les actions sensibles.
2. **Non interactive** : `claude -p "consigne"`. Claude Code exécute la tâche et rend un résultat (texte ou JSON),
   comme n'importe quel programme. C'est ce qui permet de **planifier des agents**. Voir
   [chapitre 6](06-mode-non-interactif.md).

## Abonnement ou API

Claude Code peut s'authentifier de deux façons :

- **Abonnement claude.ai** (Pro, Max…) : connexion par `/login`. L'usage est compris dans l'abonnement, avec des
  limites par période (sur 5 heures et sur la semaine).
- **Clé API** (Console) : facturation à l'usage, au jeton.

En mode `-p`, **une variable `ANTHROPIC_API_KEY` présente dans l'environnement est utilisée en priorité**. Un projet
qui veut rester sur l'abonnement doit donc vérifier activement l'absence de clé avant chaque exécution : c'est ce que
fait Noryx Business Desk ([partie 2, chapitre 5](../partie-2-noryx-business-desk/05-executer-un-agent.md)).
