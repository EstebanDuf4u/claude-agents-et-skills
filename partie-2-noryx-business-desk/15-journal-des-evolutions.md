# 15. Journal des évolutions

Le Desk a été construit en deux jours de sessions Claude Code, à partir d'un cahier des charges unique, puis amélioré
au fil de l'usage réel. Ce journal garde les décisions et surtout **les problèmes rencontrés**, parce que ce sont eux
qui ont façonné l'architecture.

## Jour 1 — 28 octobre 2026

### Construction

- **Socle** : FastAPI + SQLite + gabarits, comptes et rôles, prospects, pipeline, devis (calculs `Decimal`, PDF),
  messages avec validation, conformité, ordonnanceur systemd, dix agents, skills, 156 tests.
- **Premier lancement** : compte administrateur, informations de l'entreprise, catalogue d'offres. Les tarifs
  restent « à confirmer » tant qu'ils n'ont pas été validés par le dirigeant (rien d'inventé).
- **Estimation automatique** : deux agents (Estimator et Estimate Reviewer) qui ne rendent que des durées ; le prix
  est calculé par l'application. Demande précisée en cours de route : l'estimation doit partir **des e-mails clients**
  → relève IMAP en lecture seule, agent Inbox Triage, chaîne e-mail → estimation → devis brouillon.
- **Boîte mail** branchée sur la messagerie de l'entreprise avec un mot de passe d'application dédié, en lecture
  seule.
- **IA activée**, premier flux complet, recherche ciblée (entreprises sans site ou avec un besoin d'outil métier).
  Objectif fixé par le dirigeant : « la seule chose que j'ai à faire, c'est vérifier le mail et l'envoyer si je suis
  d'accord ».
- **Bot Discord** et serveur Discord de pilotage : salons, boutons d'approbation liés à l'empreinte du contenu.
- **Passage sur un serveur** : déploiement par `deploy-server.sh --migrer-donnees`, arrêt du Desk local (une seule
  instance, sinon double relève et doubles envois).

### Problèmes et corrections

| Problème | Cause | Correction |
|---|---|---|
| Sortie des agents parfois vide dans `structured_output` | Claude Code 2.1.312 renvoie parfois le JSON en texte dans `result`, et n'impose pas strictement le schéma | Lecture des deux formats, schéma recopié dans la consigne, validation pydantic stricte |
| Exécutions interrompues jamais marquées comme telles | Verrou nettoyé trop tôt | Marquage « interrompu » à l'expiration du verrou |
| Restauration par copie pouvant corrompre la base | Mode WAL | Restauration par l'API de sauvegarde SQLite |
| Premier e-mail envoyé rejeté | IP du serveur listée chez Spamhaus, rDNS mal configuré | Envoi suspendu ; agents maintenus en préparation |
| Recherche de prospects sans résultat (3 étapes sur 60) | La page web de l'annuaire officiel s'affiche vide pour un robot ; la recherche web ne renvoie que des annuaires | Méthode imposée : API JSON du registre, puis recherche du site et des coordonnées |
| Rendement faible | Seules les adresses génériques (`contact@`) étaient acceptées ; les petits artisans en publient rarement | Voir jour 2 |

### Organisation du travail des agents

- **Cadences** : « normale » à 8 h, « nuit » à 22 h (prospection en continu jusqu'au quota, e-mails préparés pour le
  matin), « intense » à la demande. Compte rendu de la nuit publié à 8 h.
- **Annonce de chaque recherche** sur Discord, suppression de la limite artificielle de dossiers par recherche
  (« un dossier complet et vérifié »).
- **Salon #assistant** : un assistant conversationnel (Claude, sans outils) qui fait des comptes rendus et relaie
  quelques actions de pilotage.
- Fin de journée : **tout arrêté** (pause globale, envoi suspendu), pour reprendre le lendemain.

## Jour 2 — 29 octobre 2026

### Matin : reprise et adresses e-mail

- Reprise vérifiée : pause levée depuis Discord, premier passage de l'ordonnanceur sans erreur.
- **Adresses nominatives et webmails** acceptés, à condition d'être publiés par l'entreprise elle-même pour son
  activité ; garde-fou : l'adresse doit figurer dans l'extrait de la page citée. **Téléphone seul** : dossier gardé,
  tâche « Appeler ». Décision inscrite au registre de conformité (à valider juridiquement).
- **🎯 Annonce dans #assistant** à chaque entreprise trouvée. Avant déploiement, les 17 entreprises existantes ont
  été marquées « déjà annoncées » pour éviter une publication en masse.

### Midi : le panneau de contrôle

Demande : « le Discord doit devenir le panneau de contrôle de tout ça : les agents actifs, les inactifs, les délais,
tout, et que ça reste lisible et agréable ».

- Salon **#tableau-de-bord** : quatre messages réédités sur place (état général, agents IA, tâches automatiques,
  activité), heures relatives gérées par Discord, prochain passage estimé avec ce qui le retarde, fiches privées pour
  activer, désactiver, lancer, changer l'intervalle.
- L'assistant connaît désormais l'état de chaque agent.

### Après-midi : faire persévérer la prospection

| Constat | Analyse | Correction |
|---|---|---|
| L'agent s'arrête après ~15 recherches | Pas une limite technique : 32 tours sur 60, arrêt volontaire | Mémoire des entreprises examinées (60 j), tableau de couverture activité × département, objectif de 30 candidats nouveaux, 150 tours, 50 min |
| L'ordonnanceur aurait coupé à 15 min | Durée maximale enregistrée en base (900 s) ; le nouveau défaut ne s'applique pas aux réglages déjà enregistrés | Mise à jour auditée de la valeur enregistrée (3 300 s) |
| Première recherche « nouvelle méthode » : 30 examinées, **12 retenues** | Les offices de tourisme publient les coordonnées des hôtels-restaurants ; les webmails sont maintenant acceptés | — |
| Consigne contradictoire | La description du client idéal (réglages) disait encore « ne retenir QUE des adresses génériques » | Phrase corrigée |
| Recherche toutes les semaines en journée | Profil de cadence « normale » | Toutes les 2 h en journée |
| Zone étroite | 3 départements | 6 départements |
| Sources trop génériques | Recherche web surtout | Annuaires de métiers (RGE, Qualibat, chambres de métiers, offices de tourisme, Logis…) |
| Après deux bonnes recherches, 11 recherches à 0 | L'agent conclut après un seul lot (« je me suis arrêté sans raison technique ») | **Relances dans la même session** (`--session-id` / `--resume`), 3 au plus, arrêt sans progrès ; résultat : 31 examinées sur 11 combinaisons en une exécution |
| Relances : 31 examinées mais 0 retenue sur des métiers pauvres | Écarts trop rapides | Consigne : privilégier les sources et métiers au meilleur rendement, consulter vraiment fiche d'annuaire et page Facebook |
| **Aucun premier message rédigé** de l'après-midi | Le Qualifier laissait « Qualifié » ; le Copywriter ne traite que « À contacter » | Relais automatique « Qualifié + e-mail publié → À contacter » ; 29 fiches rattrapées |
| Le Copywriter ne passait jamais | La prospection, première de la liste, prenait le créneau IA à chaque passage | Un seul agent IA par passage, l'aval du pipeline d'abord (correction prête, déploiement à confirmer) |

### Soir : rouvrir l'envoi

- **rDNS corrigé** : l'IP pointe vers le nom du serveur de mail, qui pointe vers la même IP ; SPF, DKIM, DMARC en
  place.
- **Listes noires** : non listé chez Spamhaus (ZEN et DBL, vérifié en interrogeant directement leurs serveurs), ni
  sur neuf autres listes d'IP ; SURBL et URIBL non concluants (résolveur public refusé).
- **Envoi rouvert**, 20 par jour, chaque e-mail toujours approuvé par une personne.
- Le premier e-mail, rejeté la veille, était resté marqué « envoyé » et bloquait son renvoi (« déjà contacté »).
  Correction ponctuelle auditée, puis ajout du bouton **« Marquer non distribué »** pour que le cas soit traité
  proprement à l'avenir.
- **Comptes rendus de Claude Code** dans un salon #claude-code (`desk discord note`).

### Bilan chiffré du 29 octobre (12 h – 20 h)

27 nouveaux prospects (25 avec adresse publiée, 2 avec téléphone), 31 fiches « À contacter », 25 exécutions d'agents
sur 120, aucun incident. Tests : 156 → 204.
