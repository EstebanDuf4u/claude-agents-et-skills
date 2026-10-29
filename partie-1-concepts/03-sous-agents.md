# 3. Les sous-agents

Un **sous-agent** est un assistant spécialisé auquel Claude délègue une tâche. Il travaille dans **son propre
contexte**, avec **sa propre consigne** (le corps du fichier) et **ses propres outils**. Seul son résumé final revient
dans la conversation principale : les recherches, journaux et fichiers lus restent chez lui.

Pourquoi c'est utile :

- **Contexte propre** : la conversation principale n'est pas encombrée par 50 pages web lues.
- **Spécialisation** : une consigne courte et précise vaut mieux qu'une consigne générale qui couvre tout.
- **Restriction** : un agent de rédaction n'a pas besoin de `Bash`, un agent de recherche n'a pas besoin d'`Edit`.
- **Réutilisation** : le même agent sert en session interactive et en exécution planifiée (`claude -p --agent …`).

## Le fichier

Un fichier Markdown avec un en-tête YAML. Le corps devient la consigne système de l'agent.

```markdown
---
name: redacteur
description: Rédige des e-mails B2B courts appuyés uniquement sur des faits vérifiables. À utiliser pour « écris un premier message à… ».
tools: Read, Write
model: sonnet
maxTurns: 6
color: orange
---

# Rédacteur

## Responsabilités
- Ouvrir sur un constat précis et vérifiable concernant l'entreprise…

## Interdits
- Relation préexistante inventée, chiffres non sourcés, urgence artificielle…
```

Un sous-agent reçoit **cette consigne**, le message de délégation écrit par Claude, les `CLAUDE.md`, un instantané
`git status` et les skills préchargés. **Il ne voit pas l'historique de votre conversation**, ni les fichiers que
Claude a déjà lus. Le message de délégation doit donc contenir tout ce dont il a besoin.

## Où les placer

| Emplacement | Portée | Priorité en cas de même nom |
|---|---|---|
| Réglages gérés (organisation) | toute l'organisation | 1 (la plus forte) |
| Option `--agents '<json>'` | la session en cours | 2 |
| `.claude/agents/` | le projet (à versionner) | 3 |
| `~/.claude/agents/` | tous vos projets | 4 |
| Dossier `agents/` d'un plugin | là où le plugin est activé | 5 |

Les dossiers sont parcourus récursivement ; seule la valeur de `name` compte. Les modifications sont détectées en
quelques secondes ; un dossier `agents/` nouvellement créé demande un redémarrage.

## Les champs de l'en-tête

Seuls `name` et `description` sont obligatoires. **Un champ mal orthographié est ignoré sans erreur** : respectez la
casse (`maxTurns`, pas `max_turns`).

| Champ | Rôle |
|---|---|
| `name` | Identifiant unique (sans `:`). |
| `description` | **Quand déléguer** à cet agent. C'est ce que Claude lit pour choisir : mettre le cas d'usage en premier. « Use proactively » encourage la délégation spontanée. |
| `tools` | Liste blanche d'outils (`Read, Grep, Bash`). Absent = tous les outils disponibles hérités. |
| `disallowedTools` | Outils retirés de la liste héritée. |
| `model` | `sonnet`, `opus`, `haiku`, `fable`, un identifiant complet, ou `inherit`. |
| `permissionMode` | `default`, `acceptEdits`, `auto`, `dontAsk`, `bypassPermissions`, `plan`. |
| `maxTurns` | Nombre maximal de tours. Au-delà, la sortie est marquée partielle. |
| `skills` | Skills dont le contenu est injecté au démarrage de l'agent. |
| `mcpServers` | Serveurs MCP propres à l'agent. |
| `hooks` | Hooks limités à cet agent (`PreToolUse`, `PostToolUse`, `Stop`…). |
| `memory` | Mémoire persistante : `user`, `project` ou `local`. |
| `background` | `true` : toujours en arrière-plan. |
| `isolation` | `worktree` : travaille dans une copie git temporaire. |
| `effort` | `low` à `max`. |
| `color` | Couleur d'affichage. |
| `omitClaudeMd` | `true` : ne charge pas les `CLAUDE.md`. |
| `initialPrompt` | Premier message envoyé automatiquement quand l'agent est l'agent principal de la session. |

> Les sous-agents de plugins ignorent `hooks`, `mcpServers` et `permissionMode`, par sécurité.

## Comment Claude délègue

1. **Automatiquement** : Claude compare la tâche aux `description` des agents.
2. **En le demandant** : « utilise le sous-agent redacteur pour… ».
3. **Par mention** : `@"redacteur (agent)" écris à…` garantit l'appel.
4. **Toute la session comme un agent** : `claude --agent redacteur` (ou `"agent": "redacteur"` dans les réglages).
   La session prend la consigne, les outils et le modèle de l'agent. **C'est ce mode qu'on utilise pour planifier un
   agent** : `claude -p --agent redacteur …`.

## Agents intégrés

| Agent | Rôle |
|---|---|
| `Explore` | Recherche rapide en lecture seule dans le code (ne charge ni `CLAUDE.md` ni `git status`) |
| `Plan` | Recherche en lecture seule pour préparer un plan |
| `general-purpose` | Tâches en plusieurs étapes, lecture et écriture |

## Limites utiles à connaître

- **Imbrication** : un sous-agent peut en lancer d'autres, jusqu'à trois niveaux par défaut
  (`CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH`).
- **Concurrence** : 20 sous-agents simultanés par défaut.
- **Quota** : leurs requêtes comptent dans les mêmes limites que la session principale.
- **Reprise** : un sous-agent terminé peut être relancé avec son historique (outil `SendMessage`) ; `Explore` et
  `Plan` sont à usage unique.

## Écrire une bonne consigne d'agent

La structure qui a le mieux fonctionné dans le cas Noryx :

1. **Rôle** en une phrase.
2. **Méthode** numérotée : ce qu'il fait, dans l'ordre.
3. **Sources ou entrées autorisées.**
4. **Interdits** explicites, formulés comme des actions (« deviner une adresse », « inventer une référence »).
5. **Critères de réussite** : à quoi ressemble une bonne sortie.
6. **Règles communes** à toute l'équipe (données non fiables, aucune invention, aucune action externe).
7. **Mode interactif** : comment l'utiliser depuis une session (commandes, fichiers).

Voir l'exemple complet [`exemples/agents/redacteur.md`](../exemples/agents/redacteur.md).
