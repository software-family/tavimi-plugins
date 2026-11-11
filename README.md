# Tavimi : serveur MCP et plugin Claude Code

[Tavimi](https://tavimi.com) est la mémoire de travail partagée par votre équipe et vos assistants
IA, via MCP : notes, tâches, documentations et fichiers, rangés par espace.

*Tavimi is a shared working memory for teams and their AI assistants, served over MCP: notes,
tasks, docs and files, organised by space. Hosted in France. [English summary](#in-english) below.*

## Le serveur MCP

| | |
|---|---|
| Adresse | `https://app.tavimi.com/mcp` |
| Transport | HTTP streamable |
| Authentification | OAuth 2.1 (rien à copier : le client ouvre la page de connexion) |
| Assistants | Claude (claude.ai, Claude Desktop, Claude Code), ChatGPT, Le Chat de Mistral, et tout client MCP distant |
| Hébergement | France |
| Tarifs | gratuit pour une personne et deux espaces ; 4 € HT par personne et par mois en équipe, assistants illimités ([tarifs](https://tavimi.com/tarifs)) |

C'est un service hébergé : il n'y a rien à installer ni à faire tourner. Ce dépôt contient le
plugin Claude Code, pas le code du serveur.

Ce que vos assistants y font :

- **Notes** : écrire, relire et corriger des notes classées par catégorie, chacune avec ses règles
  d'écriture ; rien ne s'écrase, chaque modification garde la version précédente.
- **Tâches** : créer et suivre des tâches (statuts, priorités, échéances, responsables), tenir
  leur journal, et poser à un humain les questions qu'ils ne doivent pas trancher seuls.
- **Documentations** : tenir à jour des synthèses à partir des notes et des tâches, et une
  chronologie de ce qui a été livré, décidé et appris.
- **Fichiers** : déposer et retrouver des fichiers liés aux notes et aux tâches.
- **Recherche et reprise** : chercher dans tous les espaces, et savoir ce qui a changé depuis le
  dernier passage.

Plusieurs personnes et plusieurs assistants partagent le même espace : ce que Claude a noté le
matin, ChatGPT ou un collègue le retrouve l'après-midi. Chaque connexion peut être limitée à
certains espaces, ou à la lecture seule.

Pour commencer : [créer un compte](https://app.tavimi.com/), puis suivre
[Démarrer](https://tavimi.com/demarrer). Voir aussi [Comment ça marche](https://tavimi.com/comment-ca-marche)
et [Sécurité](https://tavimi.com/securite).

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

## In English

Tavimi is a hosted MCP server that gives a team and its AI assistants one shared working memory.

- **Endpoint**: `https://app.tavimi.com/mcp` (streamable HTTP, OAuth 2.1).
- **Works with**: Claude (claude.ai, Claude Desktop, Claude Code), ChatGPT, Mistral Le Chat, and
  any remote MCP client.
- **What assistants do with it**: write and read versioned notes, create and track tasks, ask
  humans the questions they should not decide alone, maintain summary docs and a timeline, store
  files, search across spaces, and see what changed since their last visit.
- **Sharing**: several people and several assistants work in the same space; each connection can
  be limited to chosen spaces or to read-only.
- **Hosting and pricing**: hosted in France; free for one person and two spaces, then €4 excl. VAT
  per person per month, with unlimited assistants.

Claude Code:

```bash
claude mcp add --transport http tavimi https://app.tavimi.com/mcp
```

This repository holds the Claude Code plugin (skills that drive a Tavimi space), not the server
code. The interface and the skills are in French.

## Licence

Ce dépôt est distribué sous la [licence Apache 2.0](LICENSE) : vous pouvez l'utiliser, le modifier
et le redistribuer, y compris commercialement. Toute redistribution, modifiée ou non, doit garder le
fichier [NOTICE](NOTICE), qui cite **Elie Terrien (Tavimi) — https://tavimi.com**.
