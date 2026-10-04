# Plugins Claude Code de Tavimi

[Tavimi](https://tavimi.com) conserve la mémoire de travail de vos agents IA : notes, tâches et
documentations, rangées par espace et partagées avec votre équipe.

## Plugin `tavimi` : signaux des agents

Quand un agent Claude Code s'arrête pour vous poser une question ou attend une permission, il
prévient Tavimi : une pastille « un agent vous attend » s'allume dans l'interface.

### Installation

1. Brancher Claude Code sur Tavimi en MCP, **sous le nom `tavimi`** (les hooks du plugin visent ce
   nom) :

   ```bash
   claude mcp add --transport http tavimi https://app.tavimi.com/mcp
   ```

   puis s'authentifier depuis `/mcp` dans Claude Code.

2. Ajouter ce marketplace et installer le plugin :

   ```
   /plugin marketplace add software-family/tavimi-plugins
   /plugin install tavimi@tavimi
   ```

`/hooks` doit alors montrer cinq entrées de type « MCP tool hook ».

### Ce qui est transmis

Les hooks appellent le tool MCP `signal_session` par la connexion OAuth déjà établie : aucun jeton,
aucune variable d'environnement. Seuls partent l'identifiant de session, le nom de l'évènement, le
nom de l'outil en attente et le dernier segment du dossier courant. Jamais le contenu du terminal,
ni la question, ni la commande en attente de permission.

Si Tavimi est injoignable, le hook échoue en cinq secondes sans bloquer le terminal.
