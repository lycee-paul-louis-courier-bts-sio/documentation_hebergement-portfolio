# Suppression d'un portfolio étudiant - Ansible

![Bannière BTS SIO](https://raw.githubusercontent.com/lycee-paul-louis-courier-bts-sio/documentation_hebergement-portfolio/assets/banniere_bts-sio.png)

## Informations

  - **Mainteneur :** Louis MEDO
  - **Date de création :** 07/05/2026

---

## Contexte

Cette procédure explique comment supprimer le dépôt GitHub d'un étudiant au sein de l'organisation BTS SIO - Paul-Louis Courier et synchroniser l'état de l'infrastructure en supprimant ses ressources associées à l'aide d'Ansible.

---

## Sommaire

1. Suppression du dépôt sur GitHub
2. Modification des variables d'environnement Ansible
3. Exécution du playbook de suppression

---

## 1. Suppression du dépôt sur GitHub

1.  **Supprimer le dépôt de l'étudiant.** Rendez-vous sur GitHub dans l'organisation BTS SIO - Paul-Louis Courier. Accédez au dépôt ciblé, allez dans l'onglet `Settings`, descendez jusqu'à la `Danger Zone`, cliquez sur `Delete this repository` et confirmez l'action.

---

## 2. Modification des variables d'environnement Ansible

**Prérequis**

Avant de procéder, assurez-vous de disposer d'un environnement Ansible fonctionnel et d'avoir cloné en local le [dépôt de l'infrastructure Ansible](https://github.com/lycee-paul-louis-courier-bts-sio/infrastructure_ansible-portfolio). Pour la configuration du poste de travail, veuillez vous référer à la [procédure d'Onboarding Ansible](https://docs-portfolio.bts-sio.eu/1-vue-d%27ensemble/runbook/onboarding_confguration-poste-travail-ansible/).

1.  **Déclarer l'étudiant à supprimer.** Éditez le fichier `inventories/production/group_vars/all.yml` en déplaçant ou ajoutant l'identifiant de l'étudiant dans la section appropriée.

    > ⚠️ **AVERTISSEMENT :** Cette section entraîne des actions destructrices irréversibles sur l'infrastructure. Vérifiez scrupuleusement l'orthographe et l'identité des utilisateurs renseignés dans la liste avant d'enregistrer.

    ```yaml
    # ---
    # Titre       : GROUP VARS - Production
    # Auteur      : Louis MEDO
    # Date        : 14/04/2026
    # Rôle        : Contient les variables de production pour Ansible
    # ---
    ---
    domain_name: "bts-sio.eu"
    admin_email: "contact@bts-sio.eu"

    students:
      - medo-louis
      - desouza-leidiane
      - delalande-nathan

    students_to_remove:
      - <nom-prenom>
    ```

    `students` : Tableau YAML (liste) contenant les utilisateurs dont l'infrastructure doit être maintenue active.

    `students_to_remove` : Tableau YAML listant les identifiants cibles dont l'infrastructure et les accès associés seront purgés lors de la prochaine exécution.

---

## 3. Exécution du playbook de suppression

1.  **Lancer le nettoyage de l'infrastructure.** Exécutez la commande Ansible pour appliquer le retrait des accès et la destruction des ressources de l'étudiant sur les serveurs de production.

    ```bash
    ansible-playbook -i inventories/production/hosts.yml playbooks/delete_students.yml --ask-become-pass
    ```

    `ansible-playbook` : Binaire de l'outil Ansible permettant de lire et d'exécuter un fichier de configuration YAML (playbook).

    `-i inventories/production/hosts.yml` : Paramètre `--inventory` permettant de cibler le fichier listant les adresses IP et connexions des serveurs de production.

    `playbooks/delete_students.yml` : Fichier contenant les instructions et tâches séquentiellement exécutées pour supprimer les ressources de l'étudiant.

    `--ask-become-pass` : Paramètre demandant la saisie du mot de passe d'élévation de privilèges (sudo) requis pour exécuter les tâches d'administration sur les serveurs distants.

---

## Annexe

- [Procédure d'Onboarding Ansible](https://docs-portfolio.bts-sio.eu/1-vue-d%27ensemble/runbook/onboarding_confguration-poste-travail-ansible/)