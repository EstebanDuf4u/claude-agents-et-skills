# 9. Prospection

L'agent **Prospector** constitue des dossiers de prospects **complets et vérifiés** à partir de sources publiques
autorisées. Il ne contacte jamais personne.

## La cible (réglages)

- Secteurs : artisans du bâtiment, restauration et hôtellerie, commerces de proximité, professions libérales, PME de
  services, PME industrielles. Exclus : grands groupes, administrations.
- Zones : six départements du Sud-Ouest.
- Client idéal : priorité 1, TPE et artisans **sans aucun site** ; priorité 2, TPE/PME de 1 à 50 salariés avec un
  besoin probable d'outil métier signalé publiquement, ou un site très ancien.
- Nombre maximal de nouveaux dossiers par recherche : 30.

## Ce qu'est un dossier complet

- nom, SIREN et activité **confirmés sur le registre public** (entreprise active) ;
- secteur, code postal, commune ; taille seulement si un registre l'indique (avec sa source) ;
- site officiel, ou absence de site vérifiée ;
- un moyen de contact **publié**, avec l'URL de la page : adresse e-mail et/ou téléphone professionnel ;
- au moins un signal vérifié et sourcé (site absent ou obsolète, non adapté au mobile, sans HTTPS, recrutement,
  projet annoncé…) ;
- une explication d'adéquation reliant ce signal à une offre ;
- chaque fait avec sa source et sa date de vérification.

Une entreprise incomplète n'est pas retenue : elle est citée dans les notes avec ce qui manque, et enregistrée comme
« écartée ».

## Les adresses e-mail : ce qui est accepté

| Accepté | Refusé |
|---|---|
| Adresse générique publiée (`contact@`, `info@`…) | Adresse **devinée** ou reconstruite (`prenom.nom@`, domaine supposé) |
| Adresse **nominative** publiée par l'entreprise sur ses pages | Outils de recherche d'e-mails ou d'enrichissement |
| **Webmail** d'artisan (Gmail, Orange, Free, Outlook…) publié comme adresse de l'entreprise | Fuites de données, profils personnels |
| | Adresse d'un salarié trouvée hors des pages de l'entreprise |

Garde-fou dans le code : une adresse non générique n'est enregistrée que si elle figure **telle quelle** dans
l'extrait recopié de la page citée (les écritures « nom [at] domaine » sont reconnues). Le nom et la fonction de la
personne sont enregistrés quand la page les donne, et le contact reçoit une finalité professionnelle (« adresse
publiée par l'entreprise pour son activité », avec l'URL). Sans adresse publiée, un téléphone professionnel suffit :
le dossier est traité par appel.

La décision est inscrite au registre de conformité, marquée « à faire valider juridiquement » pour les entrepreneurs
individuels (voir [chapitre 12](12-conformite.md)).

## Les sources

1. **Registre public des entreprises** par son API (`recherche-entreprises.api.gouv.fr`, filtre activité,
   département, entreprises actives) : SIREN, activité, effectif, dirigeants, adresse. La page web de l'annuaire
   s'affiche vide pour un robot ; l'API répond en JSON.
2. **Annuaires de métiers** où les entreprises publient leurs coordonnées : RGE / France Rénov', Qualibat (bâtiment),
   chambres de métiers, offices de tourisme et Logis (hôtellerie, restauration), unions commerciales, ordres et
   annuaires professionnels. Chaque candidat est ensuite confirmé par l'API.
3. Pour chaque candidat : site officiel (ou absence vérifiée), puis l'adresse e-mail **avant d'y renoncer** : page
   contact, pied de page, mentions légales, CGV, devis et PDF, fiche Pages Jaunes, page Facebook professionnelle,
   annuaires professionnels, recherches « <nom> <commune> e-mail ».

Interdits : comptes fictifs, contournement de CAPTCHA, de limites de débit ou de murs de connexion, non-respect des
conditions d'utilisation ou du fichier robots, collecte massive, achat de fichiers.

## La mémoire et la couverture

Table `prospect_candidates` : chaque entreprise examinée, retenue ou écartée, avec la raison et la recherche
d'origine (« plombier 40 »). Clé : SIREN, sinon nom normalisé + commune.

L'agent reçoit dans son contexte :

- `deja_examines` : les entreprises examinées depuis 60 jours (à ne pas revérifier) ;
- `couverture` : pour chaque recherche, combien d'entreprises examinées, combien retenues, et la date ;
- `objectif_candidats_nouveaux` : 30.

Consigne : privilégier les activités et les sources au meilleur rendement (retenues / examinées), puis les
combinaisons absentes ou peu examinées ; utiliser la page suivante de l'API pour un lot déjà couvert ; consulter
vraiment la fiche d'annuaire et la page Facebook d'un candidat avant de l'écarter.

## La persévérance

- Objectif chiffré : au moins 30 entreprises **nouvelles** examinées par recherche, sauf si le nombre maximal de
  dossiers est atteint.
- Relances automatiques dans la même session si l'objectif n'est pas atteint (au plus 3), voir
  [chapitre 5](05-executer-un-agent.md).
- 150 tours et 50 minutes au plus par recherche.

## Ce qu'on a mesuré

| Version | Examinées / recherche | Retenues / recherche |
|---|---|---|
| Première version (adresses génériques seulement, pas de mémoire) | non mesuré, environ 15 recherches web | 0 à 3 |
| Adresses publiées acceptées + objectif + nouvelles sources | 27 à 30 | 12 (hôtels-restaurants via offices de tourisme) |
| Même version, segments pauvres (coiffeurs, boulangeries…) | 3 à 5 (arrêt prématuré) | 0 |
| Avec relances dans la session | 31, sur 11 combinaisons | 0 à 12 selon le segment |

Enseignement : le rendement dépend surtout de **la source** (les annuaires où les entreprises publient leurs
coordonnées rendent bien mieux qu'une recherche web générique) et **du métier** (hôteliers et plombiers publient leurs
coordonnées ; coiffeurs et boulangers beaucoup moins).

## Les annonces

À chaque entreprise retenue : une annonce courte dans #assistant (« 🎯 Hop, entreprise trouvée : … », secteur,
commune, contact « ✉️ … (nominative) » ou « 📞 … (à appeler) ») et une fiche dans #prospects. À la fin de chaque
recherche : un compte rendu 🔎 avec les retenus, le nombre d'entreprises examinées, les écartés et leurs raisons.
