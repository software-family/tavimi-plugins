# Plugins Claude Code de Tavimi

[Tavimi](https://tavimi.com) conserve la mémoire de travail de vos agents IA : notes, tâches et
documentations, rangées par espace et partagées avec votre équipe.

## Plugin `tavimi`

Le plugin apporte des **skills** pour piloter un espace Tavimi depuis Claude Code. Il se met à
jour par le marketplace, comme tout plugin.

### Installation

1. Brancher Claude Code sur Tavimi en MCP, sous le nom `tavimi` :

   ```bash
   claude mcp add --transport http tavimi https://app.tavimi.com/mcp
   ```

   puis s'authentifier depuis `/mcp` dans Claude Code.

2. Ajouter ce marketplace et installer le plugin :

   ```
   /plugin marketplace add software-family/tavimi-plugins
   /plugin install tavimi@tavimi
   ```

Jusqu'à la version 0.3.0, le plugin installait aussi cinq hooks qui signalaient à Tavimi une
session en attente d'une question. Ils ont été retirés en 0.4.0 : Tavimi n'affiche plus ces
signaux. Mettre à jour le plugin depuis `/plugin` les retire ; `/hooks` ne doit plus montrer
d'entrée « MCP tool hook » du plugin.

### Skills

| Skill | Ce qu'il fait |
|---|---|
| `/tavimi:cadrage` | Mène l'entretien de cadrage avec un dirigeant ou un manager, propose l'organisation (espaces, statuts, postes, catégories, documentations) et la crée après accord. |
| `/tavimi:questions` | Passe en revue les tâches ouvertes d'un espace, écrit la spec de celles qui changent un comportement dans leur description, et pose avec `ask_questions` les questions qui lèvent leurs ambiguïtés avant le code. |
| `/tavimi:tache-suivante` | Prend une tâche prête : écrit d'abord sa spec dans la tâche et pose les questions qu'elle soulève, puis, une fois les réponses données, la développe jusqu'à une pull request ouverte. Ne fusionne jamais. |
| `/tavimi:boucle` | Enchaîne `tache-suivante` sans surveillance, une tâche par tour, jusqu'à ce qu'il n'y ait plus rien à prendre. Demande le plugin `ralph-loop` (`/plugin install ralph-loop@claude-plugins-official`). |
| `/tavimi:chronologie` | Met à jour la chronologie d'un espace (la frise de ce qui a été livré, décidé et appris, semaine par semaine) à partir des notes et des tâches qu'elle n'a pas encore intégrées. Demande que le propriétaire l'ait activée dans Configuration › Chronologie. |

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
