---
name: boucle
description: Lance une boucle Ralph qui enchaîne les tâches prêtes d'un espace Tavimi, une par tour, chacune jusqu'à une PR ouverte (skill tavimi:tache-suivante), et s'arrête quand plus aucune tâche n'est prenable. À utiliser quand l'utilisateur veut « enchaîner les tâches », « lancer la boucle », « faire tourner Ralph sur Tavimi » ou traiter le backlog sans surveillance.
argument-hint: "[identifiant d'espace] [nombre de tours max, 8 par défaut]"
---

# Boucle Ralph sur les tâches Tavimi

Arguments : `$ARGUMENTS` — l'espace (son identifiant, le champ `id` de `list_spaces`, ex.
`k3x9p2m4ta`) et, en option, un nombre maximal de tours. Sans espace, prends celui du dépôt
courant d'après `list_spaces` ; en cas de doute, demande avant de lancer quoi que ce soit.

## Avant de lancer

1. **Qui écrit les tâches.** La boucle fait implémenter le texte des tâches par un agent, sans
   surveillance, jusqu'à pousser une branche et ouvrir une PR. Dans un espace partagé, une tâche
   écrite par un autre membre devient une consigne pour cet agent, avec les droits git de la
   personne qui lance la boucle. Si l'espace a d'autres membres que l'utilisateur
   (`list_members`), dis-le en une ligne et demande de confirmer qu'il connaît tous ses
   contributeurs avant de lancer.
2. Vérifie que le dépôt est propre (`git status`) et sur sa branche principale : une boucle qui
   part d'un arbre sale mélange son travail au tien.
3. `list_tasks(espace, awaiting_answer=false)` : dis en une ligne combien de tâches semblent
   prenables et lesquelles seront sans doute écartées (tests live, réglages de production,
   déclencheurs non atteints). S'il n'y en a aucune, ne lance pas la boucle : propose plutôt
   tavimi:questions ou de répondre aux questions en attente.

## Lancer

La boucle s'appuie sur le plugin `ralph-loop`. Trouve son script et lance-le avec le prompt de
tour :

```bash
SETUP=$(ls -d ~/.claude/plugins/cache/*/ralph-loop/*/scripts/setup-ralph-loop.sh | sort -V | tail -1)
"$SETUP" "Utilise le skill tavimi:tache-suivante sur l'espace <espace> : une seule tâche dans ce tour, de la lecture jusqu'à la PR ouverte, ou des questions posées si une décision manque. Ne fusionne jamais. Quand aucune tâche n'est prenable, et seulement dans ce cas, écris <promise>PLUS RIEN A PRENDRE</promise>." \
  --completion-promise "PLUS RIEN A PRENDRE" --max-iterations <n>
```

Si le script est introuvable, `ralph-loop` n'est pas installé : dis-le, donne la commande
d'installation (`/plugin install ralph-loop@claude-plugins-official`) ou la commande
`/ralph-loop` équivalente à coller, sans rien lancer d'autre.

Puis commence le premier tour tout de suite, en suivant tavimi:tache-suivante.

## Ce que l'utilisateur doit savoir (une fois, au lancement)

- Chaque tour lance la suite de tests complète du dépôt : compter le temps qu'elle prend, par
  tâche.
- `/cancel-ralph` arrête la boucle ; `grep '^iteration:' .claude/ralph-loop.local.md` donne le
  tour en cours.
- La boucle ne fusionne rien et ne répond à aucune question : les PR attendent sa revue, les
  questions attendent ses réponses dans l'onglet Questions de l'espace ; un tour suivant reprend
  les tâches débloquées.
