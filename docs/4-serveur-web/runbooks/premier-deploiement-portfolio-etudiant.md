# Premier déploiement du portfolio sur GitHub

![Bannière BTS SIO](https://raw.githubusercontent.com/lycee-paul-louis-courier-bts-sio/documentation_hebergement-portfolio/assets/banniere_bts-sio.png)

## Informations

  - **Mainteneur :** Louis MEDO
  - **Date de création :** 08/04/2026

-----

## Contexte

Cette procédure guide les étudiants pour envoyer (pousser) le code source de leur portfolio vers le dépôt GitHub de l'établissement pour la toute première fois. Cette action est essentielle car elle déclenchera automatiquement le pipeline CI/CD qui publiera le site sur le serveur web de production.

-----

## Sommaire

1.  Vérification du point d'entrée du site web
2.  Envoyer votre portfolio sur le dépôt Git distant
3.  Vérification

-----

## 1. Vérification du point d'entrée du site web

1.  **Vérifier la présence du fichier index.** Le serveur web Apache requiert un fichier principal à la racine du projet pour router la requête. Sans cela, le serveur retournera une erreur de sécurité (403 Forbidden). Naviguez dans le dossier de votre projet et listez son contenu.

-----

## 2. Envoyer votre portfolio sur le dépôt Git distant

### 2.1 **Rappel sur le fonctionnement de Git**

![Schéma de fonctionnement de Git - OpenClassrooms](https://user.oc-static.com/upload/2021/10/05/16334576106761_image27.png)

*Schéma de fonctionnement de Git - [OpenClassrooms](https://openclassrooms.com/en/courses/7162856-gerez-du-code-avec-git-et-github/7165726-travaillez-depuis-votre-depot-local-git?archived-source=5641721)*

Il est fondamental de distinguer **Git** de **GitHub**. 

* **Git** est le logiciel installé sur votre machine. C'est le moteur qui traque les modifications de vos fichiers locaux.
* **GitHub** est un service d'hébergement en ligne (un serveur distant). 

**Git** est l'équivalent de la fonctionnalité "Historique des versions" d'un logiciel de traitement de texte sur votre ordinateur, tandis que GitHub est comme **Google Drive**, l'endroit sécurisé sur le cloud où vous envoyez votre document final pour le stocker ou le partager.

Le cycle de vie d'un fichier avec Git suit trois grandes étapes avant d'arriver sur le serveur :

1. **Working Directory (Dossier de travail) :** Vos fichiers actuels sur votre ordinateur.
2. **Staging Area (Zone d'attente) :** L'espace où vous préparez les fichiers qui feront partie de la prochaine sauvegarde (`git add`).
3. **Local Repository (Dépôt local) :** La sauvegarde validée dans l'historique Git de votre ordinateur (`git commit`).
4. **Remote Repository (Dépôt distant) :** L'envoi de l'historique local vers les serveurs de GitHub (`git push`).

### 2.2 **Procédure de déploiement**

Afin d'éviter les conflits d'historique et de récupérer automatiquement les scripts de déploiement (GitHub Actions) déjà configurés par l'établissement, vous devez d'abord récupérer (cloner) le dépôt distant sur votre machine.

1.  **Cloner le dépôt distant.** Récupérez la structure officielle configurée pour vous.

    ```bash
    git clone https://github.com/lycee-paul-louis-courier-bts-sio/portfolio-<nom-prenom>.git
    cd portfolio-<nom-prenom>
    ```

     **💡 Noubliez pas de changer `<nom-prenom>` par votre nom et prenom.**

    `git clone` : Télécharge le dépôt distant et son historique Git complet pour créer une copie conforme sur votre machine.

    `[URL]` : L'adresse de votre dépôt GitHub (disponible via le bouton "Code").

    `cd portfolio-<nom-prenom>` : Modifie le répertoire de travail pour entrer dans le dossier fraîchement cloné.

2.  **Intégrer vos fichiers.** Copiez l'intégralité des fichiers de votre portfolio personnel à l'intérieur de ce nouveau dossier `portfolio-nom-prenom`. 

     - *Assurez-vous que le fichier `index.php` ou `index.html` se trouve bien à la racine de ce dossier, et non dans un sous-dossier.*

3.  **Ajouter et valider les fichiers.** Placez vos fichiers dans la zone d'attente, puis validez-les dans l'historique local.

    ```bash
    git add .
    git commit -m "feat: premier déploiement du portfolio"
    ```

    `git add` : Indique à Git de préparer des fichiers pour la validation.

    `.` : Sélecteur global qui cible tous les fichiers et dossiers modifiés du répertoire courant.

    `git commit` : Enregistre définitivement les modifications préparées dans la base de données locale de Git.

    `-m "..."` : (*Message*) Argument obligatoire permettant d'attacher un texte descriptif à cette version pour comprendre ce qui a été modifié.

4.  **Pousser le code vers GitHub.** Envoyez votre historique local vers le serveur distant.

    ```bash
    git push origin main
    ```

    `git push` : Expédie (pousse) les commits locaux vers le dépôt distant.

    `origin` : Le nom par défaut attribué à l'adresse de votre dépôt GitHub lors du clone.

    `main` : Le nom de la branche principale sur laquelle vous envoyez votre travail.

---

## 3. Vérification

Une fois la commande `git push` exécutée avec succès, il est impératif de contrôler que le déploiement CI/CD s'est bien déroulé jusqu'à la mise en production.

1. **Vérifier les fichiers sur GitHub.** Rendez-vous sur la page web de votre dépôt GitHub. Assurez-vous que tous vos fichiers (y compris le `index.php`/`index.html`) sont bien visibles à la racine.

2. **Contrôler le pipeline de déploiement (GitHub Actions).** Cliquez sur l'onglet **Actions** en haut de votre dépôt GitHub.

     - Vous devriez voir un flux d'exécution en cours (nommé selon le message de votre commit).
     - Attendez quelques secondes. Si l'icône devient une **pastille verte (✅)**, le serveur VPS a bien récupéré vos fichiers. Si c'est une croix rouge, ouvrez le log pour lire l'erreur.

3. **Tester l'accès au site de production.**

     - Si le pipeline est au vert : 
         1. ouvrez un nouvel onglet dans votre navigateur web.
         2. Saisissez votre URL sous la forme : `https://nom-prenom.bts-sio.eu`
         3. Votre portfolio doit s'afficher correctement ✅.

**⚠️ En cas de problème vous pouvez contactez Louis MEDO à l'adresse email suivante [louis.medo@loutik.fr](mailto:louis.medo@loutik.fr) avec une description de votre problème.**

-----

## Annexe

  - [Documentation officielle Git - Les bases](https://git-scm.com/book/fr/v2)