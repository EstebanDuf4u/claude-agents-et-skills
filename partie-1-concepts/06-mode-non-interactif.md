# 6. Le mode non interactif (`claude -p`)

`claude -p` (ou `--print`) exécute une tâche sans terminal interactif et rend un résultat sur la sortie standard.
C'est la brique qui transforme un sous-agent en **tâche planifiable** : un cron, un timer systemd ou votre propre
application peut lancer un agent, récupérer sa sortie et la traiter.

```bash
claude -p "Résume README.md" --output-format json
```

Le code de sortie vaut 0 en cas de succès, autre chose en cas d'échec. Une erreur survenue pendant l'exécution
(authentification absente…) est rendue comme résultat sur la sortie standard.

## Les options utiles pour un agent automatisé

| Option | Rôle |
|---|---|
| `--agent <nom>` | Toute la session prend la consigne, les outils et le modèle de `.claude/agents/<nom>.md`. |
| `--output-format json` | Rend un objet JSON : `result` (texte), `structured_output`, `session_id`, `num_turns`, `usage`, `is_error`, `subtype`… |
| `--json-schema '<schéma>'` | Demande une sortie conforme à un schéma JSON, rendue dans `structured_output`. |
| `--max-turns N` | Plafond de tours. |
| `--permission-mode dontAsk` | Refuse tout ce qui n'est pas explicitement autorisé. |
| `--tools "WebSearch,WebFetch"` | Outils disponibles (chaîne vide = aucun outil). |
| `--allowedTools "…"` | Outils autorisés sans demande. |
| `--strict-mcp-config` | N'utilise que les serveurs MCP passés en option (aucun du dossier). |
| `--no-session-persistence` | N'enregistre pas la session sur disque. |
| `--session-id <uuid>` | Fixe l'identifiant de la session (pour la reprendre ensuite). |
| `--resume <id>` | Reprend une session existante avec un nouveau message. |
| `--model`, `--effort` | Modèle et effort. |
| `--append-system-prompt` | Ajoute des consignes sans remplacer celles de Claude Code. |

La consigne peut être passée en argument ou **par l'entrée standard**, ce qui évite les limites de longueur de ligne
de commande pour un gros contexte (10 Mo au plus).

## Sortie structurée : ne faites pas confiance aveuglément

Constat sur la version 2.1.312 : la sortie arrive **tantôt dans `structured_output`, tantôt sous forme de texte JSON
dans `result`**, et le schéma n'est pas toujours strictement imposé (des clés en plus sont possibles). Une exécution
peut aussi se terminer sans sortie structurée (refus, limite de tours).

La bonne pratique :

1. lire `structured_output`, sinon extraire le premier objet JSON de `result` ;
2. **valider strictement** dans votre code (pydantic avec `extra="forbid"`, par exemple) ;
3. rejeter toute sortie non conforme sans rien écrire, et l'enregistrer pour diagnostic.

Voir [`exemples/executeur_minimal.py`](../exemples/executeur_minimal.py).

## Reprendre une session : relancer un agent qui s'arrête trop tôt

Un agent en `-p` décide seul quand il a fini. Il peut s'arrêter bien avant l'objectif (constaté : un agent de
prospection qui conclut au bout de 4 tours alors qu'il en a 150). Une consigne plus insistante ne suffit pas toujours.

Solution robuste, côté code :

```bash
claude -p --agent prospecteur --session-id "$ID" … < consigne.txt        # 1re passe
claude -p --agent prospecteur --resume "$ID" … <<< "Tu en es à 12 sur 30, continue."   # relance
```

La relance garde tout l'historique de la session (pages déjà lues, entreprises déjà vues). Le programme cumule les
sorties des passes, s'arrête quand l'objectif est atteint, quand une relance n'apporte rien de nouveau, ou après un
nombre maximal de relances. Il supprime ensuite la session du disque. Voir
[partie 2, chapitre 5](../partie-2-noryx-business-desk/05-executer-un-agent.md).

## Mode `--bare` : attention à l'authentification

`--bare` démarre plus vite en ignorant hooks, skills, agents, plugins, MCP et `CLAUDE.md`. Mais **en mode `--bare`,
Claude Code n'utilise jamais la connexion à l'abonnement** : il faut une clé API (`ANTHROPIC_API_KEY` ou
`apiKeyHelper`). Un projet qui veut rester sur l'abonnement ne doit donc **pas** utiliser `--bare`.

## Abonnement : les contrôles à faire avant chaque appel

En `-p`, Claude Code utilise une clé API dès qu'elle est présente. Pour garantir l'usage de l'abonnement :

- vérifier l'absence de `ANTHROPIC_API_KEY`, `ANTHROPIC_AUTH_TOKEN`, `ANTHROPIC_BASE_URL`, `CLAUDE_CODE_USE_BEDROCK`,
  `CLAUDE_CODE_USE_VERTEX`, `CLAUDE_CODE_USE_FOUNDRY`… dans l'environnement du processus ;
- vérifier l'absence d'`apiKeyHelper` dans les réglages de Claude Code ;
- vérifier que `claude auth status` indique une connexion `claude.ai` ;
- lancer l'enfant avec un environnement nettoyé de ces variables.

Le quota de l'abonnement est **partagé** entre vos sessions interactives et vos agents planifiés. Un agent qui tourne
toute la nuit entame aussi la limite hebdomadaire.
