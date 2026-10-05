# Plugins Claude Code de Tavimi

[Tavimi](https://tavimi.com) conserve la mémoire de travail de vos agents IA : notes, tâches et
documentations, rangées par espace et partagées avec votre équipe.

## Plugin `tavimi`

Le plugin apporte deux choses : les **signaux des agents** et des **skills** pour piloter un
espace Tavimi depuis Claude Code. Il se met à jour par le marketplace, comme tout plugin.

### Signaux des agents

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

### Skills

| Skill | Ce qu'il fait |
|---|---|
| `/tavimi:cadrage` | Mène l'entretien de cadrage avec un dirigeant ou un manager, propose l'organisation (espaces, statuts, postes, catégories, documentations) et la crée après accord. |
| `/tavimi:questions` | Passe en revue les tâches ouvertes d'un espace et pose, avec `ask_questions`, les questions qui lèvent leurs ambiguïtés avant le code. |
| `/tavimi:tache-suivante` | Prend une tâche prête et la mène jusqu'à une pull request ouverte, ou pose les questions qui manquent. Ne fusionne jamais. |
| `/tavimi:boucle` | Enchaîne `tache-suivante` sans surveillance, une tâche par tour, jusqu'à ce qu'il n'y ait plus rien à prendre. Demande le plugin `ralph-loop` (`/plugin install ralph-loop@claude-plugins-official`). |

Chacun prend l'identifiant de l'espace (le champ `id` que donne `list_spaces`) ; sans lui, il
cherche l'espace du dépôt courant. Les réponses aux questions se donnent dans l'onglet Questions
de l'espace, jamais par un agent.

> **Attention à la boucle.** `tache-suivante` et `boucle` font implémenter par un agent le texte
> des tâches, jusqu'à pousser une branche et ouvrir une pull request, avec vos droits git. Dans un
> espace partagé, une tâche écrite par un autre membre devient une consigne pour cet agent. Ne
> lancez la boucle que sur un espace dont vous connaissez tous les contributeurs, et relisez
> chaque pull request : la boucle n'en fusionne aucune.

## Licence

Ce dépôt est distribué sous la [licence Apache 2.0](LICENSE) : vous pouvez l'utiliser, le modifier
et le redistribuer, y compris commercialement. Toute redistribution, modifiée ou non, doit garder le
fichier [NOTICE](NOTICE), qui cite **Elie Terrien (Tavimi) — https://tavimi.com**.
