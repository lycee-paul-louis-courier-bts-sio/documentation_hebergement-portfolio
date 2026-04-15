# Runbook : Mise à jour de la liste des étudiants

![Bannière BTS SIO](https://raw.githubusercontent.com/lycee-paul-louis-courier-bts-sio/documentation_hebergement-portfolio/assets/banniere_bts-sio.png)

## Informations

  - **Mainteneur :** Louis MEDO
  - **Date de création :** 16 avril 2026

---

## Contexte

Cette procédure décrit le workflow opérationnel pour ajouter ou retirer un étudiant de la plateforme. Elle assure que la modification est correctement testée, déployée sur le serveur de production et historisée dans le dépôt Git de l'infrastructure.

---

## Sommaire

1.  Modification des variables d'inventaire
2.  Validation en environnement de test (Staging)
3.  Déploiement en production
4.  Validation et Versioning

---

## 1. Modification des variables d'inventaire

1.  **Mise à jour de la liste des étudiants.** Éditez le fichier de variables de production pour ajouter l'identifiant de l'étudiant (format `nom-prenom`).

    ```bash
    vim inventories/production/group_vars/all.yml
    ```

    `inventories/production/group_vars/all.yml` : Fichier contenant la liste `students` utilisée par les rôles Ansible pour créer les comptes et les VirtualHosts.

---

## 2. Validation en environnement de test (Staging)

1.  **Déploiement sur le serveur de test.** Avant toute action en production, appliquez la modification sur l'environnement de staging (Vagrant ou VPS de test) pour vérifier l'absence d'erreurs de syntaxe.

    ```bash
    ansible-playbook -i inventories/staging/hosts.yml playbooks/deploy_students.yml --ask-vault-pass
    ```

    `-i inventories/staging/hosts.yml` : Cible l'inventaire de test pour ne pas impacter la production immédiatement.

---

## 3. Déploiement en production

1.  **Exécution du playbook de déploiement.** Une fois le test réussi, poussez la configuration vers le serveur de production. Nous utilisons un playbook spécifique plus léger que le déploiement complet.

    ```bash
    ansible-playbook -i inventories/production/hosts.yml playbooks/deploy_students.yml --ask-vault-pass
    ```

    `playbooks/deploy_students.yml` : Scénario optimisé qui exécute uniquement le rôle `student_deploy` pour accélérer l'opération.

    `--ask-vault-pass` : Requis car le playbook accède aux secrets chiffrés pour configurer les accès Git.

---

## 4. Validation et Versioning

1.  **Vérification du service.** Testez l'accès URL de l'étudiant fraîchement ajouté (ex: `https://nouvel-etudiant.bts-sio.eu`).

2.  **Sauvegarde des modifications (Git).** Une fois le déploiement confirmé, archivez la modification dans le dépôt central.

    ```bash
    git add inventories/production/group_vars/all.yml
    git commit -m "feat: ajout de l'étudiant nouvel-etudiant"
    git push origin main
    ```

---

## Annexe

  - [Dépôt Github - Infrastructure Ansible](https://github.com/lycee-paul-louis-courier-bts-sio/infrastructure_ansible-portfolio)
  - [Architecture - Fonctionnement infrastructure Ansible](https://lycee-paul-louis-courier-bts-sio.github.io/documentation_hebergement-portfolio/1-vue-d'ensemble/architecture-iac/)