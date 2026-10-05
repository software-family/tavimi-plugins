---
name: cadrage
description: Mène l'entretien de cadrage avec un chef d'entreprise, un dirigeant d'association ou un manager pour organiser Tavimi chez lui — organisation, espaces, membres et invités, statuts de tâche, postes, catégories de notes, documentations — puis propose le plan et, après accord, crée espaces, statuts, postes, catégories et documentations ; seuls membres et invitations restent à l'interface. À utiliser quand l'utilisateur veut « mettre en place Tavimi », « organiser les espaces », « cadrer », « onboarder une entreprise ou une association », ou préparer un pilote. Ne pas utiliser pour un espace déjà en service (tavimi:questions pour clarifier ses tâches).
argument-hint: "[nom de la structure] [ce qu'on sait déjà d'elle]"
---

# Cadrer l'organisation d'une structure dans Tavimi

Le but : qu'à la fin de l'entretien, tu puisses dire à la personne quels espaces créer, qui y
entre, comment les tâches y avancent et ce que les agents y écrivent — et qu'elle soit d'accord.
Ton interlocuteur dirige ou anime une équipe ; il ne connaît pas Tavimi et n'a pas à en apprendre
le vocabulaire. Tu poses des questions sur **son** travail, tu traduis toi-même en espaces,
statuts et catégories.

Structure visée : `$ARGUMENTS`. Commence par `list_spaces` : si des espaces existent déjà pour
elle, pars de l'existant et ne propose que ce qui manque.

## 1. Conduire l'entretien

- **Un thème à la fois**, trois questions au plus par message, dans l'ordre ci-dessous. Quand une
  réponse appelle un choix fermé, propose les options avec `AskUserQuestion` (ta recommandation
  en premier) ; sinon, question ouverte en une phrase.
- **Des exemples de son monde**, pas du produit : « Qui doit pouvoir lire le budget ? » plutôt
  que « Quels espaces sont privés ? ».
- Ce que l'argument ou un document fourni dit déjà n'est pas redemandé : reformule-le en une
  ligne et demande seulement de confirmer.
- Une réponse vague (« tout le monde voit tout ») se vérifie une fois par un cas concret (« même
  les partenaires extérieurs ? les comptes ? »), puis tu avances.
- Pas plus de six thèmes : si la personne n'a pas d'avis, prends le défaut indiqué et dis-le.

## 2. Les thèmes, et ce que chacun décide

| # | Ce que tu demandes | Ce que tu en tires | Défaut si pas d'avis |
|---|---|---|---|
| 1 | La structure : activité, taille, entités séparées (filiales, antennes) | Une **organisation** par entité | Une seule organisation |
| 2 | Les grands chantiers, clients, événements ou services ; ce qui ne doit **pas** circuler de l'un à l'autre | Un **espace** par périmètre de confidentialité ou de responsabilité ; un espace fermé pour ce qui est sensible (budget, RH, conventions) | Un espace par chantier en cours + un espace de direction fermé |
| 3 | Qui travaille sur quoi ; qui décide ; qui vient de l'extérieur (client, partenaire, intervenant) | **Propriétaire** de chaque espace, **membres**, **invités extérieurs** limités à un espace | Le dirigeant propriétaire de tout |
| 4 | Comment une affaire avance, de l'idée au bilan ; à quel moment on la considère terminée | Les **statuts de tâche** de chaque espace, et lequel termine la tâche | À faire, En cours, Fait |
| 5 | Les fonctions qui reçoivent du travail, indépendamment des personnes (et le turnover) | Les **postes** (« Programmation », « QA ») | Aucun poste : on assigne aux personnes |
| 6 | Ce qu'on écrit et relit souvent (comptes rendus, décisions, bilans, procédures) ; ce qu'un nouvel arrivant devrait lire en premier | Les **catégories de notes** et leur gabarit ; les **documentations de synthèse** qu'elles nourrissent | Une catégorie « Décision », une documentation « Guide » |

Termine par deux questions sur les agents : **ce qu'ils font** (préparer des brouillons, tenir
les bilans, trier, coder…) et **ce qu'ils ne décident jamais seuls** (publier, engager de
l'argent, répondre à un client). La seconde liste devient la règle : ces points passent par
`ask_questions`, et un humain répond dans l'interface web.

## 3. Proposer le plan

Avant de créer quoi que ce soit, présente le plan dans le terminal et demande un accord explicite :

1. **Organisation** et, pour chaque **espace** : nom, une phrase de description, propriétaire,
   membres, invités extérieurs.
2. Par espace : **statuts** (dans l'ordre, celui qui termine marqué), **postes**, **catégories**
   avec leur gabarit en une ligne, **documentation** éventuelle.
3. **Règles des agents** : ce qu'ils font, ce qui passe par une question.
4. **Pilote** : un seul espace, une catégorie, une première tâche réelle — on étend après une
   semaine d'usage.

Exemple de plan pour une association qui organise des rencontres autour du numérique :

> **Organisation** `asso-numerique`. **Espaces** : Rencontres (soirées mensuelles, ateliers),
> Festival annuel (avec intervenants et lieux partenaires en invités extérieurs),
> Communication (lettre d'information, réseaux), Bureau (fermé : budget, conventions).
> **Statuts de Rencontres** : Idée → Intervenant trouvé → Lieu calé → Annoncé → Bilan fait (termine).
> **Postes** : Programmation, Lieux, Communication — un bénévole qui passe la main ne réassigne rien.
> **Catégorie** « Bilan de rencontre » : thème, intervenant, lieu, participants, ce qui a marché,
> à refaire ou pas. **Documentation** « Guide de l'organisateur », nourrie des bilans.
> **Agents** : préparent les questions de table ronde, le message à l'intervenant, la lettre
> d'information ; ne publient ni n'envoient rien sans réponse d'un membre du bureau.
> **Pilote** : l'espace Rencontres, la catégorie Bilan, la prochaine soirée comme première tâche.

## 4. Créer, après accord seulement

Avec les tools MCP, seulement ce que la personne a validé, **dans cet ordre** (chaque étape sert
la suivante). Relis `list_spaces` avant de commencer : il donne les identifiants des espaces et
des statuts, et ce qui existe déjà ne se recrée pas.

1. **Espaces** : `create_space(organization, name, description)`. L'organisation doit exister ;
   sinon, demande de la créer dans l'interface et attends.
2. **Statuts** : un nouvel espace en a trois (à faire, en cours, terminée). **Renomme**-les
   (`update_task_status`) plutôt que d'en créer et d'archiver : ils gardent leur identifiant, et
   un statut porté par une tâche ne s'archive pas. Ajoute les autres (`create_task_status`, qui
   les place en dernier, après celui qui termine), puis range-les avec `reorder_task_statuses` :
   `expected_order` est l'ordre que tu viens de lire. Le statut qui termine est `done: true`.
3. **Postes** : `create_task_post(space, name, label)`, un nom court et figé (`bar`,
   `partenaires`) et le libellé lu par les humains.
4. **Catégories** : `create_category(space, name, summary, guidance, required_sections)`. Le
   gabarit décidé pendant l'entretien devient `required_sections` ; `guidance` dit comment
   l'écrire (faits, pas d'impressions ; où va quoi).
5. **Documentations** : `create_document`, d'un modèle (`list_document_templates`) ou avec un
   `outline` et une `guidance` tirés de l'entretien ; `source_categories` nomme les catégories
   créées à l'étape 4.
6. **Note « Cadrage »** dans chaque espace (`create_note`) : les décisions de l'entretien —
   périmètre, statuts, postes, catégories, règles des agents — et leur raison ; c'est ce que
   liront les agents suivants. Range-la dans la catégorie `decision` si l'espace l'a encore (ses
   règles, `list_categories`, exigent « Pourquoi » et « Alternatives » : les options écartées
   pendant l'entretien y vont).
7. **Catégories par défaut** (`decision`, `piege`, `reference`) : archive-les
   (`archive_category`) seulement si le plan l'a prévu, et **après** la note « Cadrage ». Archiver
   la dernière catégorie active rend la catégorie facultative dans `create_note`.
8. **Première tâche** du pilote (`create_task`), si elle a été donnée, confiée à son poste.

Ces tools sont réservés au **propriétaire** de l'espace. Un refus « Réservé au propriétaire »
signifie que la connexion n'est pas la sienne : arrête-toi là pour cet espace et mets ce qui
reste dans la liste à cocher, avec les valeurs exactes. De même si un tool manque sur le serveur
(version antérieure) : l'étape va à la liste à cocher.

Restent à l'interface web, par le propriétaire : **membres et invitations** (onglet Membres de
l'espace et de l'organisation), et l'**ordre** des catégories et des postes s'il compte. Ne
prétends pas les avoir faits : donne-les comme une liste à cocher, espace par espace.

## 5. Récapituler

Termine par un tableau, d'après les réponses des tools (pas d'après le plan) :

| Espace | Créé | Statuts | Postes | Catégories | À faire dans l'interface |
|---|---|---|---|---|---|
| nom | oui / existait | ordre final | noms | noms | N invitations |

Puis rappelle comment brancher un agent sur un seul espace : au consentement OAuth, ne cocher que
les espaces dont il a besoin.
