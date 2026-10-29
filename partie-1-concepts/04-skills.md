# 4. Les skills

Un **skill** est un dossier contenant un fichier `SKILL.md` : un en-tête YAML suivi d'instructions en Markdown.
Claude le charge **quand la demande correspond à sa description**, ou vous le lancez avec `/nom-du-skill`. Les skills
suivent le standard ouvert [Agent Skills](https://agentskills.io), étendu par Claude Code.

Un skill répond à la question « **comment** faire telle tâche ici ». Un sous-agent répond à « **qui** la fait ».
Souvent, un skill décrit une procédure qui délègue une partie du travail à un sous-agent.

## Chargement progressif : pourquoi les skills sont légers

1. **Liste** : seuls le `name` et la `description` (plus `when_to_use`) sont toujours dans le contexte, tronqués à
   1 536 caractères.
2. **Corps** : le `SKILL.md` complet n'est chargé que lorsque le skill est invoqué. Il reste ensuite dans la
   conversation.
3. **Fichiers annexes** : référencés depuis `SKILL.md`, lus seulement si besoin. Garder `SKILL.md` sous 500 lignes.

Dix skills ne coûtent donc presque rien tant qu'on ne s'en sert pas, contrairement à dix pages dans `CLAUDE.md`.

## Organisation

```
.claude/skills/
└── point-du-jour/
    ├── SKILL.md          # obligatoire : en-tête + instructions
    ├── reference.md      # lu seulement quand utile
    └── scripts/outil.py  # exécuté, pas chargé dans le contexte
```

| Portée | Chemin |
|---|---|
| Personnelle | `~/.claude/skills/<nom>/SKILL.md` |
| Projet | `.claude/skills/<nom>/SKILL.md` |
| Sous-dossier | `<dossier>/.claude/skills/<nom>/SKILL.md` (actif dans ce dossier) |
| Plugin | `<plugin>/skills/<nom>/SKILL.md`, appelé `/plugin:nom` |

En cas de même nom : organisation > personnel > projet. Un skill remplace une ancienne commande
`.claude/commands/<nom>.md` du même nom (les commandes restent supportées, mais les skills sont à préférer).

## Les champs de l'en-tête

Tous sont facultatifs ; `description` est fortement recommandé. Un champ inconnu est ignoré sans erreur.

| Champ | Rôle |
|---|---|
| `name` | Nom dans le menu `/` (par défaut : nom du dossier). |
| `description` | Ce que fait le skill et **quand** l'utiliser. Mettre les formulations des utilisateurs (« on commence la journée », « prépare un devis pour… »). |
| `when_to_use` | Phrases déclencheuses supplémentaires. |
| `argument-hint` | Aide à la saisie, par exemple `[numéro]`. |
| `arguments` | Arguments nommés, utilisables comme `$nom`. |
| `disable-model-invocation` | `true` : seul l'utilisateur peut le lancer (pour les procédures à effets, comme un déploiement). |
| `user-invocable` | `false` : caché du menu, seul Claude l'utilise (connaissances de fond). |
| `allowed-tools` | Outils autorisés sans demande **pendant le tour** où le skill est invoqué. |
| `disallowed-tools` | Outils retirés pendant que le skill est actif. |
| `model`, `effort` | Modèle et effort pour la suite du tour. |
| `context: fork` + `agent` | Exécute le skill dans un sous-agent (`Explore`, `Plan`, `general-purpose` ou un agent du projet). Le sous-agent ne voit pas la conversation : le skill doit se suffire à lui-même. |
| `paths` | Motifs de fichiers limitant l'activation automatique. |
| `hooks` | Hooks enregistrés quand le skill est invoqué. |

## Substitutions

| Écriture | Remplacée par |
|---|---|
| `$ARGUMENTS` | tous les arguments |
| `$0`, `$1`… | argument n° N |
| `${CLAUDE_PROJECT_DIR}` | racine du projet |
| `${CLAUDE_SKILL_DIR}` | dossier du skill |
| `` !`commande` `` | sortie de la commande, exécutée **avant** que Claude lise le skill |

Exemple d'injection dynamique :

```markdown
---
name: etat-du-depot
description: Résumer l'état du dépôt avant une revue
allowed-tools: Bash(git *)
---

- Branche : !`git branch --show-current`
- Fichiers modifiés : !`git status --short`

Résume ces changements en trois points.
```

Une commande `!` en échec fait échouer l'invocation (ajouter `|| true` pour tolérer l'échec).

## Qui peut lancer quoi

| En-tête | Vous (`/nom`) | Claude (automatique) | Description visible de Claude |
|---|---|---|---|
| (par défaut) | oui | oui | oui |
| `disable-model-invocation: true` | oui | non | non |
| `user-invocable: false` | non | oui | oui |

## Un skill de procédure type

```markdown
---
name: noryx-qualification
description: Qualifier un ou plusieurs prospects avec le sous-agent de qualification, en séparant faits, hypothèses
  et informations manquantes, puis enregistrer le résultat après accord. À utiliser pour « qualifie ce prospect ».
---

1. `./bin/desk agent context qualifier --company PRS-XXXXXX`
2. Déléguer au sous-agent `noryx-qualifier`. Sa sortie doit respecter `./bin/desk agent schema qualifier`.
3. Écrire la sortie dans `data/agent-io/qualifier-<horodatage>.json`.
4. Simuler : `./bin/desk agent submit qualifier <fichier> --dry-run` et montrer le résultat.
5. Après accord explicite : `./bin/desk agent submit qualifier <fichier>`.
```

Ce schéma (contexte → sous-agent → fichier → simulation → accord → application) est repris par tous les skills du
cas Noryx. Il garantit qu'**aucune écriture n'a lieu sans relecture humaine**, même en session interactive.
