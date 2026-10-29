# 6. De la sortie de l'agent à la base

**Un agent n'écrit jamais dans la base.** Il rend un JSON. Le module `desk/agents/ingest.py` le valide, puis
l'applique avec l'acteur `agent:<nom>`, qui porte un plafond de droits. C'est la frontière de sécurité principale du
système.

```
sortie JSON ──▶ schéma pydantic strict ──▶ ingest.apply(agent, sortie, simulation?)
                                              │
                                              ├─ contrôles métier (sources, doublons, extraits…)
                                              ├─ services métier (crm, mail, quotes…) avec acteur « agent:<nom> »
                                              │     └─ permissions.require(acteur, action) → plafond niveau 3
                                              └─ rapport : [{op, resultat: applique|simule|refuse, detail, raison}]
```

## Les niveaux d'autorisation

`desk/permissions.py` définit cinq niveaux et associe chaque action à un niveau :

| Niveau | Actions |
|---|---|
| 1. Lecture et analyse | `lire` |
| 2. Préparation | `message.brouillon`, `devis.brouillon`, `rapport.generer` |
| 3. Modification de données internes | `prospect.creer`, `prospect.modifier`, `prospect.etape`, `prospect.qualifier`, `source.ajouter`, `contact.creer`, `tache.creer`, `opportunite.creer`, `facture.preparer` |
| 4. Validation humaine | `prospect.etape_humaine`, `prospect.supprimer`, `message.approuver`, `devis.emettre`, `devis.decision`, `facture.statut`, `conformite.modifier`, `rgpd.traiter` |
| 5. Action externe | `message.envoyer`, `offre.prix`, `parametres.modifier`, `utilisateurs.gerer` |

Un acteur `agent:*` ou `systeme:*` est plafonné au niveau 3, quel que soit le réglage. Une personne (`user:<email>`)
a les droits de son rôle (admin ou membre).

## Ce qui est contrôlé à l'application

| Contrôle | Refus |
|---|---|
| Un signal « vérifié » doit citer une URL présente dans les sources rendues | « signal vérifié sans source correspondante » |
| Doublon : même SIREN, même domaine, même nom normalisé, nom proche au même endroit | « doublon : PRS-… » |
| Coordonnées : l'URL de contact doit être une des sources | « coordonnées sans source correspondante » |
| Adresse e-mail non générique : elle doit figurer telle quelle dans l'extrait de la page citée (y compris écrite « nom [at] domaine ») | « adresse nominative absente de l'extrait de la page citée » |
| Contact nominatif créé par un agent : finalité professionnelle obligatoire | « un agent doit indiquer la finalité… » |
| Étape réservée à une personne (Contacté, Gagné…) | refus de permission |
| Message vers une adresse qui n'est pas un contact enregistré du prospect | refus |
| Devis : prix du catalogue imposé ; sans tarif, ligne « hypothèse » | — |

Chaque refus est consigné dans le rapport de l'exécution, visible dans l'interface, et n'interrompt pas le reste.

## Effets de bord utiles

- **Prospect sans adresse e-mail mais avec un téléphone** : contact enregistré avec le téléphone et tâche « Appeler
  <entreprise> au <numéro> : aucune adresse e-mail publiée ».
- **Candidats examinés** : enregistrés (retenus ou écartés, avec la raison et la recherche d'origine) pour que la
  recherche suivante ne les revérifie pas.
- **Qualification « Qualifié » + adresse e-mail publiée** : étape « À contacter », pour que le Copywriter prenne le
  relais.
- **Score** recalculé après chaque modification, justifié ligne à ligne (signaux vérifiés, hypothèses, offre confirmée
  correspondante, site connu…) ; 0 en cas d'opposition ou de secteur exclu.

## Le mode simulation

Avec le mode simulation (actif par défaut à l'installation), `ingest.apply(..., dry_run=True)` ne modifie rien : le
rapport indique « simulé ». Une personne relit l'exécution dans **Agents et exécutions** puis « Appliquer après
relecture » ; les droits restent ceux de l'agent, et une même exécution ne peut pas être appliquée deux fois.

## Ajouter un agent

1. `.claude/agents/noryx-<nom>.md` (en-tête `name`, `description`, `tools`, `maxTurns`).
2. Schéma de sortie dans `desk/agents/schemas.py` (`extra="forbid"`).
3. Contexte, consigne de tâche, travail en attente dans `AGENTS` (`desk/agents/registry.py`).
4. Fonction d'application dans `desk/agents/ingest.py`, toujours avec l'acteur `agent:<nom>`.
5. Tâche planifiée `ai_<nom>` dans `desk/settings.py` et `desk/scheduler.py` (désactivée par défaut) et sa place dans
   la priorité des agents.
6. Tests avec le faux exécutable `claude` (aucune consommation réelle).
