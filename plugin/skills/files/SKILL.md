---
name: files
description: Dépose un fichier dans un espace Tavimi — rapport, export, image, PDF, archive — en choisissant la bonne voie : save_file pour un texte, create_file_upload puis curl pour un fichier binaire local, sans jamais recopier son base64 à la main. À utiliser quand l'utilisateur veut « déposer », « ranger », « envoyer » ou « joindre » un fichier dans Tavimi, ou quand un agent veut y garder un artefact qu'il a produit. Ne pas utiliser pour une note (create_note) ni pour une documentation (update_document).
argument-hint: "[identifiant d'espace] [chemin du fichier]"
---

# Déposer un fichier dans un espace

Arguments : `$ARGUMENTS` — l'espace (son identifiant, `id` de `list_spaces`, ex. `k3x9p2m4ta`) et
le fichier. Sans espace, prends celui du dépôt courant d'après `list_spaces` ; en cas de doute,
demande.

Un fichier vaut 10 Mo au plus. Son nom est unique dans l'espace : renvoyer le même nom en dépose
une **version** suivante, jamais un écrasement.

## 1. Choisir la voie

- **Un texte** (Markdown, HTML, CSV, JSON, code…) que tu as en main : `save_file` avec
  `encoding="text"` et le contenu tel quel.
- **Un fichier binaire local** (image, PDF, tableur, archive…) et un shell : **lien d'envoi**
  (section 2). Ne recopie jamais un base64 dans `content` : la recopie d'une longue chaîne n'est
  pas fiable, et le fichier arrive corrompu.
- **Un binaire sans shell** (le base64 vient d'un outil, pas de ta recopie) : `save_file` avec
  `encoding="base64"` **et** `sha256`, l'empreinte du fichier d'origine ; un contenu qui ne lui
  correspond pas est refusé, rien n'est déposé.

## 2. Envoyer un fichier local

1. Calcule l'empreinte et la taille :

   ```bash
   sha256sum <chemin>
   stat -c %s <chemin>   # macOS : stat -f %z <chemin>
   ```

2. `create_file_upload(space, name, sha256, content_type, description, expected_version,
   linked_notes, linked_tasks)` — les arguments de `save_file`, sans le contenu. `content_type` :
   le vrai type (`image/png`, `application/pdf`…). Rien n'est écrit à ce stade ; une erreur ici
   (espace en lecture seule, nom invalide, tâche d'un autre espace) se corrige avant d'envoyer.
3. Lance la commande `curl` renvoyée, `<path>` remplacé par le chemin du fichier, **dans les
   15 minutes**. Le lien n'accepte que le contenu de cette empreinte, et sert une fois.
4. Lis la réponse :
   - `201` et `{"id", "version"}` : déposé. Pour vérifier, `get_file(id, with_content=false)`
     donne `sha256` et `size`, à comparer avec l'étape 1 ;
   - `401` : lien expiré ou connexion coupée : redemande un lien ;
   - `400` sur `sha256` : le fichier envoyé n'est pas celui de l'empreinte (mauvais chemin,
     fichier modifié entre-temps) : recalcule et redemande ;
   - `409` : le nom existe déjà, ou la version a bougé : lis le fichier (`get_file`) et redemande un
     lien avec `expected_version` ;
   - `413` : plus de 10 Mo, le fichier ne passe pas.

## 3. Une version suivante

Même nom, et `expected_version` = la version courante (`get_file`, ou l'erreur du premier essai qui
la donne). `description`, `linked_notes` et `linked_tasks` omis gardent ceux de la version
précédente. Pour changer seulement les liens, `link_file` suffit : il ne redépose pas le contenu.

## 4. Rendre compte

Donne à l'utilisateur le nom, la version et la taille déposés. Si le fichier sert une tâche,
lie-la (`linked_tasks`) plutôt que de recopier son contenu dans le journal.
