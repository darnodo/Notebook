---
title: "Gestionnaire de Certificats Auto-Hébergé"  
date: 2024-08-01T20:00:00+02:00  
weight: 2
cascade:  
  type: docs
---

## 🔗 Sources

- [📖 Documentation Officielle](https://smallstep.com/docs/tutorials/)
- [🛠️ Step-CA en tant que Service systemd](https://angrysysadmins.tech/index.php/2022/09/grassyloki/step-ca-run-as-a-systemd-service/)
- [🔐 Gestion des Certificats avec OpenSSL](https://www.golinuxcloud.com/tutorial-pki-certificates-authority-ocsp/)

## 🤖 À propos de Step-CA

Step-CA est un ensemble d'outils astucieux développé par Smallstep, une entreprise spécialisée dans la gestion sécurisée des identités et l'automatisation des certificats. 🚀  
Sa mission ? Simplifier la mise en place et la gestion de vos propres autorités de certification (AC) avec facilité et sécurité !

### Principales fonctionnalités

1. **Gestion des Autorités de Certification** 🔑  
   Configurez et gérez facilement vos propres AC. Créez des AC racines et intermédiaires, délivrez des certificats et gérez les révocations comme un pro.

2. **Gestion Sécurisée des Clés** 🛡️  
   Respecte les bonnes pratiques pour le stockage et la gestion sécurisés des clés, garantissant que vos clés cryptographiques restent protégées contre les accès non autorisés.

3. **Automatisation et Scalabilité** ⚙️  
   Idéal pour les déploiements de petite à grande envergure. Profitez des API et intégrations qui automatisent la délivrance, le renouvellement et la révocation des certificats pour un cycle de vie sans accrocs.

4. **Sécurité Renforcée** 🔒  
   Grâce à l'utilisation d'algorithmes et de protocoles cryptographiques modernes, Step-CA prend en charge les certificats X.509 conformes aux normes de l'industrie, offrant un chiffrement robuste et des signatures numériques.

5. **Intégration avec l'Infrastructure** 🌐  
   S'intègre parfaitement avec vos outils et systèmes existants. Prend en charge diverses méthodes d'authentification telles que nom d'utilisateur/mot de passe, MFA et fournisseurs d'identité externes.

6. **Auditabilité et Conformité** 📜  
   Avec des capacités complètes de journalisation et d'audit, vous pouvez suivre les activités des certificats et satisfaire aux exigences de conformité en toute simplicité.

7. **API Conviviales pour les Développeurs** 👩‍💻👨‍💻  
   Des API et SDK conçus pour les développeurs facilitent l'intégration de la gestion des certificats dans vos applications et flux de travail personnalisés.

**En résumé :** Step-CA de Smallstep est conçu pour rendre la gestion des autorités de certification à la fois ludique et sans tracas. Grâce à ses fonctionnalités sécurisées, scalables et conviviales, vous pouvez gérer facilement le cycle de vie de vos certificats tout en protégeant votre infrastructure !

## 🚀 Installation

### 🔧 Installation Binaire

#### 1. Step CLI

```bash
wget https://dl.step.sm/gh-release/cli/docs-cli-install/v0.24.3/step-cli_0.24.3_amd64.deb
sudo dpkg -i step-cli_0.24.3_amd64.deb
```

#### 2. Step-CA

```bash
wget https://dl.step.sm/gh-release/certificates/docs-ca-install/v0.24.1/step-ca_0.24.1_amd64.deb
sudo dpkg -i step-ca_0.24.1_amd64.deb
```

#### 3. Création d'un Utilisateur Spécifique

```bash
adduser adminCA
```

#### Configuration

```bash
$ step ca init --password-file=password.txt
✔ Type de déploiement : Autonome
Quel nom souhaitez-vous donner à votre nouvelle PKI ?
✔ (ex. Smallstep) : Lab
✔ (ex. ca.example.com[,10.1.2.3,etc.]) : ca.lab.loc, localhost, 192.168.1.101
À quelle IP et sur quel port votre nouvelle AC sera-t-elle liée ? (:443 se lie à 0.0.0.0:443). 1.101
✔ (ex. :443 ou 127.0.0.1:443) : :443
Quel nom souhaitez-vous donner au premier provisionneur de l'AC ?
✔ (ex. vous@smallstep.com) : contact@lab.loc
Choisissez un mot de passe pour vos clés AC et le premier provisionneur.
✔ [laissez vide et nous en générerons un] : 

Génération du certificat racine... fait ! 🎉  
Génération du certificat intermédiaire... fait ! 🎊

✔ Certificat racine : /home/adminCA/.step/certs/root_ca.crt  
✔ Clé privée racine : /home/adminCA/.step/secrets/root_ca_key  
✔ Empreinte du certificat racine : 7d754397c6897aa87d21e33c64daad7be087dc6fe18bf04627848ae1c8e26a4f  
✔ Certificat intermédiaire : /home/adminCA/.step/certs/intermediate_ca.crt  
✔ Clé privée intermédiaire : /home/adminCA/.step/secrets/intermediate_ca_key  
✔ Dossier de la base de données : /home/adminCA/.step/db  
✔ Configuration par défaut : /home/adminCA/.step/config/defaults.json  
✔ Configuration de l'Autorité de Certification : /home/adminCA/.step/config/ca.json  

Votre PKI est prête ! Pour générer des certificats pour des services individuels, consultez `step help ca`.

💌 **RETROACTION**  
L'utilitaire step n'est pas instrumenté pour les statistiques d'utilisation. Il ne contacte pas un serveur central. Toutefois, vos retours sont très précieux ! N'hésitez pas à nous écrire à feedback@smallstep.com, rejoindre les Discussions GitHub ou nous rejoindre sur Discord à [https://u.step.sm/discord](https://u.step.sm/discord).
```

Démarrer Step-CA :

```bash
step-ca .step/config/ca.json
```

#### Activer ACME

```bash
$ step ca provisioner add acme --type ACME
✔ Configuration de l'AC : /home/adminCA/.step/config/ca.json

Succès ! La configuration de votre `step-ca` a été mise à jour. Pour prendre en compte la nouvelle configuration, envoyez un SIGHUP (kill -1 <pid>) ou redémarrez le processus step-ca. 🎉
```

#### Exécuter Step-CA en tant que Service systemd

Créez un fichier :

```bash
vim /etc/systemd/system/step-ca.service
```

Copiez-collez ce qui suit :

```config
[Unit]
Description=step-ca
After=syslog.target network.target

[Service]
User=adminCA
Group=adminCA
ExecStart=/bin/sh -c '/bin/step-ca /home/adminCA/.step/config/ca.json --password-file=/home/step/.step/pwd >> /var/log/step-ca/output.log 2>&1'
Type=simple
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Créez le répertoire de logs :

```bash
mkdir -p /var/log/step-ca
chown -R adminCA:adminCA /var/log/step-ca
```

Rechargez le démon systemd :

```bash
systemctl daemon-reload
systemctl start step-ca.service
```

### 🐳 Installation avec Docker

```bash
docker run -it -v step:/home/step \
    -p 9000:9000 \
    -e "DOCKER_STEPCA_INIT_NAME=Lab" \
    -e "DOCKER_STEPCA_INIT_DNS_NAMES=caserver.lab.loc,localhost,192.168.1.101" \
    -e "DOCKER_STEPCA_INIT_REMOTE_MANAGEMENT=true" \
    -e "DOCKER_STEPCA_INIT_ACME=true" \
    smallstep/step-ca
```

## 🔑 Accès à l'AC avec un Autre Client

> **NOTE :**  
> Adaptez le port en fonction de votre installation :  
>
> - **Binaire :** port **443**  
> - **Docker :** port **9000**

Installez le Step CLI :

```bash
wget https://dl.step.sm/gh-release/cli/docs-cli-install/v0.24.3/step-cli_0.24.3_amd64.deb
sudo dpkg -i step-cli_0.24.3_amd64.deb
```

Initialisez votre AC :

```bash
step ca bootstrap --ca-url https://caserver.lab.loc:$PORT/ --fingerprint 685059c30eb305db5272a7a199a2b5823624d55c732121ac65c06b0915d3c887
```

> **ASTUCE :**  
> Pour obtenir l'**empreinte**, exécutez simplement :
>
> ```bash
> step certificate fingerprint $(step path)/certs/root_ca.crt
> ```
>
> Pour Docker, consultez les logs du conteneur.

Exemple de sortie :

```bash
admin@User:~$ step ca bootstrap --ca-url https://caserver.lab.loc:$PORT --fingerprint 685059c30eb305db5272a7a199a2b5823624d55c732121ac65c06b0915d3c887
Le certificat racine a été enregistré dans /home/admin/.step/certs/root_ca.crt.
La configuration de l'autorité a été enregistrée dans /home/admin/.step/config/defaults.json.
```

Installez le certificat :

```bash
step certificate install $(step path)/certs/root_ca.crt
```

---

> **ASTUCE :**  
> **Installation sur Debian :**  
>
> - Copiez les fichiers CRT individuels (format PEM) dans `/usr/local/share/ca-certificates/`  
> - Les fichiers doivent appartenir à `root:root` avec les droits `644`  
> - Assurez-vous que le paquet `ca-certificates` est installé (sinon, installez-le)  
> - Ensuite, exécutez en tant que root :
>
> ```bash
> # /usr/sbin/update-ca-certificates
> ```
>
> Tous les certificats seront consolidés dans : `/etc/ssl/certs/ca-certificates.crt`

---

### 📝 Obtenir un Certificat

```bash
admin@User:~$ step ca certificate nas.lab.loc srv.crt srv.key
✔ Provisionneur : contact@lab.loc (JWK) [kid: chyGkrZqp-BGSHUZ8v3jsPipegt2JLcC7y6RPq4OOkU]
Veuillez entrer le mot de passe pour déchiffrer la clé du provisionneur :
✔ AC : https://caserver.lab.loc:443
✔ Certificat : srv.crt
✔ Clé Privée : srv.key
```

---

> **ASTUCE :**  
> Pour effectuer un test de santé :
>
> ```bash
> curl https://caserver.lab.loc:443/health -k
> ```

---

Il pourrait être nécessaire de personnaliser le fichier `ca.json` pour augmenter la durée minimale de validité des certificats. Voici la structure du répertoire :

```bash
.
|-- certs
|   |-- intermediate_ca.crt
|   `-- root_ca.crt
|-- config
|   |-- ca.json
|   `-- defaults.json
|-- db
|   |-- 000000.vlog
|   |-- 000020.sst
|   |-- KEYREGISTRY
|   |-- LOCK
|   `-- MANIFEST
|-- secrets
|   |-- intermediate_ca_key
|   |-- password
|   `-- root_ca_key
`-- templates
```

Exemple de fichier `ca.json` :

```json
{
    "root": "/home/step/certs/root_ca.crt",
    "federatedRoots": null,
    "crt": "/home/step/certs/intermediate_ca.crt",
    "key": "/home/step/secrets/intermediate_ca_key",
    "address": ":9000",
    "insecureAddress": "",
    "dnsNames": [
        "caserver.lab.loc",
        "caserver",
        "localhost",
        "192.168.1.200"
    ],
    "logger": {
        "format": "text"
    },
    "db": {
        "type": "badgerv2",
        "dataSource": "/home/step/db",
        "badgerFileLoadingMode": ""
    },
    "authority": {
        "enableAdmin": true,
        "claims": {
            "maxTLSCertDuration": "4380h"
        }
    },
    "tls": {
        "cipherSuites": [
            "TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256",
            "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256"
        ],
        "minVersion": 1.2,
        "maxVersion": 1.3,
        "renegotiation": false
    }
}
