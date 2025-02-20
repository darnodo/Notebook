---
title: "DevPod sur AWS"
date: 2025-02-11T20:00:00+02:00
weight: 1
cascade:
  type: docs
---

## Sources

- [DevPod](https://devpod.sh/docs/what-is-devpod) ⚙️
- [DevContainer](https://containers.dev) 🐳

## Introduction 🚀

Dans cet article, je souhaite présenter un outil fantastique qui fait partie de la même famille que GitPod et Codespaces : **DevPod** ! Il vous permet de créer des environnements de développement sans effort — sans se retrouver bloqué par un fournisseur. 🔒❌

DevPod est basé sur l'architecture **DevContainer** et utilise les spécifications contenues dans un fichier [devcontainer.json](https://containers.dev) pour lancer votre environnement de développement. Personnellement, je l'utilise pour déployer rapidement des maquette réseau avec ContainerLab. 💻🧰

## Qu'est-ce qu'un DevContainer ? 🤔

Un conteneur de développement (souvent appelé « dev container ») vous permet d'utiliser un conteneur comme un environnement de développement complet. (Consultez la documentation officielle de [containers.dev](https://containers.dev) pour plus de détails.)

Décomposons cela :

Avez-vous déjà entendu l'expression « Ça marche sur ma machine » ? Si vous êtes développeur, vous avez sans doute rencontré ce problème en collaborant avec d'autres. Mais devinez quoi ? Les **DevContainers** résolvent le problème de la dérive des environnements en fournissant des environnements Docker cohérents et reproductibles. Adieu les maux de tête liés à l'installation ! 💆‍♀️

Avec les DevContainers, il vous suffit de :

1. Un dossier `.devcontainer` dans votre projet.
2. Un fichier `devcontainer.json` qui indique à VS Code comment configurer le conteneur.
3. Docker installé localement

La pierre angulaire du DevContainer est le fichier `devcontainer.json`. Par exemple :

```json
{
  "name": "Python DevContainer",
  "image": "mcr.microsoft.com/devcontainers/python:3.10",
  "features": {
    "docker-in-docker": "latest"
  },
  "extensions": [
    "ms-python.python",
    "ms-azuretools.vscode-docker"
  ]
}
```

**Que se passe-t-il ici ?** 🕵️‍♀️

- **"image"** – Cela utilise un environnement Python 3.10 prêt à l'emploi.
- **"features"** – Cela ajoute la prise en charge de Docker dans le conteneur.
- **"extensions"** – Installe des extensions utiles pour Python et Docker dans VS Code.

Les DevContainers sont formidables pour le développement local, mais parfois vous avez besoin de plus de puissance — peut-être que vos charges de travail sont énormes, ou que vous souhaitez exécuter des laboratoires spécialisés. C'est là qu'intervient **DevPod**. 💪  
**Pour pouvoir déployer facilement des laboratoires sur le Cloud**

## Qu'est-ce que DevPod ? 🤖

**DevPod** est un outil open-source qui vous permet de lancer des environnements de développement soit sur votre machine locale, soit dans le cloud de votre choix. Imaginez une version auto-hébergée et ultra-personnalisable de GitHub Codespaces. 🎉

Dans mes aventures quotidiennes en réseau, je déploie des configurations basées sur ContainerLab avec des DevContainers sur AWS. Voyons comment vous pouvez utiliser DevPod pour faire exactement cela (les détails sur ContainerLab suivront dans un autre article, promis !). 😜

## Fournisseur AWS 🌐

DevPod utilise des **Providers** (fournisseurs), qui sont des modules de configuration définissant où et comment DevPod lance votre environnement. Voici la liste des fournisseurs :

![Provider_list](provider.fr.png#center)

Nous allons nous concentrer sur le **Provider AWS** — bien qu'il existe de nombreuses options de configuration :

![Provider_list](aws_options.fr.png#center)

Avant de paniquer devant tous ces réglages, ne vous inquiétez pas. Si vous ne faites que quelques expérimentations, les valeurs par défaut conviennent généralement. 🙌

> [!NOTE] **Les avantages de l'open-source** 🎁
>
> Le fait que DevPod soit open-source signifie que vous pouvez jeter un œil sous le capot pour voir exactement comment il fonctionne. Consultez le code AWS [ici](https://github.com/loft-sh/devpod-provider-aws/tree/main) si vous êtes curieux.

## Fonctionnement du code AWS

Examinons ce que fait DevPod sur AWS en regardant le [code aws.go](https://github.com/loft-sh/devpod-provider-aws/blob/main/pkg/aws/aws.go). À un niveau global, il gère :

1. **Initialisation** : Lecture de la configuration et mise en place des clients du SDK AWS (avec des identifiants personnalisés si nécessaire).
2. **Réseautage** : Recherche ou création du sous-réseau, VPC et groupes de sécurité appropriés.
3. **Sélection de l'AMI** : Choix d'une AMI adaptée (par défaut, une image Ubuntu 22.04 récente) et détermination du périphérique racine.
4. **Configuration IAM** : Vérification qu'un rôle d'instance approprié et un profil d'instance existent, avec les politiques associées.
5. **Cycle de vie de l'instance** : Création, démarrage, arrêt, vérification du statut et suppression des instances.
6. **Injection de données utilisateur** : Génération d'un script (encodé en Base64) qui configure l'instance (ajoute des utilisateurs et des clés SSH) au premier démarrage.
7. **DNS optionnel** : Gestion des enregistrements Route 53 pour l'instance si la configuration l'exige.

De mon point de vue, deux points se distinguent comme étant potentiellement les plus critiques :

- **(#4) Configuration IAM**
- **(#6) Injection de données utilisateur**

### Pourquoi les points #4 et #6 sont-ils « délicats » ?

- **Configuration IAM** est principalement gérée par la fonction `CreateDevpodInstanceProfile`. Elle crée un rôle nommé `devpod-ec2-role` qui peut effectuer les opérations suivantes :
  - **Opérations EC2** : Par exemple, il peut décrire ou arrêter des instances — en particulier l'instance qui lui est associée.
  - **Opérations SSM** : En attachant la politique AmazonSSMManagedInstanceCore, l'instance peut être gérée par AWS Systems Manager.
  - **Opérations KMS (optionnel)** : Si configuré, il peut exécuter `kms:Decrypt` sur une clé KMS spécifique, ce qui est utile pour gérer les données de session ou les secrets.

- **Injection de données utilisateur** est essentiellement un script de démarrage inséré dans l'instance sous forme de Base64. Ce script configure un utilisateur `devpod` avec des droits sudo, crée les dossiers SSH et insère votre clé publique dans le fichier approprié. Dans le code, [cela ressemble à](https://github.com/loft-sh/devpod-provider-aws/blob/9d2730c34ecee40cb42596c602381b92ad9c6682/pkg/aws/aws.go#L967-L980) :

  ```bash
  useradd devpod -d /home/devpod
  mkdir -p /home/devpod
  if grep -q sudo /etc/groups; then
    usermod -aG sudo devpod
  elif grep -q wheel /etc/groups; then
    usermod -aG wheel devpod
  fi

  echo "devpod ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/91-devpod
  mkdir -p /home/devpod/.ssh
  echo " + string(publicKey) + " >> /home/devpod/.ssh/authorized_keys
  chmod 0700 /home/devpod/.ssh
  chmod 0600 /home/devpod/.ssh/authorized_keys
  chown -R devpod:devpod /home/devpod
  ```

Comme DevPod est open-source, vous pouvez facilement vérifier par vous-même. C'est un excellent outil d'apprentissage si vous souhaitez comprendre tous les rouages ! 🔩

## Rôles et politiques IAM

Vous devrez créer un utilisateur IAM et lui attacher une politique IAM qui accorde juste assez de permissions pour DevPod. Par exemple :

- **Actions EC2 :**
  - Description : `ec2:DescribeSubnets`, `ec2:DescribeVpcs`, `ec2:DescribeImages`, `ec2:DescribeInstances`, `ec2:DescribeSecurityGroups`
  - Gestion des instances : `ec2:RunInstances`, `ec2:StartInstances`, `ec2:StopInstances`, `ec2:TerminateInstances`, `ec2:CancelSpotInstanceRequests`
  - Groupes de sécurité & Tags : `ec2:CreateSecurityGroup`, `ec2:AuthorizeSecurityGroupIngress`, `ec2:CreateTags`
- **Actions IAM :**
  - `iam:GetInstanceProfile`, `iam:CreateRole`, `iam:PutRolePolicy`, `iam:AttachRolePolicy`, `iam:CreateInstanceProfile`, `iam:AddRoleToInstanceProfile`
- **Route 53 (optionnel) :**
  - `route53:ListHostedZones`, `route53:GetHostedZone`, `route53:ChangeResourceRecordSets`

## Configuration AWS 🏗️

J'utilise généralement la console web AWS pour configurer cela, mais vous pouvez tout à fait le faire via la CLI.

### Étape 1 : Connectez-vous à la console AWS

1. Rendez-vous sur [AWS Management Console](https://aws.amazon.com/console/).
2. Utilisez un compte disposant des droits appropriés pour créer des ressources IAM.

### Étape 2 : Créer une politique IAM personnalisée

#### **A. Accédez à la console IAM**

- Dans le menu AWS, trouvez **IAM**.

#### **B. Créer une nouvelle politique**

1. Cliquez sur **Policies** dans le menu de gauche.
2. Cliquez sur **Create policy**.

#### **C. Passez à l'onglet JSON**

- Collez quelque chose comme ceci, en ajustant si nécessaire :

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "EC2Actions",
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeSubnets",
        "ec2:DescribeVpcs",
        "ec2:DescribeImages",
        "ec2:DescribeInstances",
        "ec2:DescribeSecurityGroups",
        "ec2:RunInstances",
        "ec2:StartInstances",
        "ec2:StopInstances",
        "ec2:TerminateInstances",
        "ec2:CancelSpotInstanceRequests",
        "ec2:CreateSecurityGroup",
        "ec2:AuthorizeSecurityGroupIngress",
        "ec2:CreateTags",
        "ec2:DeleteTags"
      ],
      "Resource": "*"
    },
    {
      "Sid": "IAMActions",
      "Effect": "Allow",
      "Action": [
        "iam:GetInstanceProfile",
        "iam:CreateRole",
        "iam:PutRolePolicy",
        "iam:AttachRolePolicy",
        "iam:CreateInstanceProfile",
        "iam:AddRoleToInstanceProfile"
      ],
      "Resource": "*"
    },
    {
      "Sid": "Route53Actions",
      "Effect": "Allow",
      "Action": [
        "route53:ListHostedZones",
        "route53:GetHostedZone",
        "route53:ChangeResourceRecordSets"
      ],
      "Resource": "*"
    }
  ]
}
```

- Cliquez sur **Next** (les tags sont optionnels), puis sur **Next: Review**.

#### **D. Vérifier et créer**

1. Nommez-la `DevpodToolPolicy` (ou comme vous le souhaitez).
2. Ajoutez une description facultative.
3. Cliquez sur **Create policy**.

### Étape 3 : Créer ou mettre à jour l'utilisateur IAM

#### **A. Créer un nouvel utilisateur (si nécessaire)**

1. Cliquez sur **Users** dans IAM.
2. Cliquez sur **Add user**.
3. Donnez-lui un nom (par exemple, `devpod-tool-user`).
4. Choisissez **Programmatic access** si vous souhaitez un accès via la CLI. 🤖
5. Cliquez sur **Next**.

#### **B. Attacher votre nouvelle politique**

1. Sur la page des permissions, choisissez **Attach policies directly**.
2. Cochez `DevpodToolPolicy`.
3. Cliquez sur **Create**.
4. Et voilà !

### Étape 4 : Vérifier et c'est fini

Retournez dans **Users** → **devpod-tool-user** → **Permissions** pour confirmer que `DevpodToolPolicy` est bien attachée. ✅

### Étape 5 : Utilisez ces identifiants

- Si vous avez créé un utilisateur programmatique, n'oubliez pas de noter l'**Access Key ID** et le **Secret Access Key**.

![devpod_user_sumup](devpod_user.fr.png#center)

**Bonus** : Notez votre **ID VPC** (dans la section VPC sur AWS). Vous en aurez besoin lors de la configuration de DevPod.

## Configurer DevPod 🛠️

### 1. Configurer le profil AWS

```bash
aws configure --profile Devpod
```

Lorsqu'on vous le demande :

1. L'Access Key ID
2. Le Secret Access Key
3. La région par défaut (par exemple, `eu-west-1`)
4. Le format de sortie (je le laisse généralement vide)

### 2. Ajouter le profil à DevPod

1. Dans DevPod, créez un nouveau provider et choisissez **AWS**.
2. Sélectionnez la **région AWS** (par exemple, `eu-west-1`).
3. Développez les options AWS.
4. **Taille du disque AWS** : par exemple, 40 GB pour commencer.
5. **Type d'instance** : par exemple, `t2.small`.
6. **Profil AWS** : sélectionnez `Devpod` (ou le nom que vous avez choisi).
7. **ID VPC AWS** : ajoutez votre VPC.
8. Vous pouvez laisser le reste par défaut.

Cliquez sur **Add Provider**.

![added_new_provider](new_provider.fr.png#center)

## Tester un déploiement 🧪

### Déployer

Nous allons effectuer un test rapide en utilisant l'une des images Docker préconstruites :

1. Rendez-vous dans **Workspaces** dans DevPod.
2. Cliquez sur **Create Workspace**.
3. Choisissez votre nouveau provider **AWS**.
4. Choisissez votre IDE préféré (VS Code, etc.).
5. À droite, sélectionnez un exemple de démarrage rapide (par exemple, Python). 🐍
6. Cliquez sur **Create Workspace**.

![new_worspace](new_worskapce.fr.png#center)

Attendez quelques instants, et votre environnement basé sur le cloud apparaîtra dans VS Code. 🎊

![vscode](vscode.fr.png#center)

### Arrêter

Lorsque vous n'utilisez pas l'environnement, cliquez sur **Stop** pour éteindre l'instance EC2. Vous ne paierez que pour le stockage — aucun temps de calcul. Idéal pour votre portefeuille. 💰

![Stopped Instance](stopped_instance.fr.png#center)

### Supprimer

Supprimer le workspace supprime toutes les ressources AWS associées à cet environnement, vous ne paierez donc pas un centime. Mais vous devrez redéployer si vous souhaitez l'utiliser à nouveau. ♻️

![Delete Instance](delete_instance.fr.png#center)

## Conclusion 💡

En combinant **DevContainers** et **DevPod** sur **AWS**, vous pouvez créer des environnements de développement flexibles et autogérés qui évoluent avec votre croissance — sans être enfermé dans des plateformes spécifiques à un fournisseur. Dites adieu aux problèmes du « Ça marche sur ma machine ! » et accueillez un codage sans friction. 🚀✨

Amusez-vous bien ! 🎉
