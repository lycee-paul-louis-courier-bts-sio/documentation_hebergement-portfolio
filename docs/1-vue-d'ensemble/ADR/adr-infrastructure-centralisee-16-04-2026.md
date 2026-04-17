# ADR 002 : Évolution vers une gestion centralisée de l'infrastructure

![Bannière BTS SIO](https://raw.githubusercontent.com/lycee-paul-louis-courier-bts-sio/documentation_hebergement-portfolio/assets/banniere_bts-sio.png)

## Informations

  - **Mainteneur :** Louis MEDO
  - **Date de création :** 16/04/2026
  - **Numéro ADR :** 002

---

## 1. Résumé exécutif

Ce document présente la stratégie d'évolution de l'infrastructure d'hébergement des portfolios étudiants. L'objectif est de transiter d'une administration décentralisée sur terminaux personnels vers un modèle de gestion centralisé et segmenté. Cette mutation vise à garantir la sécurité des secrets, la traçabilité des opérations et l’isolation des environnements.

---

## 2. Analyse de l'existant et limites

Actuellement, l'exécution des playbooks Ansible est réalisée depuis les postes de travail.

- **Sécurité des secrets** : Bien que chiffrées via `ansible-vault`, les variables sensibles résident sur des disques locaux non maîtrisés.
- **Absence de traçabilité** : Aucun journal d'audit centralisé ne permet de corréler une modification d'infrastructure à un utilisateur précis.
- **Dette technique** : La gestion individuelle des versions d'Ansible et des dépendances Python crée des risques d'incohérence lors des déploiements.

---

## 3. Détails des configurations proposées

### 3.1. Estimation des besoins capacitaires (Dimensionnement)
Afin de choisir la configuration la plus adaptée, il convient d'estimer la consommation moyenne d'un portfolio étudiant :

* **RAM (PHP-FPM) :** Chaque processus actif consomme entre 30 et 50 Mo. L'OS et Apache nécessitent environ 500 Mo.
* **Stockage :** Un portfolio standard (code, médias légers) occupe environ 500 Mo.
* **Besoins selon la promotion :**
  * *Petite promotion (3 à 5 étudiants) :* Un VPS de 2 Go de RAM est suffisant (charge très faible).
  * *Promotion complète (10 à 15 étudiants) :* Un VPS de 4 Go à 8 Go de RAM est recommandé pour absorber les processus PHP isolés sans saturer la mémoire.

---

### 3.2. Scénarios d'architecture

**Configuration A : Modèle centralisé (1 Serveur WEB portfolio + 1 Serveur administration)**

* **Architecture** :
    * **1 Serveur Web** (8 Go RAM) : Héberge l'ensemble des portfolios.
    * **1 Serveur Administration** (KVM/CLI) (8 Go RAM) : Agit comme bastion de déploiement Ansible et nœud de supervision.
* **Avantages** : Isolation totale des clés SSH, traçabilité des logs, segmentation par VM des outils critiques.

**Configuration B : Modèle haute disponibilité (2 Serveurs WEB portfolio + 1 Serveur administration)**

* **Architecture** : 
    * **2 Serveurs Web** (2 Go RAM chacun) : Répartition des portfolios (ex: une promotion par VPS).
    * **1 Serveur Administration** (KVM/CLI) (8 Go RAM) : Pilotage centralisé des deux nœuds.
* **Avantages** : Pas de **SPOF** (point de défaillance unique) sur la partie Web. En cas de panne du VPS Web 1, Ansible permet un redéploiement d'urgence des portfolios sur le VPS Web 2.

**Configuration C : Modèle minimaliste (1 Serveur WEB portfolio sans serveur administration)**

* **Architecture** : 1 Serveur Web (8 Go RAM). L'administration reste faite depuis les PC locaux.
* **Avantages** : Facturation simplifiée au maximum.
* **Limites** : Retour aux risques de l'analyse de l'existant (secrets dispersés, pas de logs d'audit, dépendance au poste de travail).

---

## 4. Analyse comparative et budget prévisionnel

Retrouvez ci-dessous une matrice contenant les différentes solutions du marché qui répondent à l'expression des besoins de chaque configuration.

| Critère | Configuration A <br>(1 web-portfolio + 1 admin) | Config B <br>(2 web-portfolio + 1 admin) | Config C <br>(1 web-portfolio) |
| :--- | :--- | :--- | :--- |
| **Hébergement WEB** | OVH (VPS 8Go)[^1] | Infomaniak (VPS 2x2Go)[^2] | OVH (VPS 8Go) |
| **Hébergement Admin** | OVH (VPS 8Go) | OVH (VPS 8Go) | - |
| **Coût WEB** | 6,62 € | 5,40 € | 6,62 € |
| **Coût Admin** | 6,62 € | 6,62 € | - |
| **Coût mensuel total** | **13,24 €** | **12,02 €** | **6,62 €** |
| **Sécurité** | Bastion | Bastion + Redondance | Local |
| **Gestion** | Centralisée | Centralisée | Décentralisée |

- [^1] : Le VPS 8 Go de chez OVH a été privilégié, car Infomaniak propose une tarification plus élevée pour des performances inférieures sur cette gamme de RAM.
- [^2] : **Point de vigilance :** L'hébergement de plus de 10 à 15 portfolios sur un VPS de 2 Go (Config B) peut entraîner une saturation de la mémoire (OOM). Si le budget le permet, il est recommandé de surclasser la RAM ou d'optimiser strictement les pools PHP.

*Sources :*

- [VPS OVH](https://www.ovhcloud.com/fr/vps/)
- [VPS Infomaniak](https://www.infomaniak.com/en/hosting/vps-lite)

---

## 5. Détails techniques de réalisation (Serveur Admin)

Pour les configurations incluant une administration centralisée :

1.  **Hyperviseur** : Installation de `libvirt` et `qemu-kvm` sur base Debian sans interface graphique.
2.  **Segmentation** : Création de deux VM via `virt-install` (VM Ansible pour le déploiement et VM Supervision pour le monitoring).
3.  **Réseau** : Configuration en mode NAT. Le serveur d'administration utilise l'IP publique de l'hôte pour sortir, mais les flux entrants sont strictement limités au SSH filtré par IP.

---

## 6. Analyse des risques et points de vigilance

* **Saturation mémoire (VPS 2 Go)** : La Configuration B présente un risque de goulot d'étranglement matériel. En cas de pic de trafic ou d'accumulation de processus, la RAM de 2 Go s'avérera insuffisante pour une charge dépassant une dizaine de portfolios.
* **Support VT-x** : Obligatoire pour les options A et B afin de permettre la virtualisation imbriquée (*Nested Virtualization*) sur le serveur d'administration.
* **Optimisation PHP-FPM** : Pour les environnements contraints en mémoire, il est impératif d'utiliser `pm = ondemand` (qui libère les processus inactifs) pour éviter l'épuisement de la RAM.
* **Sécurisation SSH** : Authentification par clé privée obligatoire sur tous les nœuds de l'infrastructure.

---

## 7. Conclusion et préconisation

La **Configuration A** est préconisée. Pour un coût d'infrastructure total de **13,24 € / mois**, elle permet de mettre en place un véritable serveur d'administration (bastion professionnel) tout en offrant des ressources amplement suffisantes (8 Go) côté serveur Web pour héberger l'ensemble des portfolios étudiants de manière fluide, évolutive et ultra-sécurisée.