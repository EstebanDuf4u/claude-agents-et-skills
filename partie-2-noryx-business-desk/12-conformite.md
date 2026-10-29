# 12. Conformité et RGPD

> Ce chapitre décrit des **garde-fous techniques**. Il ne constitue pas un avis juridique. Les points marqués ⚖ sont à
> faire valider par un professionnel avant les campagnes.

## Les principes appliqués par l'application

| Exigence | Mise en œuvre |
|---|---|
| Finalité professionnelle | Champ « finalité » par contact ; un contact nominatif sans finalité ne peut pas recevoir de message |
| Traçabilité | Chaque fiche a au moins une source datée ; chaque fait « vérifié » est rattaché à une source ; chaque adresse nominative à la page et à l'extrait où elle est publiée |
| Minimisation | Champs strictement professionnels ; pas de champ pour une adresse ou un téléphone personnels |
| Adresses des prospects | Générique, nominative ou webmail, seulement si l'entreprise l'a publiée elle-même pour son activité ; jamais devinée ; pas d'outil de recherche d'e-mails |
| Conservation limitée | Fiches sans échange depuis 36 mois proposées à la purge (tâche hebdomadaire) ; aucune suppression automatique |
| Droits des personnes | Écran Conformité : accès (export JSON), rectification, effacement (adresse conservée en exclusion), opposition ; échéance à 30 jours avec rappel |
| Opposition | Un clic, ou « STOP » en tête d'une réponse : contact marqué, adresse exclue, messages non envoyés annulés ; contrôlé à l'approbation **et** juste avant l'envoi |
| Listes d'exclusion | Adresses et domaines (sous-domaines inclus) |
| Identification | Contrôle bloquant : la marque doit figurer dans le message |
| Moyen d'opposition simple | Mention ajoutée à chaque message ; message bloqué si elle est retirée |
| Contrôle humain | Aucun envoi sans approbation individuelle du contenu exact |
| Fréquence | Délai minimal entre deux envois au même destinataire, nombre maximal de relances, limite quotidienne |
| Achat de fichiers / enrichissement payant | Absent ; interdit sauf décision explicite au registre |
| Sources consultées par l'agent | Publiques, sans compte fictif ni contournement de protection ; contenus traités comme non fiables |

## Le registre des décisions

L'écran **Sources et conformité** contient un registre, initialisé à l'installation et complété au fil des décisions :

| Sujet | Décision | ⚖ |
|---|---|---|
| Prospection B2B par e-mail | Adresses professionnelles, objet en lien avec l'activité, identification de la marque, opposition simple | oui |
| Adresses nominatives et webmails | Enregistrés seulement si publiés par l'entreprise pour son activité, avec la page et l'extrait ; jamais d'adresse devinée ; sans adresse, appel téléphonique | oui (entrepreneurs individuels) |
| Validation humaine | Aucun envoi sans approbation individuelle du contenu exact ; toute modification l'annule | |
| Traçabilité | Prospect et contact rattachés à une source datée | |
| Minimisation | Données professionnelles utiles seulement | |
| Conservation | 36 mois sans échange, purge proposée | oui |
| Achat de fichiers / enrichissement payant | Interdit sans décision explicite | |
| Sources | Pas de contournement, pas de comptes fictifs, contenus non fiables | |
| Information des personnes | Mention d'information et d'opposition ; réponse aux demandes sous 30 jours | oui |

## Les points à valider juridiquement ⚖

- La prospection par e-mail entre professionnels sans consentement préalable est généralement admise en France si le
  message est en rapport avec la fonction ou l'activité de la personne et comporte un moyen d'opposition. À vérifier
  pour chaque type de cible : adresses nominatives, **entrepreneurs individuels** (dont l'adresse peut être un webmail
  personnel utilisé pour l'activité), professions réglementées.
- Le texte exact de la mention d'information (identité, finalité, droits, contact).
- La durée de conservation retenue.
- La base légale (intérêt légitime) et sa documentation.
- Toute campagne hors de France.
- Le transit des données par Discord.

Une adresse professionnelle publiée n'autorise pas automatiquement tous les usages commerciaux : en cas de doute, ne
pas approuver le message.
