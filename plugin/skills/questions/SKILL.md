---
name: questions
description: Passe en revue les tâches ouvertes d'un espace Tavimi et pose, avec ask_questions, les questions qui lèvent leurs ambiguïtés avant toute implémentation. À utiliser dès que l'utilisateur veut « débloquer », « clarifier », « préparer » ou « lever les ambiguïtés » des tâches Tavimi, faire poser des questions sur le backlog, ou vérifier que les tâches sont prêtes à être prises — même s'il ne cite pas ask_questions. Ne pas utiliser pour implémenter une tâche ni pour répondre aux questions (les réponses se donnent dans l'interface web).
argument-hint: "[identifiant d'espace] [tâches, catégorie ou statut à cibler]"
---

# Lever les ambiguïtés des tâches Tavimi

Le but : qu'une fois les réponses données, chaque tâche ouverte puisse être implémentée sans
deviner. Tu prépares le terrain, tu n'implémentes rien. Une question bien posée coûte trente
secondes à l'humain ; une hypothèse silencieuse coûte une implémentation à refaire.

Cible : `$ARGUMENTS` ; l'espace se désigne par son identifiant (`id` de `list_spaces`, ex.
`k3x9p2m4ta`). Sans espace indiqué, appelle `list_spaces` et prends celui qui correspond au dépôt
courant ; s'il y a un doute, demande lequel avant de commencer.

## 1. Rassembler

- `list_tasks` sur l'espace, sans filtre : ce sont les tâches ouvertes. Restreins-toi à ce que
  l'argument désigne (tâches, catégorie, statut) s'il en désigne.
- `get_task` sur chacune : description, journal, questions déjà posées et leurs réponses. Ne
  repose jamais une question déjà posée, qu'elle ait une réponse ou non.
- Les sources qui répondent sans déranger personne : le code du dépôt, les specs et plans
  (par exemple `docs/superpowers/specs/` et `docs/superpowers/plans/`, s'ils existent), `CLAUDE.md`, et les
  documentations de l'espace (`list_documents` / `get_document`). Lis ce qui touche la tâche,
  pas tout.

## 2. Repérer ce qui manque

Pour chaque tâche, demande-toi ce qu'un développeur compétent, qui ne connaît que le dépôt et
la tâche, devrait inventer pour la terminer. Pistes fréquentes :

- périmètre : ce qui est dedans, ce qui est explicitement dehors ;
- comportement visible : texte, écran, message d'erreur, ce qu'on voit quand c'est vide ;
- droits : qui peut faire quoi (propriétaire, membre, agent, interface web) ;
- cas limites : archivage, conflit de version, données existantes, migration et bascule ;
- arbitrages produit ou UX que le code ne peut pas trancher ;
- contradictions entre la tâche, la spec et le code actuel.

Ce qui se tranche par le code, la spec ou une convention établie n'est **pas** une question :
note plutôt ta conclusion et sa source avec `add_task_log`, pour que le prochain agent n'ait
pas à refaire l'enquête. Si une tâche est trop grosse ou en recouvre une autre, c'est aussi une
question (découper ? fusionner ?).

## 3. Poser les questions

Un appel `ask_questions` par tâche, avec toutes ses questions. Pour chaque question :

- **une seule décision** : deux décisions dans une question donnent une réponse à moitié ;
- **autoportante** : l'humain répond depuis l'interface web, souvent sans le code sous les yeux.
  Donne le contexte utile en une ou deux phrases (ce qui existe, ce qui coince) ;
- **des options concrètes** quand c'est possible, dans le champ `options` (2 à 8, chacune avec
  une `description` d'une phrase qui dit ce qu'elle implique), et **ta recommandation** par
  `recommended` (indice), placée en premier : cliquer l'option recommandée doit suffire. Une
  question ouverte (sans options) seulement quand il s'agit d'un fait que tu ne peux pas
  connaître (« as-tu mesuré… ? ») ;
- `multiple: true` quand plusieurs choix se cumulent ;
- en texte brut, courte ; pas de question rhétorique ni de « es-tu sûr ».

Exemple :

> text : L'export CSV des tâches inclut-il les tâches archivées ? Aujourd'hui `list_tasks` les
> exclut et l'interface n'a aucun filtre pour les montrer.
> options : « Jamais » (l'archivage vaut retrait partout ailleurs) ; « Case à cocher, décochée
> par défaut » (un filtre de plus à maintenir).
> recommended : 0

Une tâche déjà claire ne reçoit pas de question : écris dans son journal qu'elle est prête et
sur quoi tu t'appuies.

## 4. Ne pas franchir la ligne

- Aucune implémentation, aucune modification du dépôt.
- Ne change pas le statut des tâches et ne réécris pas leur description (`update_task`) :
  les questions et le journal suffisent, et l'humain garde la main sur le contenu.
- Aucun tool MCP ne répond aux questions, c'est voulu : un agent ne débloque pas sa propre
  boucle. N'essaie pas de contourner.

## 5. Récapituler

Termine par un tableau dans le terminal :

| Tâche | État | Questions posées |
|---|---|---|
| titre | prête / bloquée | N (sujets en quelques mots) |

Puis rappelle que les réponses se donnent dans l'onglet Questions de l'espace, et qu'ensuite
`list_tasks(awaiting_answer=false)` donne les tâches qui peuvent repartir.
