---
name: chronologie
description: Met à jour la chronologie d'un espace Tavimi — la frise de ce qui a été livré, décidé et appris, semaine par semaine — à partir des notes et des tâches qu'elle n'a pas encore intégrées. À utiliser quand l'utilisateur veut « mettre à jour la chronologie », « rédiger la frise », « résumer la semaine », ou quand la chronologie annonce des évènements en attente. Ne pas utiliser pour une documentation ordinaire (ce n'est pas une chronologie) ni pour créer la chronologie : seul le propriétaire l'active, dans l'interface.
argument-hint: "[identifiant d'espace]"
---

# Mettre à jour la chronologie d'un espace

La chronologie dit en une page, à un humain qui ne lit pas toutes les notes, ce qui a été livré,
décidé et appris dans l'espace, semaine par semaine. Tu **choisis** : une entrée par fait
marquant, souvent tirée de plusieurs notes et tâches ; rien pour le bruit. Ce n'est pas la liste
des tâches terminées.

Espace : `$ARGUMENTS`, son identifiant (`id` de `list_spaces`, ex. `k3x9p2m4ta`). Sans espace,
prends celui du dépôt courant d'après `list_spaces` ; en cas de doute, demande.

## 1. Trouver la chronologie

`list_spaces` : l'espace annonce `chronology` (ses types et `pending_count`). S'il ne l'annonce
pas, la chronologie n'est pas activée : dis-le en une phrase — son propriétaire l'active dans
Configuration › Chronologie, depuis l'interface web — et arrête-toi. Ne crée pas de documentation à
sa place : le nom `chronologie` est réservé et `create_document` le refuse.

Si `pending_count` vaut 0, la chronologie est à jour : dis-le et arrête-toi.

## 2. Lire

- `get_document(space, name="chronologie")` : son contenu, sa `version`, ses règles (`guidance`)
  et ses types (`chronology_types`). Le format est dans `guidance` ; relis-le, il est vérifié à
  chaque écriture.
- `pending_sources(space, name="chronologie")`, toutes les pages (`next_cursor`) : ce qu'elle
  n'a pas encore intégré. Pour chaque source « nouvelle » ou « modifiee », lis-la : `get_note`
  pour une note, `get_task` pour une tâche (journal compris). Une source « sortie » (archivée ou
  hors périmètre) n'a rien à lire : retire de la chronologie ce qu'elle seule justifiait.
- Garde, pour chaque source lue, la version lue (`version`) et, pour une tâche, le nombre
  d'entrées de journal (`log_count`) : tu les déclareras.

## 3. Choisir et rédiger

- **Une entrée par fait marquant** : une fonctionnalité livrée, une décision et sa raison, un
  résultat mesuré, un abandon (une tâche dont le statut abandonne la tâche, `status_abandoned`).
  Regroupe ce qui raconte la même chose ; laisse tomber ce qui ne changera rien pour un lecteur
  dans un mois. Les décisions se cachent parfois dans d'autres notes : lis, ne te contente pas
  des catégories.
- **Le type** est le libellé d'un des `chronology_types`, tel quel. Si aucun ne convient, choisis
  le plus proche et signale-le à la fin, sans inventer de type : le propriétaire les règle.
- **La date** d'une entrée est celle du fait (la décision, la livraison), pas celle de ta lecture ;
  elle range l'entrée dans sa semaine (« Semaine du » le lundi de cette date).
- **La phrase** dit ce qui s'est passé et pourquoi, pour quelqu'un qui n'a pas suivi. Des
  **chiffres** quand il y en a de vrais (mesures, volumes) ; jamais d'estimation. Les **sources** :
  `note <id>` et `tâche <id>` pour les notes et tâches de l'espace, et des références libres
  (`PR #22`).
- **Le résumé** de chaque semaine touchée : deux ou trois phrases, réécrites avec ses nouvelles
  entrées.
- **Les Repères** (la colonne latérale) : mets à jour ce qui a changé — décisions en vigueur, ce
  qu'on tient pour acquis, rendez-vous à venir, état des hypothèses. Une décision remplacée en
  sort ; un rendez-vous passé aussi.

## 4. Écrire

- **Une semaine qui existe** ou les **Repères** : `update_document(section="Semaine du AAAA-MM-JJ")`
  (ou `section="Repères"`), avec la section entière, titre compris, et rien d'une autre.
- **Une semaine nouvelle** : elle n'existe pas encore comme section ; écris la chronologie
  entière (`content` sans `section`), la nouvelle semaine à sa place (les plus récentes en haut).
- Chaque écriture passe `expected_version` (la version lue, puis celle que rend l'écriture
  précédente). Un refus de format cite la ligne en cause : corrige et réécris. Un conflit de
  version : relis avec `get_document` et réapplique.
- **Déclare tout ce que tu as lu** dans `integrated` — retenu ou non, sorties comprises — avec la
  version lue et, pour une tâche, `log_entries`. Cinquante sources au plus par appel : au-delà,
  déclare le reste par des appels sans `content`. C'est ce qui remet la fraîcheur à zéro ; une
  source lue et écartée qu'on ne déclare pas reviendra au prochain passage.

## 5. Ne pas franchir la ligne

- N'écris que la chronologie : pas de note, pas de tâche, pas de type nouveau.
- N'active ni ne désactive la chronologie, et ne renomme pas ses types : c'est au propriétaire.
- Ne recopie pas de secret ni de donnée personnelle d'une note dans la frise : elle est lue par
  tous les membres de l'espace.

## 6. Récapituler

En quelques lignes : les semaines écrites, les entrées ajoutées ou retirées (date, type, titre),
le nombre de sources déclarées, et ce qui t'a manqué (un type qui n'existe pas, une source
illisible).
