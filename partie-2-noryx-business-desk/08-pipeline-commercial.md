# 8. Pipeline commercial

## Les étapes

| Étape | Clé | Qui peut y placer un prospect | Fermée |
|---|---|---|---|
| Nouveau | `nouveau` | agent ou personne | |
| À qualifier | `a_qualifier` | agent ou personne | |
| Qualifié | `qualifie` | agent ou personne | |
| À contacter | `a_contacter` | agent ou personne | |
| Contact préparé | `contact_prepare` | agent ou personne | |
| Contacté | `contacte` | **personne** (ou l'envoi réel d'un premier contact) | |
| En discussion | `en_discussion` | **personne** | |
| Devis à préparer | `devis_a_preparer` | agent ou personne | |
| Devis envoyé | `devis_envoye` | **personne** | |
| Accord à confirmer | `a_confirmer` | **personne** | |
| En attente de signature | `attente_signature` | **personne** | |
| Gagné | `gagne` | **personne**, avec confirmation explicite d'un accord écrit ou d'une signature | oui |
| Perdu | `perdu` | **personne** | oui |
| Sans suite | `sans_suite` | agent ou personne | oui |

Les étapes sont configurables (libellés, ordre, étapes fermées, réservées à une personne). Une étape utilisée ne peut
pas être supprimée.

## Les passages de relais automatiques

| Événement | Effet |
|---|---|
| Le Prospector retient une entreprise | Fiche « À qualifier », score calculé, contact publié enregistré |
| …sans adresse e-mail mais avec un téléphone | Tâche « Appeler … au … » |
| Le Qualifier recommande « Qualifié » et la fiche a une adresse e-mail publiée | **« À contacter »** |
| …sans adresse e-mail | Reste « Qualifié » (traitement par appel) |
| Fiche « À contacter » avec contact joignable et aucun message en cours | Le Copywriter rédige un premier message |
| Premier contact réellement envoyé | « Contacté » |
| Premier contact marqué « non distribué » (et aucun autre envoi) | Retour à « À contacter » |
| Inbox Triage détecte une demande de projet | Fiche, opportunité, « Devis à préparer », tâche prioritaire |
| Fiche sans prochaine action | Tâche « Définir la prochaine action » (hebdomadaire) |
| Devis envoyé | Tâches de relance à J+5 et J+12 |
| Premier contact sans réponse | Tâche de relance à J+7 |

> Le relais « Qualifié → À contacter » a été ajouté après avoir constaté que les fiches restaient bloquées : le
> Qualifier recommandait « Qualifié » avec la prochaine action « préparer un premier message », mais le Copywriter ne
> traite que « À contacter ». Voir le [journal](15-journal-des-evolutions.md).

## Le score

Sur 100, recalculé à chaque modification et justifié ligne à ligne :

| Critère | Points |
|---|---|
| Signal vérifié avec source | +15 chacun, 45 au plus |
| Hypothèse non vérifiée | +5 chacune, 15 au plus |
| Correspond à une offre confirmée (sinon « à confirmer ») | +15 (sinon +5) |
| Site officiel identifié | +5 |
| SIREN connu | +5 |
| Données vérifiées depuis moins de 90 jours | +10 |
| Contact professionnel exploitable (adresse générique ou finalité renseignée) | +10 |
| Secteur ciblé | +10 |
| Information manquante à obtenir | −3 chacune, −15 au plus |
| Opposition, exclusion ou secteur exclu | score **0** |

## Les tâches

Types : relance, qualification, devis, admin, vérification, RGPD, autre. Les tâches créées automatiquement ont une
clé d'origine : relancer une détection ne crée jamais de doublon.
