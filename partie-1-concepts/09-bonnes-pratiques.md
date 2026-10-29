# 9. Bonnes pratiques

Retour d'expérience tiré du cas Noryx : ce qui a fonctionné, et les pièges rencontrés.

## Concevoir

- **Un agent = un métier.** Onze agents courts valent mieux qu'un agent qui sait tout faire.
- **Le prix, les totaux et les dates sont calculés par le code**, jamais par le modèle. Les agents d'estimation ne
  rendent que des durées ; l'application calcule le prix à partir du catalogue.
- **Un schéma de sortie par agent**, validé strictement. Les noms de champs en anglais, les valeurs en français : le
  modèle se trompe moins sur les clés.
- **Fournir le contexte plutôt que des outils.** Un agent de rédaction reçoit la fiche complète en JSON ; il n'a pas
  besoin d'accéder à la base.
- **Le mode simulation d'abord** : les premières exécutions rapportent ce qu'elles *feraient*, une personne relit,
  puis on passe en réel.

## Écrire les consignes

- **Des interdits formulés comme des actions précises.** « Deviner ou reconstruire une adresse (format prenom.nom,
  domaine supposé) » est suivi ; « respecte la vie privée » l'est moins.
- **Pas de contradiction entre deux sources de consigne.** Piège réel : la consigne de l'agent acceptait les adresses
  nominatives, mais la description du client idéal, saisie dans les réglages et transmise dans le contexte, disait
  « ne retenir QUE des adresses génériques ». L'agent écartait la plupart des entreprises.
- **Des objectifs chiffrés** (« examine au moins 30 entreprises nouvelles ») plutôt que « persévère ».
- **Ne pas compter sur la consigne seule pour la persévérance** : relancer l'agent dans sa session, côté code.

## Exécuter

- **Toujours vérifier l'authentification** avant d'appeler `claude -p` (voir chapitre 6).
- **Une seule exécution IA à la fois**, et un délai minimal entre deux.
- **L'ordre de passage compte.** Piège réel : à chaque passage de l'ordonnanceur, l'agent de prospection passait en
  premier et prenait le créneau ; l'agent de rédaction, après lui, tombait toujours sur le délai minimal et n'écrivait
  jamais rien. Solution : un seul agent par passage, l'aval du pipeline d'abord, et un agent sans travail ne prend pas
  le créneau.
- **Un agent sans travail ne lance pas Claude** : compter les éléments à traiter avant l'appel.
- **Les passages de relais entre agents doivent être vérifiés de bout en bout.** Piège réel : l'agent de qualification
  laissait les fiches « Qualifié » avec la prochaine action « préparer un premier message », mais l'agent de rédaction
  ne traitait que les fiches « À contacter ». Aucun e-mail n'était rédigé.

## Mesurer

- **Enregistrer ce que l'agent a examiné**, pas seulement ce qu'il a retenu : c'est ce qui révèle qu'il s'arrête trop
  tôt, et ce qui permet de ne pas refaire le même travail à la passe suivante.
- **Lire les notes de l'agent.** Elles disent souvent pourquoi il s'est arrêté (« je me suis arrêté après un seul lot,
  sans raison technique »).
- **Un tableau de bord des agents** (actif, dernier passage, prochain passage et ce qui le retarde) évite de découvrir
  un blocage des heures plus tard.

## Déployer

- **Ne pas déployer pendant une exécution** si le déploiement redémarre le processus qui la porte.
- **Les réglages enregistrés en base ne prennent pas les nouvelles valeurs par défaut** : changer un défaut dans le
  code ne suffit pas, il faut aussi mettre à jour la valeur enregistrée.
- **Toute écriture directe en production est une décision humaine** : outillez-la (bouton, commande auditée) plutôt
  que de la faire à la main.
