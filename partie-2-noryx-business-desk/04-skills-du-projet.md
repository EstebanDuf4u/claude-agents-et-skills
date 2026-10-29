# 4. Les skills du projet

Six skills, dans `.claude/skills/noryx-*/SKILL.md`. Ils servent en **session interactive** : vous lancez `claude` dans
le dossier du projet et vous parlez normalement (« on commence la journée », « qualifie PRS-… »). Claude reconnaît
la demande grâce à la `description`, charge la procédure et l'exécute.

| Skill | Déclencheurs | Ce qu'il fait |
|---|---|---|
| `noryx-journee` | « on commence la journée », « point du jour », « qu'est-ce que j'ai à faire » | Diagnostic, état des tâches, délégation au Sales Manager, plan d'action proposé |
| `noryx-prospection` | « trouve des prospects », « cherche des entreprises qui… » | Contexte, délégation au Prospector, import simulé puis réel après accord |
| `noryx-qualification` | « qualifie ce prospect », « analyse la fiche PRS-… » | Contexte, délégation au Qualifier, simulation, accord, enregistrement |
| `noryx-relances` | « prépare les relances » | Délégation au Follow-up, messages ajoutés à la file de validation |
| `noryx-devis` | « prépare un devis pour… » | Délégation au Quote Architect, devis brouillon à relire |
| `noryx-estimation` | « estime ce projet », « combien de temps / combien ça coûte » | Estimator puis Estimate Reviewer, prix calculé par l'application |

## Le circuit commun

Tous les skills suivent le même schéma, qui empêche toute écriture sans relecture :

```
1. ./bin/desk agent context <agent> [--company PRS-…] [--opportunity N] [--instruction "…"]
        → le même contexte JSON que l'exécution planifiée
2. délégation au sous-agent noryx-<agent>
3. sortie écrite dans data/agent-io/<agent>-<horodatage>.json
        → conforme à ./bin/desk agent schema <agent>
4. ./bin/desk agent submit <agent> <fichier> --dry-run
        → montre ce qui serait appliqué et ce qui serait refusé
5. accord explicite de la personne
6. ./bin/desk agent submit <agent> <fichier>
        → application avec les droits de l'agent, exactement comme en planifié
```

Les permissions du projet (`.claude/settings.json`) renforcent ce circuit : lire le contexte et le schéma est
autorisé sans demander, **soumettre ou lancer un agent demande toujours une confirmation**, et l'accès direct à la base
est interdit.

```json
{
  "permissions": {
    "allow": ["Bash(./bin/desk agent context *)", "Bash(./bin/desk agent schema *)",
              "Bash(./bin/desk doctor)", "Bash(./bin/desk scheduler status)",
              "Write(./data/agent-io/**)", "Read(./data/agent-io/**)"],
    "ask":   ["Bash(./bin/desk agent submit *)", "Bash(./bin/desk agent run *)", "Bash(./bin/desk scheduler run *)"],
    "deny":  ["Read(./data/secret.key)", "Read(./data/smtp.password)", "Read(./config/desk.toml)",
              "Edit(./data/**)", "Write(./data/*.db)", "Bash(sqlite3 *)",
              "Bash(./bin/desk restore *)", "Bash(./bin/desk user *)", "Bash(rm -rf *)"]
  }
}
```

## Exemple : `noryx-journee`

```markdown
---
name: noryx-journee
description: Démarrer la journée commerciale Noryx - état du pipeline, relances du jour, devis en attente, e-mails à
  valider, incidents d'automatisation, puis plan d'action proposé. À utiliser quand la personne dit « on commence la
  journée », « point du jour », « qu'est-ce que j'ai à faire ».
---

1. Lancer `./bin/desk doctor` et signaler tout point « !! » (notamment un risque de facturation API).
2. Lancer `./bin/desk scheduler status` : repérer les tâches en échec ou jamais exécutées.
3. Obtenir le contexte du coordinateur : `./bin/desk agent context sales-manager`.
4. Déléguer l'analyse au sous-agent `noryx-sales-manager` en lui transmettant ce contexte.
5. Présenter, en peu de lignes : relances et tâches du jour, devis en attente, e-mails à valider ; priorités
   proposées avec leur raison ; incidents ouverts.
6. Proposer les actions suivantes, sans les exécuter sans accord.

Ne jamais présenter un chiffre absent du contexte. Un indicateur `null` est « non calculable ».
```

## Pourquoi un skill plutôt qu'une consigne dans `CLAUDE.md`

La procédure de qualification ne sert que quand on qualifie. En skill, elle ne coûte que sa description tant qu'on ne
l'utilise pas ; dans `CLAUDE.md`, elle serait chargée à chaque session. `CLAUDE.md` ne garde que le circuit général
en trois lignes et les règles impératives.
