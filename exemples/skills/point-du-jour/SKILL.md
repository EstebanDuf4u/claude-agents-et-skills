---
name: point-du-jour
description: Faire le point de la journée commerciale - pipeline, relances du jour, devis en attente, e-mails à valider, incidents, puis plan d'action proposé. À utiliser quand la personne dit « on commence la journée », « point du jour », « qu'est-ce que j'ai à faire ».
---

# Point du jour

1. Lancer `./bin/app doctor` et signaler tout point « !! » (en particulier un risque de facturation API ou une
   connexion Claude Code absente).
2. Lancer `./bin/app scheduler status` : repérer les tâches en échec ou jamais exécutées.
3. Obtenir le contexte : `./bin/app agent context coordinateur`.
4. Déléguer l'analyse au sous-agent `coordinateur` en lui transmettant ce contexte en entier (il ne voit pas cette
   conversation).
5. Présenter en peu de lignes :
   - les relances et tâches du jour, les devis en attente, les e-mails à valider ;
   - les priorités proposées, chacune avec sa raison ;
   - les incidents ouverts.
6. Proposer les actions suivantes **sans les exécuter** sans accord.

Ne jamais présenter un chiffre absent du contexte. Un indicateur `null` est « non calculable ».
