---
title: "VXLAN pour les débutants"
date: 2024-08-01T20:00:00+02:00
cascade:
  type: docs
---

## Comprendre VLAN et VXLAN : Simplifié pour les non-techniciens

Dans le monde rapide de la technologie, comprendre les concepts de réseau peut être intimidant, surtout si vous n'êtes pas un expert en la matière.  
Aujourd'hui, nous allons décomposer deux concepts de réseau importants : **VLAN** et **VXLAN**, en utilisant des analogies simples et des explications claires.  
Nous aborderons également leurs limites, leurs cas d’usage concrets, ainsi que quelques notions techniques pour les plus curieux.  

Allons-y ! 🚀

## Qu'est-ce qu'un VLAN ? 🏢

**VLAN (Virtual Local Area Network)**, ou Réseau Local Virtuel, c'est comme organiser un grand immeuble de bureaux avec plusieurs départements : Marketing, Ventes, RH, et Informatique. Pour maintenir l'ordre, chaque département obtient son propre étage. De cette manière, les personnes du Marketing restent à leur étage, celles des Ventes au leur, et ainsi de suite.

Un **VLAN** fonctionne de manière similaire pour les réseaux informatiques. Il divise un grand réseau physique en réseaux plus petits et isolés. Chaque VLAN est comme un étage séparé pour un département, permettant aux dispositifs au sein du même VLAN de communiquer facilement, tout en gardant le trafic séparé des autres VLAN.

### Points clés sur le VLAN ✅

- **Séparation :** Garde les différents groupes (comme les départements) séparés.  
- **Efficacité :** Réduit le trafic inutile et les potentiels problèmes de réseau.  
- **Sécurité :** Limite l'accès et renforce la sécurité en isolant les groupes.

### Limites du VLAN ⚠️

- **Limite d’ID :** Historiquement, un VLAN est identifié sur 12 bits, permettant jusqu’à 4094 VLANs (de 1 à 4094). Pour une grande entreprise ou un datacenter, cela peut s’avérer insuffisant.  
- **Isolation locale :** Les VLANs sont plutôt conçus pour un usage local (un même site ou un ensemble de switches connectés localement). Dès qu’on veut étendre ce concept à plusieurs sites, on a besoin de solutions plus avancées.

## Qu'est-ce que le VXLAN ? 🌆

**VXLAN (Virtual Extensible LAN)** va plus loin. Imaginez que votre entreprise grandisse et s'étende à plusieurs immeubles à travers la ville. Vous voulez toujours que les départements se sentent comme s'ils étaient sur leurs propres étages, même s'ils sont maintenant répartis dans différents endroits. Pour ce faire, vous créez un système virtuel qui connecte tous les étages à travers les bâtiments, de sorte que le Marketing au 3e étage d'un bâtiment soit toujours virtuellement connecté au Marketing du 3e étage d'un autre bâtiment.

Le **VXLAN** fait cela pour les réseaux. Il étend les VLANs à travers plusieurs emplacements physiques en utilisant une technique appelée **tunneling**. Ainsi, les dispositifs dans le même VLAN peuvent communiquer comme s'ils étaient sur le même réseau local, même s'ils sont éloignés géographiquement.

### Points clés sur le VXLAN ⭐

- **Évolutivité :** Étend les réseaux à différents emplacements, et dépasse la limite de 4094 VLANs.  
- **Flexibilité :** Permet des conceptions de réseau plus grandes et dynamiques.  
- **Connectivité :** Assure une communication fluide à travers des réseaux dispersés.

## Plongée technique dans le VXLAN 🔍

Le **VXLAN** a été développé pour répondre aux limites des VLANs traditionnels (scalabilité, étendue géographique). Il utilise un identifiant de réseau VXLAN (**VNI**) de 24 bits pour identifier jusqu'à **16 millions** de segments logiques, surpassant ainsi largement la limite de 4094 VLANs.

En raison de la virtualisation, les tables d'adresses MAC dans les datacenters peuvent devenir très grandes, tandis que les switches physiques ont des capacités limitées. Le VXLAN répond à ce défi en utilisant l’encapsulation **MAC-in-UDP**, permettant de transporter des trames Ethernet (couche 2) à travers un réseau IP (couche 3).

### Comment ça marche ? 🤔

L’objectif du **VXLAN** est de **prolonger la couche 2** à travers un réseau de couche 3 (IP). Cela revient à « tromper » la couche 3 pour faire croire à l’utilisateur ou à la machine virtuelle qu’il est toujours dans le même réseau local (couche 2).

> **En clair :** On encapsule les trames Ethernet (couche 2) dans un paquet UDP (couche 4), lui-même transporté par IP (couche 3).  

![OSI Layers](media_layers.fr.png#center)

> [!NOTE]**Les couches “matérielles”**  
>
> - La couche **Liaison (2)** est communément gérée par des switches.  
> - La couche **Réseau (3)** est communément gérée par des routeurs.  

En encapsulant la couche 2 dans la couche 3, on profite des avantages du routage IP (souplesse, scalabilité) tout en conservant l’isolation et la simplicité de la couche 2 pour les applications et machines virtuelles.

### VXLAN expliqué par l’analogie du transport de conteneurs 🚚 🚂

#### 1. Les camions (couches basses)  

Imaginez des camions sur la route. Leur mission est d’acheminer des conteneurs (vos données) d’un point A à un point B. Ces camions représentent la **couche Ethernet** (niveau 2), où chaque véhicule (frame) possède une « plaque d’immatriculation » (adresse MAC).

#### 2. Le train (tunnel VXLAN)  

Quand il s’agit de couvrir de plus longues distances ou de traverser des infrastructures variées, charger les camions sur un train devient plus efficace. Ici, **le train représente VXLAN** : il encapsule les camions (frames Ethernet) dans un wagon (le tunnel). Chaque train est identifié par un **VNI (VXLAN Network Identifier)**, un peu comme un numéro de convoi pour chaque ligne de fret.

#### 3. Les voies ferrées (réseau IP)  

Le train roule sur des rails (le **réseau IP**, couche 3). Les voies ferrées sont déjà construites et gérées pour trouver le meilleur itinéraire : elles assurent la convergence des routes et peuvent rediriger le trafic en cas de problème (panne, congestion, etc.). De la même façon, le réseau IP choisit automatiquement le chemin le plus optimal pour transporter les paquets VXLAN.

### Points clés à retenir

- **Superposition (Overlay)** : VXLAN est un système de transport « par-dessus » la couche 3 (les rails). Il permet d’interconnecter plusieurs réseaux de niveau 2 (les camions) comme s’ils n’en formaient qu’un seul.  
- **Double adressage** :  
  - Les camions (frames Ethernet) s’identifient via des **adresses MAC** (plaque d’immatriculation).  
  - Le train (tunnel VXLAN) utilise les **adresses IP** (plan de route) pour circuler sur les rails.  
- **Isolation et segmentation** : Comme plusieurs trains peuvent rouler sur la même ligne ferroviaire, il est possible d’exploiter différents tunnels VXLAN (chacun avec son VNI) sur la même infrastructure IP.  
- **Élasticité et fiabilité** : En s’appuyant sur la couche 3, VXLAN profite de toutes les optimisations du routage IP (recalcul d’itinéraires, tolérance aux pannes, etc.).  

![Container transport](transports.fr.png#center)

## Cas d'usage concrets 🏭

- **Multi-datacenter :** Pour connecter plusieurs centres de données géographiquement dispersés, tout en gardant la sensation d’un réseau unique au niveau 2.  
- **Cloud hybride :** Étendre un réseau d’entreprise vers un fournisseur de cloud public ou privé sans reconfigurer tout le plan d’adressage.  
- **Migration de machines virtuelles :** Permettre la migration (VM Mobility) entre sites distants sans perdre la connectivité de couche 2.  
- **Virtualisation massive :** Dans les environnements très denses (par ex. des centaines de milliers de machines virtuelles), l’identifiant VNI de 24 bits est indispensable.

## Contrôle du VXLAN : BGP EVPN et autres protocoles 🤝

Dans les déploiements modernes, surtout en datacenter, le VXLAN n’est pas simplement configuré de manière statique. Il est souvent associé à un **contrôle de plan** via le protocole **BGP EVPN (Ethernet VPN)**.  

- **BGP EVPN :** Permet d’échanger les informations de tables MAC et IP entre les équipements, facilitant l’automatisation et l’évolutivité.  
- **Autres technologies :** Historiquement, on pouvait croiser d’autres protocoles d’overlay (NVGRE, STT), mais VXLAN s’est imposé comme standard de fait.

## Considérations de performance ⚙️

- **Surcharge d’encapsulation :** Le VXLAN ajoute un en-tête supplémentaire (8 octets + en-tête UDP/IP). Cela peut impacter la **taille maximale de trame (MTU)** et il faut souvent configurer un **Jumbo MTU** (généralement 9000 octets) pour éviter la fragmentation des paquets.  
- **Résilience du réseau IP :** La fiabilité du tunnel VXLAN dépend de la qualité du réseau IP sous-jacent (routes, congestion, etc.).

## Exemple de configuration (pour les plus curieux) 💡

Voici un **extrait simplifié** d’une configuration VXLAN sur un équipement Cisco NX-OS (les syntaxes varient selon les constructeurs) :

```plaintext
interface nve1
  no shutdown
  source-interface loopback1
  member vni 5001
    ingress-replication protocol static
    mcast-group 239.1.1.1
```

- **interface nve1 :** On crée une interface “NVE” (Network Virtualization Endpoint) pour gérer l'encapsulation VXLAN.  
- **source-interface loopback1 :** L’adresse IP de l’interface loopback1 sera utilisée pour établir les tunnels.  
- **member vni 5001 :** On associe un VNI (VXLAN Network Identifier) à notre overlay réseau.  

*Note :* Dans les environnements plus complexes, on configure également le plan de contrôle (par ex. BGP EVPN).

## En résumé 🎯

- **VLAN**  
  C’est comme avoir des étages séparés pour différents départements dans un bâtiment, en gardant leurs activités isolées. 🏢  
  \- **Limite majeure** : 4094 VLANs maximum et une portée souvent limitée à un même site.

- **VXLAN**  
  C’est comme connecter ces étages séparés à travers plusieurs bâtiments, tout en gardant l’illusion qu’ils se trouvent dans un même immeuble. 🌆  
  \- **Avantages clés** : Énorme capacité d’adressage (16 millions de segments), extension L2 sur L3, flexibilité pour la virtualisation et le multi-site.

Le **VXLAN** répond aux besoins d'isolation à grande échelle, dépasse les limitations des tables d'adresses MAC des commutateurs et permet un déploiement flexible des services. De plus, associé à un plan de contrôle efficace (BGP EVPN), il simplifie grandement la gestion des réseaux modernes en **overlay**.

### Conclusion 🏁

En bref, si vous recherchez une **segmentation de base** pour votre réseau local, un **VLAN** est largement suffisant. Mais dès lors que vous voulez relier plusieurs sites, créer un réseau hautement virtualisé, ou dépasser la limite traditionnelle de 4094 VLANs, le **VXLAN** devient incontournable.

Que vous soyez un passionné de **Lab réseau**, un ingénieur NetOps, ou tout simplement curieux des dessous de l’infrastructure informatique, comprendre ces deux notions vous aidera à mieux appréhender la magie qui se déroule lorsque vos données circulent de plus en plus loin, tout en conservant l’illusion d’être “chez soi” sur le même réseau local !

> [!TIP] **Envie d’aller plus loin ?**  
>
> - Regardez du côté du **BGP EVPN** pour le plan de contrôle du VXLAN.  
> - Explorez la **configuration Jumbo MTU** pour optimiser vos performances.  
> - Comparez VXLAN avec d’autres protocoles (NVGRE, GENEVE) pour comprendre les choix de design réseau.
