# Runbook - Initialisation du VPS Portfolio

![Bannière BTS SIO](https://raw.githubusercontent.com/lycee-paul-louis-courier-bts-sio/documentation_hebergement-portfolio/assets/banniere_bts-sio.png)

## Informations

  - **Mainteneur :** Louis MEDO
  - **Date de création :** 15 avril 2026

-----

## Contexte

Cette procédure décrit les étapes nécessaires pour initialiser un VPS vierge (provisionné chez **Infomaniak**) et déployer l'intégralité de la configuration système et applicative. L'objectif est de passer d'une machine "nue" à une plateforme d'hébergement sécurisée et opérationnelle via le playbook `site.yml`.

-----

## Sommaire

1.  Prérequis
2.  Préparation de l'environnement local
3.  Initialisation de la communication (Bootstrap SSH)
4.  Exécution du déploiement complet
5.  Vérifications post-installation

-----

## 1. Prérequis

Vous devez disposer des éléments suivant avant de pouvoir continuer la procédure :

1. **Ansible configuré.** Ansible doit être opérationnelle et vous devez diposer des identifiants nécessaire. (Ref. [Onboarding - Configuration du Poste de Travail (Prérequis Ansible)](onboarding_confguration-poste-travail-ansible.md))
2. **VPS crée.** Commandez un VPS Cloud (Debian 13). Lors de la création, **injectez votre clé SSH publique** via l'interface Infomaniak pour permettre la première connexion.
3. **Récupération de l'IPv4.** Notez l'adresse IP publique de l'instance (ex: `83.228.241.93`).
4. **Récupèrer le code du dépôt sur la machine.** Réaliser un `git clone https://github.com/lycee-paul-louis-courier-bts-sio/infrastructure_ansible-portfolio.git` pour avoir le code du dépôt de l'infrastructure as code.

-----

## 2. Préparation de l'environnement local

Votre poste de travail doit être prêt à orchestrer le déploiement.

1.  **Installation des dépendances Ansible.** Le projet utilise des modules communautaires spécifiques.

    ```bash
    ansible-galaxy install -r requirements.yml
    ```

    `install -r` : Télécharge et installe automatiquement les collections listées dans le fichier `requirements.yml` (ex: `community.general`).

2.  **Mise à jour de l'inventaire.** Vérifiez que l'adresse IP dans `inventories/production/hosts.yml` correspond bien à votre nouveau VPS.

3.  **Modification de du fichier d'inventaire.** Pour ajouter un nouveau serveur à votre inventaire Ansible, ouvrez le fichier `hosts.yml` situé dans le répertoire `inventories/production`. Dans la section `hosts` (sous `webservers`), ajoutez votre nouveau VPS en respectant strictement l'indentation existante (2 espaces par niveau). Assurez-vous de remplacer les valeurs entre crochets par vos informations réelles.

    **Exemple :**

    ```yaml
    # ---
    # Titre       : Hosts - Production
    # Auteur      : Louis MEDO
    # Date        : 14/04/2026
    # Rôle        : Contient la liste des serveurs de production
    # ---
    all:
    children:
        webservers:
        hosts:
            vps_portfolio:
            ansible_host: 83.228.241.93
            ansible_user: admin  # Remplacer par le nom d'utilisateur réel
            nouveau_vps:
            ansible_host: 123.45.67.89  # Remplacer par l'IP de votre VPS
            ansible_user: admin         # Remplacer par le nom d'utilisateur réel
    ```

    **Points de vigilance :**
    
    - L'indentation doit être cohérente (utilisez des espaces, pas des tabulations).
    - Le nom du nouvel hôte (ex: `nouveau_vps`) doit être unique dans la section.
    - Les champs `ansible_host` et `ansible_user` sont obligatoires pour chaque hôte.

-----

## 3. Initialisation de la communication

Ansible refuse la connexion si la clé du serveur n'est pas connue, par mesure de sécurité.

1.  **Ajout à la liste de confiance.** Forcez l'enregistrement de l'empreinte du serveur sur votre poste.

    ```bash
    ssh-keyscan 83.228.241.93 >> ~/.ssh/known_hosts
    ```

    `ssh-keyscan` : Récupère la clé publique d'identification du serveur distant pour éviter les attaques de type "Man-in-the-middle".

-----

## 4. Exécution du déploiement complet

Cette étape applique l'ensemble des rôles : sécurisation, serveur Web, certificats SSL et portfolios étudiants.

1.  **Lancement du Playbook maître.**

    ```bash
    ansible-playbook -i inventories/production/hosts.yml playbooks/site.yml --ask-vault-pass --ask-become-pass
    ```

    `-i inventories/production/hosts.yml` : Cible l'environnement de production.

    `playbooks/site.yml` : Exécute dans l'ordre les rôles `common`, `webserver`, `ci_cd` et `student_deploy`.

    `--ask-vault-pass` : Nécessaire pour déchiffrer les secrets (Token Infomaniak pour le certificat Wildcard).

-----

## 5. Vérifications post-installation

1.  **Vérification visuelle du MOTD.** Connectez-vous en SSH. Vous devriez voir le tableau de bord système personnalisé.

2.  **Test du certificat SSL.** Vérifiez qu'un portfolio étudiant (ex: `https://medo-louis.bts-sio.eu`) répond correctement en HTTPS sans erreur de certificat.

3.  **État des services.**

    ```bash
    systemctl status apache2 php8.4-fpm
    ```

-----

## Schéma du processus de Bootstrap

```text
[ ADMIN PC ] 
      |
      | 1. SSH-Keyscan (Trust)
      | 2. Ansible-Playbook (site.yml)
      V
[ VPS INFOMANIAK ]
      |
      +--- [ Rôle Common ] : Update, Sécurité SSH, MOTD
      +--- [ Rôle Webserver ] : Apache, SSL Wildcard (Infomaniak API)
      +--- [ Rôle CI/CD ] : Compte de déploiement GitHub
      +--- [ Rôle Student ] : Vhosts & Clonage Git des portfolios
```

-----

## Annexe

  - [Dépôt Github - Infrastructure Ansible](https://github.com/lycee-paul-louis-courier-bts-sio/infrastructure_ansible-portfolio)
  - [Architecture - Fonctionnement infrastructure Ansible](../architecture/architecture-iac.md)