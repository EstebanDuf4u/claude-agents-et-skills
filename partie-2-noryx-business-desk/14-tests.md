# 14. Tests

```bash
.venv/bin/python -m pytest      # 204 tests, environ 6 secondes
```

## Le principe : un faux exécutable `claude`

Aucun test n'appelle le modèle. Le fichier `tests/conftest.py` crée un script qui se fait passer pour `claude` :

- il enregistre chaque appel (arguments) dans un journal ;
- il répond à `claude auth status` par une connexion `claude.ai` ;
- il rejoue une réponse préparée (`stdout`, `stderr`, code de sortie), ou **une suite de réponses** pour tester les
  relances dans la même session.

```python
def test_prospector_follow_up_in_same_session_until_no_progress(conn, cfg, fake_claude):
    enable_ai(conn)
    fake_claude.many(premiere_passe, deuxieme_passe, deuxieme_passe)   # la 3e n'apporte rien : arrêt
    r = runner.run_agent(conn, cfg, "prospector")
    calls = [c for c in fake_claude.calls() if c[:1] == ["-p"]]
    session = calls[0][calls[0].index("--session-id") + 1]
    assert len(calls) == 3 and all(c[c.index("--resume") + 1] == session for c in calls[1:])
    assert db.scalar(conn, "SELECT COUNT(*) FROM agent_runs WHERE status != 'ignore'") == 1   # une seule exécution
```

On teste ainsi, sans consommation : la commande exacte (jamais `--bare`, `--permission-mode dontAsk`, outils
attendus), les refus de facturation, les deux formats de sortie, la validation stricte, les incidents de quota, et
l'application à la base.

Les données de test sont **synthétiques** : domaines `.invalid`, noms préfixés `[TEST]`, fiches `is_synthetic = 1`
(exclues des indicateurs et des annonces Discord).

## Ce qui est couvert

| Fichier | Tests | Sujet |
|---|---|---|
| `test_agents.py` | 25 | Exécuteur : commande, facturation, authentification, quotas, formats de sortie, simulation, relances en session, priorité |
| `test_mail.py` | 24 | Messages : contrôles, approbation, invalidation, exclusions, oppositions, idempotence, envoi concurrent, non-distribution |
| `test_money.py` | 16 | Calculs de devis : remises, TVA, arrondis, flottants refusés |
| `test_inbox.py` | 15 | Relève IMAP en lecture seule (faux serveur), tri, oppositions |
| `test_web.py` | 15 | Écrans, connexion, CSRF, Host/Origin, rôles, cookies, parcours |
| `test_discord.py` | 13 | Autorisations, publication sans doublon ni rétroactivité, boutons liés à l'empreinte, annonces, notes Claude Code |
| `test_scheduler.py` | 12 | Idempotence, verrous, reprise après erreur, pause, un agent IA par passage |
| `test_outreach.py` | 10 | Chaîne de prospection : adresses publiées / devinées, téléphone seul, mémoire des candidats, relais vers « À contacter » |
| `test_crm.py` | 10 | Prospects, doublons, contacts |
| `test_estimates.py` | 10 | Estimation PERT, marges, arrondis, prix calculé |
| `test_panel.py` | 9 | Panneau Discord : états, prochain passage, quotas, plage horaire, droits admin |
| `test_cli.py` | 7 | Commandes, diagnostic |
| `test_quotes.py` | 7 | Devis, PDF (aucun coût ni marge sur le document client) |
| `test_assistant.py` | 5 | Assistant : instantané, actions autorisées, refus |
| `test_integrity.py` | 5 | Aucune dépendance `anthropic`, aucune donnée personnelle dans les fichiers livrés, aucun chiffre inventé |
| `test_compliance.py` | 3 | Export, effacement RGPD, purge |

## Ce que les tests ne couvrent pas

- Le comportement réel du modèle (pertinence des recherches, qualité des messages) : il se mesure sur les exécutions
  réelles (nombre d'entreprises examinées, retenues, raisons d'écart).
- La délivrabilité réelle des e-mails.
- L'affichage dans Discord (les embeds et menus sont construits et vérifiés, mais pas affichés).
- La validation juridique.
