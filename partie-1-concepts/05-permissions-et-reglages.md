# 5. Permissions et réglages

Avant chaque appel d'outil, Claude Code décide : **autoriser**, **demander** ou **refuser**. Ces décisions viennent
des règles de permissions, du mode de permission et des hooks.

## Les fichiers de réglages

| Fichier | Portée | Versionné ? |
|---|---|---|
| `~/.claude/settings.json` | vous, tous projets | non |
| `.claude/settings.json` | le projet, pour tout le monde | oui |
| `.claude/settings.local.json` | le projet, pour vous seul | non |
| réglages gérés | l'organisation (non modifiables) | — |

## Les règles

```json
{
  "permissions": {
    "allow": ["Bash(./bin/desk agent context *)", "Read(./data/agent-io/**)"],
    "ask":   ["Bash(./bin/desk agent submit *)"],
    "deny":  ["Read(./config/secrets.toml)", "Bash(sqlite3 *)", "Bash(rm -rf *)"]
  }
}
```

- `allow` : exécuté sans demander.
- `ask` : toujours demander, même si une autre règle autorise.
- `deny` : toujours refusé. **Un refus l'emporte sur tout.**

Syntaxe : `Outil(motif)`. Pour `Bash`, le motif est la commande ; `*` final avec un espace avant
(`Bash(git diff *)`) correspond à « commence par `git diff ` ». Pour `Read`/`Edit`/`Write`, le motif est un chemin
(`**` = sous-dossiers).

> **Confiance du dossier.** En session interactive, les règles `allow` d'un `.claude/settings.json` de projet ne
> s'appliquent qu'après acceptation de la boîte de dialogue de confiance du dossier. Les règles `deny` s'appliquent
> toujours. En mode `-p`, aucune boîte de dialogue n'apparaît : un message du type « Ignoring N permissions.allow
> entries … workspace has not been trusted » peut s'afficher.

## Les modes de permission

| Mode | Comportement |
|---|---|
| `default` / `manual` | Demande pour tout ce qui n'est pas autorisé. |
| `acceptEdits` | Écritures de fichiers sans demande. |
| `plan` | Lecture seule, prépare un plan. |
| `auto` | Un classificateur examine chaque action à votre place. |
| `dontAsk` | **Refuse** tout ce qui demanderait une autorisation. Idéal pour les exécutions automatiques verrouillées. |
| `bypassPermissions` | Ne demande presque rien. À réserver aux environnements isolés. |

Pour un agent planifié, la combinaison sûre est : **`--permission-mode dontAsk` + liste explicite d'outils**
(`--tools` et `--allowedTools`). L'agent ne peut faire que ce qui est listé ; tout le reste est refusé sans attendre.

## Où se situe la vraie frontière de sécurité

Les permissions de Claude Code protègent **la machine**. Elles ne protègent pas **vos données métier** : si un agent
a le droit d'écrire dans votre base, il peut y écrire n'importe quoi.

C'est pourquoi le cas Noryx adopte une règle plus forte : **les agents n'écrivent jamais eux-mêmes dans la base**. Ils
rendent un JSON, et c'est le code de l'application qui l'applique, avec ses propres contrôles et un plafond de droits.
La règle `deny` sur `sqlite3` et sur l'écriture des fichiers `.db` n'est qu'une seconde ceinture.
Voir [partie 2, chapitre 6](../partie-2-noryx-business-desk/06-de-la-sortie-a-la-base.md).
