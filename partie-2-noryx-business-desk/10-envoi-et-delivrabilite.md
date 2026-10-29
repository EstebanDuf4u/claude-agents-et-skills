# 10. Envoi des e-mails et délivrabilité

## Le cycle de vie d'un message

```
brouillon ──▶ en_attente ──(approbation humaine)──▶ approuve ──(envoi)──▶ envoi_en_cours ──▶ envoye
                 │                                     │                                    │
                 └──────────── annule ◀────────────────┘                     (non distribué) ▼
                                                                                         echec ──▶ réapprouvable
```

- Un agent ne crée que des messages **en attente de validation**, vers un contact enregistré du prospect.
- Une personne lit le message **en entier** et l'approuve : dans le Desk (validation un par un ou en lot) ou sur
  Discord (#validation, boutons « Approuver et envoyer », « Approuver seulement », « Annuler »).
- L'approbation porte sur une **empreinte du contenu** (destinataire, objet, corps). Toute modification annule
  l'approbation ; un bouton Discord dont l'empreinte ne correspond plus est refusé.
- Un message trop long pour être affiché en entier sur Discord, ou qui a un contrôle bloquant, n'y a pas de bouton
  d'approbation.

## Les contrôles (à la création, à l'approbation et juste avant l'envoi)

| Contrôle | Niveau |
|---|---|
| Adresse invalide | bloquant |
| Destinataire exclu (adresse, domaine, sous-domaine) ou opposé | bloquant |
| Entreprise exclue, ou tous ses contacts opposés | bloquant |
| Contact nominatif sans finalité professionnelle | bloquant |
| Objet vide, corps vide ou trop court | bloquant |
| Champ « [À COMPLÉTER …] » laissé | bloquant |
| La marque n'apparaît pas dans le message | bloquant |
| Mention d'opposition absente | bloquant |
| Message identique déjà en file | bloquant |
| Déjà contacté il y a moins de 10 jours | bloquant |
| Nombre maximal de relances atteint (3) | bloquant |
| Devis joint non émis | bloquant |
| Destinataire non rattaché à un contact enregistré | alerte |
| Autres messages en file pour ce destinataire | alerte |

## L'envoi

- SMTP (STARTTLS) avec la messagerie de l'entreprise ; mot de passe hors base.
- Limite quotidienne (20 par défaut) et taille de lot (10).
- **Idempotence** : une clé par message et une contrainte d'unicité en base ; quatre envois concurrents du même
  message donnent un seul envoi.
- Un message bloqué en « envoi en cours » (arrêt pendant l'envoi) n'est **jamais renvoyé automatiquement** : un
  incident demande de vérifier dans la messagerie.
- Sans SMTP, l'envoi se fait depuis sa messagerie, puis « Marquer envoyé (depuis ma messagerie) ».
- Un premier contact réellement envoyé fait passer la fiche « Contacté ».

## « Marquer non distribué »

Un message « envoyé » peut avoir été **rejeté** après coup (avis « Failed to deliver message » du serveur d'envoi).
Le bouton « Marquer non distribué », sur la page du message, demande une preuve (« avis d'échec reçu le … ») puis :

- repasse le message en « échec », avec la raison ;
- le rend de nouveau **approuvable tel quel** pour un renvoi, sans créer de copie ;
- ajoute un événement « e-mail non distribué » à l'historique du prospect ;
- remet la fiche « À contacter » si c'était un premier contact et qu'aucun autre e-mail n'a été envoyé ;
- est réservé aux personnes (niveau « validation humaine ») et inscrit au journal.

Sans cela, le contrôle « déjà contacté il y a moins de 10 jours » bloquait le renvoi d'un message jamais arrivé.

## La délivrabilité : ce qui a été vérifié

Un e-mail de prospection qui finit en spam ne sert à rien, et abîme la réputation du domaine pour les suivants.

| Élément | Attendu | Comment vérifier |
|---|---|---|
| **rDNS** (PTR) de l'IP du serveur | le nom du serveur de mail, qui pointe lui-même vers la même IP | `dig +short -x <IP>` puis `dig +short A <nom>` |
| Nom annoncé par le serveur (bannière, HELO) | le même nom | connexion au port 25 |
| **SPF** | `v=spf1 mx -all` (ou équivalent) | `dig +short TXT <domaine>` |
| **DKIM** | clé publique publiée pour le sélecteur utilisé | `dig +short TXT <sélecteur>._domainkey.<domaine>` |
| **DMARC** | une politique (`p=quarantine` ou `p=reject`) | `dig +short TXT _dmarc.<domaine>` |
| **Listes noires d'IP** | non listée | voir ci-dessous |
| **Listes noires de domaines** | non listé | voir ci-dessous |

### Interroger Spamhaus correctement

Spamhaus **refuse les requêtes passant par un résolveur public** ou un résolveur partagé : il répond `127.255.255.254`
(ou rien), ce qui ne dit rien sur l'IP. Pour un résultat fiable, interroger directement ses serveurs de noms, et
vérifier d'abord l'adresse de test :

```bash
dig +short NS zen.spamhaus.org                          # a.gns.spamhaus.org …
dig +short @a.gns.spamhaus.org 2.0.0.127.zen.spamhaus.org     # doit répondre 127.0.0.2 … : la requête est valable
dig +short @a.gns.spamhaus.org <IP inversée>.zen.spamhaus.org # vide = non listée
dig +short @a.gns.spamhaus.org <domaine>.dbl.spamhaus.org     # vide = domaine non listé
```

Même principe pour les autres listes (Barracuda, SpamCop, UCEPROTECT, PSBL, Mailspike, DroneBL, GBUdb…) : une liste
dont l'adresse de test ne répond pas correctement donne un résultat **non concluant**, pas un « non listé ».

SURBL et URIBL (domaines cités dans les e-mails) refusent aussi les résolveurs publics ; Gmail et Microsoft n'ont pas
de liste publique et jugent l'historique d'envoi.

### Démarrer doucement

Après une remise en service : 20 e-mails par jour pendant une à deux semaines, puis augmenter progressivement
(40, puis 80…), en surveillant les avis de non-distribution et les réponses.
