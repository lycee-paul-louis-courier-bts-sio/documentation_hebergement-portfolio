# Onboarding - Configuration du Poste de Travail (Prérequis Ansible)

![Bannière BTS SIO](https://raw.githubusercontent.com/lycee-paul-louis-courier-bts-sio/documentation_hebergement-portfolio/assets/banniere_bts-sio.png)

## Informations

  - **Mainteneur :** Louis MEDO
  - **Date de création :** 15 avril 2026

-----

## Contexte

Cette procédure accompagne les nouveaux administrateurs dans la préparation de leur environnement de travail. Elle détaille l'installation d'Ansible selon le système d'exploitation, l'accès aux données chiffrées du projet et la sécurisation des connexions avec le serveur de production.

-----

## Sommaire

1.  Installation d'Ansible (Linux, MacOS, Windows)
2.  Gestion du mot de passe de coffre-fort (Ansible Vault)
3.  Sécurisation de la connexion SSH (known_hosts)

-----

## 1. Installation d'Ansible

Le "nœud de contrôle" (votre PC) doit posséder Ansible pour envoyer les configurations au serveur.

1.  **Installation sur Linux (Debian/Ubuntu).** Utilisez le gestionnaire de paquets standard.

    ```bash
    sudo apt update && sudo apt install ansible -y
    ```

    `apt` : Gestionnaire de paquets utilisé pour installer des logiciels sur Debian.

2.  **Installation sur MacOS.** Utilisez le gestionnaire Homebrew.

    ```bash
    brew install ansible
    ```

    `brew` : Gestionnaire de paquets populaire pour installer des outils techniques sur Mac.

3.  **Installation sur Windows (via WSL2).** Ansible ne tourne pas nativement sur Windows pour l'administration. Il faut installer **WSL** (Windows Subsystem for Linux), puis suivre la procédure **Linux** ci-dessus à l'intérieur du terminal Ubuntu.

    ```powershell
    wsl --install
    ```

    `wsl` : Commande permettant d'installer un véritable noyau Linux au sein de Windows.

-----

## 2. Accès aux secrets (Ansible Vault)

Les fichiers sensibles comme `secrets/vault.yml` sont chiffrés pour ne pas apparaître en clair.

1.  **Récupération du mot de passe.** Le mot de passe du "Vault" vous sera communiqué par l'administrateur via un canal sécurisé ou à récupérer dans le gestionnaire de mots de passe.

2.  **Utilisation lors du déploiement.** Pour que vos commandes fonctionnent, vous devez prouver que vous connaissez ce mot de passe.

    ```bash
    ansible-playbook ... --ask-vault-pass
    ```

    `--ask-vault-pass` : Indique à Ansible de vous demander le mot de passe du coffre-fort avant de commencer.

-----

## 3. Sécurisation de la connexion (known_hosts)

La configuration `host_key_checking = True` est activée dans `ansible.cfg`. Cela signifie que vous devez enregistrer l'identité du serveur avant toute action pour éviter les usurpations.

1.  **Récupération de l'empreinte du serveur.** Scannez la clé publique du serveur de production (IP : `83.228.241.93`) pour l'ajouter à vos hôtes connus.

    ```bash
    ssh-keyscan 83.228.241.93 >> ~/.ssh/known_hosts
    ```

    `ssh-keyscan` : Récupère la signature numérique d'un serveur distant.

    `>> ~/.ssh/known_hosts` : Ajoute cette signature à votre fichier de confiance local.

-----

## Annexe

  - [Dépôt Github - Infrastructure Ansible](https://github.com/lycee-paul-louis-courier-bts-sio/infrastructure_ansible-portfolio)
  - [Architecture - Fonctionnement infrastructure Ansible](../architecture/architecture-ci-cd.md)