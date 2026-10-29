# 7. Hooks, plugins, MCP et channels

Ces briques étendent Claude Code au-delà des agents et des skills. Ce chapitre en donne l'essentiel ; la
documentation officielle détaille chaque option.

## Hooks : les automatismes garantis

Un **hook** est une commande que Claude Code exécute lui-même sur un événement : avant ou après un outil
(`PreToolUse`, `PostToolUse`), au début ou à la fin d'une session, quand Claude s'arrête (`Stop`)… Le modèle ne peut
pas l'« oublier » : c'est Claude Code qui l'exécute.

Usages typiques : formater un fichier après chaque édition, bloquer une commande dangereuse, valider qu'une requête
SQL est en lecture seule, notifier la fin d'une tâche.

```json
{
  "hooks": {
    "PreToolUse": [
      { "matcher": "Bash", "hooks": [ { "type": "command", "command": "./scripts/verifier-commande.sh" } ] }
    ]
  }
}
```

Règle simple : **si quelque chose doit arriver à chaque fois, c'est un hook, pas une consigne.** Une consigne dans
`CLAUDE.md` est suivie la plupart du temps ; un hook l'est toujours.

## Plugins : des paquets réutilisables

Un **plugin** regroupe skills, sous-agents, hooks et serveurs MCP. Il s'installe par `/plugin install
<nom>@<marketplace>` et ses skills sont appelés `/<plugin>:<skill>`. Les sous-agents de plugins ne peuvent pas définir
de hooks, de serveurs MCP ni de mode de permission (sécurité).

## MCP : des outils externes

Le **Model Context Protocol** permet de brancher des serveurs d'outils (base de données, ticketing, API interne…),
déclarés dans `.mcp.json`. En exécution automatisée, `--strict-mcp-config` évite qu'un serveur déclaré dans le dossier
ne soit chargé à votre insu.

## Channels : piloter une session depuis Discord, Telegram ou iMessage

Les **channels** (fonction en avant-première, « research preview ») sont des serveurs MCP qui **poussent** des
messages dans une session Claude Code déjà ouverte. Avec le plugin Discord officiel, vous écrivez au bot depuis votre
téléphone : le message arrive dans la session, Claude travaille sur vos vrais fichiers et répond dans Discord.

Mise en place (Discord) :

1. Créer un bot dans le portail développeur Discord, copier son jeton, activer **Message Content Intent**, l'inviter
   sur le serveur (permissions : voir les salons, envoyer des messages, lire l'historique, joindre des fichiers,
   ajouter des réactions).
2. Installer [Bun](https://bun.sh) (les plugins de channels sont des scripts Bun).
3. Dans Claude Code : `/plugin install discord@claude-plugins-official`, puis `/discord:configure <jeton>`.
4. Relancer : `claude --channels plugin:discord@claude-plugins-official`.
5. Écrire au bot en message privé, récupérer le code d'appairage, puis `/discord:access pair <code>` et
   `/discord:access policy allowlist`.

Points d'attention :

- Les messages n'arrivent **que tant que la session est ouverte** (terminal persistant ou processus en arrière-plan).
- Une demande d'autorisation met la session en attente jusqu'à votre réponse, sauf si le channel sait relayer les
  autorisations.
- Toute personne de la liste autorisée peut piloter la session : n'y mettre que vous.
- Utiliser **un bot distinct** de celui de votre application, pour que les deux ne se gênent pas.

Autres moyens de piloter à distance : **Remote Control** (piloter une session locale depuis claude.ai ou l'application
mobile), **Claude dans Slack**, ou les **sessions cloud**.
