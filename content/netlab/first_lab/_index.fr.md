---
title: "Mon Premier Lab"
date: 2025-02-14T12:00:00+02:00
weight: 1
cascade:
  type: docs
---

## Introduction 📚

Dans cet article, nous allons explorer comment installer notre tout premier netlab Containerlab en utilisant **DevPod**.  
Nous nous concentrerons sur l'utilisation d'un fournisseur cloud, en l'occurrence **AWS**, pour héberger notre projet.  
Pourquoi le **Cloud** ? Parce que les labs réseau peuvent consommer énormément de ressources, et nous avons besoin de pouvoir les déployer, les arrêter et les détruire rapidement, tant pour la performance que pour l'efficacité financière. 💡💰

Nous y parviendrons en combinant :

- **DevPod**
- **DevContainer**
- **Containerlab**

De plus, nous utiliserons une topologie simple que vous pouvez retrouver sur mon [dépôt GitHub](https://github.com/darnodo/VXLAN-EVPN). Notre objectif principal est de déployer ce lab sur AWS avec DevPod.  
Allons-y, c'est parti ! 🚀😊

## Prérequis 🔧

Avant de commencer, quelques étapes importantes sont à réaliser :

1. **Autorisation de l'environnement AWS** :  
   Assurez-vous que DevPod est autorisé à accéder à votre environnement AWS. Pour un guide détaillé sur la configuration de DevPod avec AWS, consultez mon article sur ce [sujet](/documentation/devpod). 🔑

2. **Topologie Containerlab** :  
   Nous avons besoin d'un fichier de topologie compréhensible par Containerlab. Dans notre cas, nous créons une topologie VXLAN simple. 🗺️

## Topologie Containerlab 🔄

Notre lab simulera une topologie VXLAN comprenant :

- **1 switch Spine**
- **2 switches Leaf**
- **2 nœuds Host**

Le diagramme suivant illustre la topologie VXLAN :

![Topologie VXLAN](VXLAN.fr.svg#center)

Voici le fichier de topologie Containerlab (`lab_vxlan.yml`) utilisé pour cette configuration :

```yaml
name: vxlan-evpn-irb
topology: 
  nodes:
    spine1:
      kind: ceos
      image: ceos:4.32.0.1F
      mgmt-ipv4: 172.20.20.101
    leaf1:
      kind: ceos
      image: ceos:4.32.0.1F
      mgmt-ipv4: 172.20.20.11
    leaf2:
      kind: ceos
      image: ceos:4.32.0.1F
      mgmt-ipv4: 172.20.20.12
    host1:
      kind: linux
      image: alpine:latest
      binds:
        - hosts/h1_interfaces:/etc/network/interfaces
      mgmt-ipv4: 172.20.20.21
    host2:
      kind: linux
      image: alpine:latest
      binds:
        - hosts/h2_interfaces:/etc/network/interfaces
      mgmt-ipv4: 172.20.20.22
  links:
    - endpoints: ["spine1:eth1", "leaf1:eth1"]
    - endpoints: ["spine1:eth2", "leaf2:eth1"]
    - endpoints: ["leaf1:eth2", "host1:eth1"]
    - endpoints: ["leaf2:eth2", "host2:eth1"]
```

### Décryptage de la Topologie 🧐

1. **Nom et Structure** :
   - `name: vxlan-evpn-irb` – C'est le nom du lab.
   - La topologie se divise en **nodes** (périphériques) et **links** (connexions entre les périphériques).

2. **Nodes** :
   - **Couche Spine** :
     - `spine1` : Un switch Arista cEOS containerisé utilisant l'image version `4.32.0.1F`.
     - **IP de gestion** : `172.20.20.101`
   - **Couche Leaf** :
     - `leaf1` et `leaf2` : Des switches Arista cEOS utilisant la même version d'image.
     - **IPs de gestion** : `172.20.20.11` et `172.20.20.12`
   - **Couche Host** :
     - `host1` et `host2` : Des conteneurs Linux exécutant Alpine Linux.
     - Ils incluent des configurations personnalisées pour les interfaces réseau montées depuis l'hôte.
     - **IPs de gestion** : `172.20.20.21` et `172.20.20.22`

3. **Links** :
   - **De Spine à Leaf** :
     - `spine1:eth1` ↔ `leaf1:eth1`
     - `spine1:eth2` ↔ `leaf2:eth1`
   - **De Leaf à Host** :
     - `leaf1:eth2` ↔ `host1:eth1`
     - `leaf2:eth2` ↔ `host2:eth1`

Cette topologie représente une architecture spine-leaf typique, courante dans les datacenters pour permettre une connectivité en couche 2 et en couche 3 avec des configurations VXLAN EVPN. 🔗💻

## Déployer le Lab 🛠️

Nous allons déployer le lab avec **DevPod** de deux manières :

### 1. En Utilisant le Dépôt 📥

1. **Valider la configuration du fournisseur AWS** :  
   Assurez-vous que votre fournisseur AWS est correctement configuré. Plus de détails [ici](/documentation/devpod). ✅

2. **Créer un Workspace** :  
   - Rendez-vous dans l'onglet **Workspace** et cliquez sur **Create Workspace**.
   - Indiquez la **source du Workspace** : utilisez le [dépôt GitHub](https://github.com/darnodo/VXLAN-EVPN).
   - Sélectionnez **AWS** comme fournisseur.
   - Choisissez votre IDE par défaut.
   - Enfin, cliquez sur **Create Workspace**.

   ![Configuration DevPod](devpod_configuration.fr.png#center)

### 2. En Utilisant un Dossier Local 🗂️

Si vous préférez utiliser votre dépôt local :

- La seule différence se trouve dans la **source du Workspace**.
- Il vous suffit de le pointer vers votre dépôt local.

   ![Configuration DevPod - Local](devpod_configuration_local.fr.png#center)

## Démarrer le Lab 🎬

> [!WARNING] Images cEOS  
> Le lab utilise l'**image cEOS v4.32.0.1F**.  
> Pour télécharger cette image, rendez-vous sur la [page de téléchargement Arista](https://www.arista.com/en/support/software-download). ⚠️

1. **Importer l'image cEOS** :  
   Enregistrez l'image cEOS dans votre dossier `network_images` en la glissant-déposant dans VSCode.  
   Importez l'image en utilisant la commande suivante :

   ```bash
   docker import network_images/cEOS64-lab-4.32.0.1F.tar.xz ceos:4.32.0.1F
   ```

2. **Déployer le Lab** :  
   Déployez le lab en utilisant Containerlab :

   ```bash
   sudo containerlab deploy -t lab_vxlan.yml
   ```

   Suivez les instructions du CLI pour configurer vos périphériques. Pour des étapes de configuration détaillées, consultez [ce guide](https://github.com/darnodo/VXLAN-EVPN/tree/main/documentation/eos_configuration). 🔧🖥️

3. **Visualiser l'Architecture** :  
   Vérifiez la topologie déployée grâce à la vue graphique de Containerlab :

   ```bash
   containerlab graph -t lab_vxlan.yml
   ```

   Les ports (par exemple, le port 50080 mentionné dans le `devcontainer.json`) sont redirigés. Accédez à la vue graphique via [localhost](http://localhost:50080).

   ![Vue Graphique](Graph_view.fr.png#center)

## Utiliser EdgeShark 🦈

EdgeShark est un outil web qui permet de capturer des paquets depuis votre environnement de lab. Il redirige les captures du lab vers Wireshark exécuté localement. 📡🔍

Pour plus d'informations, consultez le [guide de démarrage d'EdgeShark](https://edgeshark.siemens.io/#/getting-started?id=optional-capture-plugin).

### Configuration d'EdgeShark dans le DevContainer 🐳

Dans la configuration du **DevContainer**, la commande `postCreateCommand` suivante a été ajoutée :

```bash
sudo mkdir -p /opt/edgeshark && sudo curl -sL https://github.com/siemens/edgeshark/raw/main/deployments/wget/docker-compose.yaml -o /opt/edgeshark/docker-compose.yaml
```

Cette commande télécharge un fichier Docker Compose pour faciliter l'utilisation d'EdgeShark. 🚀

### Lancer EdgeShark ⚡

Pour démarrer EdgeShark, exécutez :

```bash
cd /opt/edgeshark
DOCKER_DEFAULT_PLATFORM= docker compose up -d
```

Accédez à EdgeShark via [localhost:5001](http://localhost:5001).

- **Vue d'EdgeShark** :  
  ![Vue d'EdgeShark](edgeshark.fr.png#center)

- **Intégration avec Wireshark** :  
  En cliquant sur l'icône Wireshark dans EdgeShark, vous pouvez lancer Wireshark localement.  
  ![Interface EdgeShark](edgeshark_interface.fr.png#center)  
  ![EdgeShark et Wireshark](edge_wireshark.fr.png#center)

## Conclusion 🎉

Dans cet article, nous avons parcouru les étapes pour déployer un lab VXLAN EVPN en utilisant Containerlab, DevPod et AWS. Nous avons abordé les points clés suivants :

- **Mise en place des prérequis** pour AWS et Containerlab. 🔑
- **Création d'un fichier de topologie détaillé** pour une architecture spine-leaf. 🗺️
- **Déploiement du lab** en utilisant à la fois un dépôt GitHub et un dossier local. 📥🗂️
- **Démarrage du lab** avec Docker et Containerlab. 🚀🐳
- **Utilisation d'EdgeShark** pour capturer des paquets et intégrer Wireshark pour une analyse approfondie. 🦈🔍

En suivant ces étapes, vous pourrez déployer et gérer facilement un environnement de lab réseau évolutif dans le cloud. Bon networking et profitez bien de vos aventures en lab ! 😄🎊
