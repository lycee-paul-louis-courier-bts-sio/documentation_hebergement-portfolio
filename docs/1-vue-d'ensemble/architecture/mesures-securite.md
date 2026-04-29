# Mesures de sécurité mis en place

![Bannière BTS SIO](https://raw.githubusercontent.com/lycee-paul-louis-courier-bts-sio/documentation_hebergement-portfolio/assets/banniere_bts-sio.png)

## Informations

  - **Mainteneur :** Louis MEDO
  - **Date de création :** 29 avril 2026

-----

## Contexte

Ce document recense les configurations techniques appliquées pour sécuriser l'hébergement des portfolios. Il vise à garantir la confidentialité des données, l'isolation des environnements utilisateurs et la protection contre les cyberattaques courantes (interception, injection, détournement).

-----

## Sommaire

1. Isolation des processus (PHP-FPM Pool)
2. Obscurcissement de la version du serveur
3. Désactivation de la méthode TRACE
4. Forçage du protocole sécurisé (HSTS)
5. Protection contre le MIME Sniffing
6. Défense contre le Clickjacking
7. Sécurisation de la couche Transport (SSL/TLS)
8. Restriction de l'exploration de fichiers
9. Contrôle d'accès et performances (AllowOverride None)
10. Redirection systématique HTTP vers HTTPS

-----

## 1. Isolation des processus (PHP-FPM Pool)

L'utilisation d'un utilisateur et d'un groupe dédiés par instance (`portfolio-{{ item }}`) limite l'impact d'une compromission : si un site est piraté, l'attaquant ne peut pas accéder aux fichiers des autres utilisateurs.

* Un **Pool PHP-FPM** est un groupe de processus isolés. En assignant un UID/GID unique, on cloisonne les droits d'exécution au niveau du système d'exploitation.

-----

## 2. Obscurcissement de la version du serveur

La directive `ServerTokens Prod` masque les versions exactes d'Apache et de l'OS. `ServerSignature Off` retire les informations système des pages d'erreur. Cela complexifie la reconnaissance pour un attaquant.

* `ServerTokens` : Définit le niveau de détail de l'en-tête HTTP "Server".
* `ServerSignature` : Configure le pied de page des documents générés par le serveur (erreurs 404, etc.).

-----

## 3. Désactivation de la méthode TRACE

`TraceEnable Off` prévient les attaques de type Cross-Site Tracing (XST), évitant le vol de cookies de session via des scripts malveillants.

* La méthode **TRACE** renvoie au client le message exact reçu par le serveur à des fins de débogage, ce qui peut exposer des en-têtes sensibles.

-----

## 4. Forçage du protocole sécurisé (HSTS)

L'en-tête `Strict-Transport-Security` avec `preload` garantit que les navigateurs communiquent exclusivement en HTTPS pendant un an, éliminant les risques d'interception (Man-in-the-Middle).

* Le **HSTS** est une instruction de sécurité mémorisée par le navigateur pour interdire toute connexion non chiffrée vers le domaine.

-----

## 5. Protection contre le MIME Sniffing

La valeur `nosniff` force le navigateur à respecter scrupuleusement le type de contenu envoyé (ex: ne pas exécuter un `.txt` comme du `.js`), empêchant l'exécution de scripts malveillants masqués.

* `X-Content-Type-Options`. Il empêche le navigateur d'interpréter le contenu différemment de ce qui est déclaré par le serveur.

-----

## 6. Défense contre le Clickjacking

La directive `SAMEORIGIN` empêche l'intégration du site dans une `<iframe>` sur un domaine tiers, protégeant les utilisateurs contre les détournements de clics.

* `X-Frame-Options`. Il contrôle si un navigateur peut afficher une page dans une balise `<frame>` ou `<iframe>`.

-----

## 7. Sécurisation de la couche Transport (SSL/TLS)

L'exclusion des protocoles obsolètes (`-SSLv3`, `-TLSv1`) et l'usage de suites de chiffrement fortes (TLS 1.3/1.2 GCM) assurent une confidentialité maximale.

* **TLS** est le successeur sécurisé de SSL. Les versions anciennes possèdent des failles connues (ex: POODLE) et doivent être désactivées.

-----

## 8. Restriction de l'exploration de fichiers

La désactivation de l'indexation (`Options -Indexes`) empêche un visiteur de voir la liste des fichiers d'un répertoire en l'absence de fichier `index.php` ou `index.html`.

* `Options` contrôle les fonctionnalités disponibles dans un répertoire. Le signe `-` désactive explicitement la génération d'index automatique.

-----

## 9. Contrôle d'accès et performances (AllowOverride None)

En interdisant les fichiers `.htaccess`, on empêche les modifications locales de configuration potentiellement dangereuses et on améliore les performances.

* `AllowOverride` détermine quelles directives placées dans les fichiers `.htaccess` peuvent surcharger la configuration principale du serveur.

-----

## 10. Redirection systématique HTTP vers HTTPS

La redirection 301 (permanente) via `RewriteEngine` assure qu'aucun trafic ne circule en clair sur le port 80, protégeant l'utilisateur dès la première connexion.

* `RewriteEngine On` active le module de réécriture d'URL. La règle **301** indique aux moteurs de recherche et navigateurs que la version sécurisée est l'adresse définitive.