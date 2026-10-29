# 16. Limites et pistes

## Limites connues

### Agents

- **Le modèle décide seul quand il a fini.** Les relances en session compensent, mais le rendement dépend fortement
  du métier et de la source : certains segments (coiffeurs, boulangeries) publient très peu de coordonnées.
- **`--json-schema` n'est pas strictement imposé** par Claude Code 2.1.312 : une sortie non conforme est rejetée (rien
  n'est écrit), mais elle a consommé une exécution.
- **Le quota de l'abonnement est partagé** avec l'usage interactif : une nuit en continu entame la limite
  hebdomadaire. Les recherches longues (jusqu'à 50 min) consomment davantage.
- Les autorisations `allow` du projet ne s'appliquent qu'après acceptation de la confiance du dossier.

### Ordonnancement

- Un seul agent IA réellement exécuté par passage de 5 minutes (délai minimal et verrou). Avec la priorité « aval
  d'abord », la prospection ne tourne que quand les autres agents n'ont rien à faire ou attendent leur intervalle.
- Le prochain passage affiché dans le panneau est une **estimation** : il ne modélise pas la priorité entre agents.
- Un changement de cadence remplace les intervalles réglés à la main pour trois tâches.

### E-mails

- Seule la syntaxe des adresses est vérifiée (pas de vérification MX).
- La relève ne lit pas les pièces jointes (seul leur nom est noté) ; IMAP uniquement.
- Les avis de non-distribution sont importés mais **pas encore rattachés automatiquement** au message d'origine :
  il faut utiliser « Marquer non distribué ».

### Application

- SQLite : une machine, une petite équipe.
- Pas d'authentification à deux facteurs ; accès distant par tunnel SSH ou proxy authentifiant.
- PDF de devis standard (pas de PDF/A ni de signature électronique).
- La facturation définitive reste dans un outil séparé.

### Conformité

- Aucune validation juridique à ce jour (voir le registre et le [chapitre 12](12-conformite.md)).

## Pistes

| Piste | Gain attendu |
|---|---|
| Rattacher automatiquement un avis de non-distribution à son message | Plus de message « envoyé » jamais arrivé |
| Piloter la session de développement depuis Discord (plugin channels officiel) | Demander une évolution depuis le téléphone, compte rendu dans Discord |
| Rendement par source et par métier dans le panneau | Orienter la cible vers ce qui rend |
| Montée progressive automatique de la limite d'envoi selon les retours | Délivrabilité préservée |
| Suivi des réponses et taux de réponse par segment | Mesurer le démarchage, pas seulement la prospection |
| Vérification MX des adresses avant rédaction | Moins de rejets |
| Régénération du jeton du bot Discord | Hygiène (le jeton a transité dans une conversation) |
