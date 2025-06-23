---
title: "SSL Bumping : Comment analyser le trafic SSL"
date: 2025-06-22T20:00:00+02:00
weight: 3
cascade:
  type: docs
---

## Introduction : Quand le Cadenas Vert Devient Transparent 🔓🔍

Vous vous souvenez de notre fameux [**cadenas vert**](/documentation/securité/cadenas_vert/ssl/) dont on a parlé dans l'article précédent ? Cette petite icône qui nous rassure, qui nous dit "tout va bien, vos secrets sont en sécurité" ? Eh bien aujourd'hui, on va voir comment certains acteurs arrivent à **regarder à travers** ce cadenas ! 😱

Mais rassurez-vous, on ne va pas devenir des pirates ! On va explorer une technique légitime (mais controversée) appelée **SSL Bumping** ou **SSL/TLS Interception**. C'est un peu comme avoir des lunettes à rayons X pour voir ce qui se cache derrière le chiffrement HTTPS.

**Pourquoi c'est important ?** 🤔 Parce que cette technique est utilisée partout : dans les entreprises pour sécuriser leurs réseaux, dans les écoles pour filtrer le contenu, et même parfois... par des acteurs moins bien intentionnés. Comprendre comment ça fonctionne, c'est comprendre à la fois comment se protéger et comment protéger les autres.

Prêts pour une plongée dans les coulisses du HTTPS ? On va décortiquer ensemble cette technique fascinante, et je vous montrerai même comment la mettre en pratique avec un lab que j'ai préparé : [squid-ssl-bumping-lab](https://github.com/darnodo/squid-ssl-bumping-lab) 🧪

## Partie 1 : Le SSL Bumping, C'est Quoi au Juste ? 🎭

### Le Principe : L'Homme du Milieu... Mais Gentil ! 👤

Imaginez cette scène : vous voulez envoyer une lettre secrète à votre ami(e). Normalement, vous la mettez dans une enveloppe scellée (le chiffrement HTTPS) et hop, direction la boîte aux lettres. Personne ne peut la lire en chemin, n'est-ce pas ?

Maintenant, imaginez que le facteur (disons, le proxy de votre entreprise) ait besoin de vérifier que vous n'envoyez pas de plans secrets de l'entreprise à la concurrence. Comment fait-il ? Il ne peut pas lire à travers l'enveloppe !

C'est là qu'intervient le **SSL Bumping** :

![SSL Bumping Expliqué](images/ssl_bumping_explique.fr.svg#center)

Le proxy va :
1. **Intercepter** votre connexion HTTPS
2. **Déchiffrer** votre trafic (comme ouvrir l'enveloppe)
3. **Analyser** le contenu
4. **Re-chiffrer** et transmettre au serveur final

En gros, au lieu d'avoir **une** connexion sécurisée de bout en bout, vous en avez **deux** : une entre vous et le proxy, une autre entre le proxy et le serveur final. Le proxy peut voir tout ce qui passe !

> [!NOTE] **Terminologie : SSL Bumping, HTTPS Interception, MITM... Késako ? 🤓**
>
> Ces termes désignent tous la même chose :
> - **SSL Bumping** : Le terme historique utilisé par Squid (un proxy populaire)
> - **TLS Interception** : La version moderne (on utilise TLS, pas SSL)
> - **HTTPS Inspection** : Le terme "corporate-friendly"
> - **MITM (Man-In-The-Middle)** : Le terme technique général
>
> Dans cet article, j'utiliserai principalement "SSL Bumping" par habitude, mais c'est bien de TLS qu'on parle !

### Pourquoi Faire Ça ? Les Cas d'Usage Légitimes 🏢

Avant de crier au scandale, sachez que le SSL Bumping a des utilisations parfaitement légitimes :

1. **Sécurité d'Entreprise 🛡️**
   - Détecter les malwares qui se cachent dans le trafic HTTPS
   - Empêcher les fuites de données sensibles
   - Bloquer l'accès à des sites malveillants

2. **Conformité et Contrôle 📋**
   - Respecter les réglementations (certains secteurs doivent tout archiver)
   - Filtrer le contenu inapproprié dans les écoles
   - Optimiser la bande passante en cachant le contenu

3. **Débogage et Développement 🐛**
   - Analyser les API de vos applications
   - Debugger les problèmes de connexion
   - Tester la sécurité de vos services

> [!WARNING] **Attention : Avec de Grands Pouvoirs... 🚨**
>
> Le SSL Bumping est une technique **très puissante** qui peut être utilisée à mauvais escient :
> - Espionnage des employés
> - Vol d'identifiants et de données personnelles
> - Violation de la vie privée
>
> Son utilisation doit **toujours** être transparente, légale et éthique !

## Partie 2 : Comment Ça Marche Techniquement ? 🔧⚙️

Bon, assez de théorie ! Rentrons dans le vif du sujet. Comment un proxy arrive-t-il à se faire passer pour le serveur légitime sans que votre navigateur ne crie au loup ? C'est toute une chorégraphie !

### Étape 1 : La Fausse Identité 🎭

Pour que le SSL Bumping fonctionne, le proxy doit pouvoir **créer de faux certificats** à la volée. Comment ? Il devient sa propre Autorité de Certification (CA) !

Le proxy génère :
1. **Sa propre CA** avec une paire de clés (publique/privée)
2. **Un faux certificat** pour chaque site que vous visitez
3. Ce faux certificat est signé par sa CA "maison"

### Étape 2 : Le Tour de Passe-Passe 🎩✨

Voici la danse complète quand vous visitez `https://www.example.com` :

![SSL Bumping Expliqué](images/generation_des_faux_certificats.fr.svg)

Le proxy joue un double jeu :
- Côté serveur : Il se fait passer pour un client normal
- Côté client : Il se fait passer pour le serveur !

> [!TIP] **Mais Attendez... Mon Navigateur N'est Pas Stupide ! 🤨**
>
> Exactement ! Normalement, votre navigateur devrait hurler "CERTIFICAT INVALIDE !"
> Pour que ça fonctionne, il faut que le certificat de la CA du proxy soit **installé comme CA de confiance** sur votre machine.
>
> C'est pourquoi :
> - Les entreprises installent leur CA sur tous les postes
> - Les antivirus vous demandent d'installer leur certificat
> - Vous devez faire super attention aux CA que vous installez !

### Étape 3 : L'Analyse du Trafic 🔍

Une fois le tunnel établi, le proxy peut voir **tout** :

- Les URL complètes (pas juste le domaine)
- Les données POST (formulaires, mots de passe...)
- Les cookies et tokens d'authentification
- Le contenu des pages
- Les fichiers téléchargés

Il peut alors :
- **Logger** tout pour analyse ultérieure
- **Bloquer** certains contenus
- **Modifier** les requêtes/réponses (dangereux !)
- **Scanner** pour des malwares

## Partie 3 : Mise en Pratique avec Squid 🦑

Assez parlé, passons à la pratique ! J'ai créé un lab complet pour que vous puissiez tester le SSL Bumping dans un environnement contrôlé : [squid-ssl-bumping-lab](https://github.com/darnodo/squid-ssl-bumping-lab).

### Qu'est-ce que Squid ? 🦑

Squid est un proxy cache très populaire qui supporte le SSL Bumping. C'est l'outil parfait pour notre expérimentation !

### Architecture du Lab 🏗️

![SSL Bumping Expliqué](images/architecture_du_lab_ssl.fr.svg)

### Installation Rapide 🚀

1. **Clonez le repository** :
   ```bash
   git clone https://github.com/darnodo/squid-ssl-bumping-lab
   cd squid-ssl-bumping-lab
   ```

2. **Générez votre CA** (ou utilisez une existante) :
   ```bash
   # Créer le dossier ssl/
   mkdir -p ssl

   # Générer une CA auto-signée
   openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 \
     -keyout ssl/squid-ca-key.pem \
     -out ssl/squid-ca-cert.pem \
     -subj "/CN=Squid CA/O=Mon Lab/C=FR"
   ```

3. **Lancez le lab** :
   ```bash
   # Sans export de logs
   docker-compose up --build -d

   # Ou avec Fluent Bit pour exporter les logs
   docker-compose --profile logging up --build -d
   ```

4. **Configurez votre navigateur** :
   - Proxy HTTP/HTTPS : `localhost:3128`
   - Importez `ssl/squid-ca-cert.pem` comme CA de confiance

> [!WARNING] **Lab Only ! Ne Faites Jamais Ça en Production ! 🚫**
>
> Ce lab est **uniquement** pour l'apprentissage !
> - N'utilisez JAMAIS cette CA en dehors du lab
> - Ne faites pas de SSL Bumping sans autorisation explicite
> - Supprimez la CA de votre système après les tests

### Ce Que Vous Verrez 👀

Une fois configuré, visitez n'importe quel site HTTPS et regardez les logs :

```bash
# Voir les logs en temps réel
docker-compose exec squid tail -f /var/log/squid/access.log
```

Vous verrez quelque chose comme :
```
1719151234.567   342 192.168.1.100 TCP_MISS/200 15234 GET https://www.example.com/api/data - HIER_DIRECT/93.184.216.34
```

Magie ! Vous voyez l'URL **complète** même en HTTPS ! 🎉

### Analyser les Certificats 🔍

Pour voir le faux certificat en action :
1. Visitez un site HTTPS via le proxy
2. Cliquez sur le cadenas dans votre navigateur
3. Regardez les détails du certificat
4. Surprise ! Il est signé par "Squid CA" et non par la vraie CA !

## Partie 4 : Se Protéger du SSL Bumping 🛡️

Maintenant qu'on sait comment ça marche, comment s'en protéger ? Ou du moins, comment le détecter ?

### Pour les Utilisateurs : Les Signaux d'Alerte 🚨

1. **Vérifiez les Certificats** 📜
   - Cliquez sur le cadenas et examinez le certificat
   - L'émetteur devrait être une CA connue (Let's Encrypt, DigiCert...)
   - Méfiez-vous des CA d'entreprise ou inconnues

2. **Certificate Pinning** 📌
   - Certaines applis vérifient que le certificat est EXACTEMENT celui attendu
   - Si c'est différent = refus de connexion
   - Les apps bancaires utilisent souvent cette technique

> [!TIP] **Le Test Ultime : Comparez les Empreintes ! 🔬**
>
> 1. Notez l'empreinte (fingerprint) SHA-256 du certificat depuis votre réseau
> 2. Comparez avec l'empreinte depuis un autre réseau (4G mobile par exemple)
> 3. Si elles sont différentes = SSL Bumping probable !

### Pour les Entreprises : Les Bonnes Pratiques 💼

Si vous DEVEZ utiliser le SSL Bumping :

1. **Transparence Totale** 📢
   - Informez clairement les utilisateurs
   - Affichez une politique claire d'utilisation
   - Respectez la législation locale

2. **Sécurité Maximale** 🔒
   - Protégez la clé privée de votre CA comme un trésor
   - Utilisez un HSM (Hardware Security Module) si possible
   - Limitez la durée de vie des certificats

3. **Exceptions Nécessaires** ⛔
   - Ne bumpez JAMAIS les sites bancaires
   - Excluez les sites de santé
   - Respectez la vie privée (messageries, etc.)

### L'Alternative : Le SNI Filtering 🎯

Plutôt que de déchiffrer, pourquoi ne pas simplement filtrer par nom de domaine ?

```
Client ──"Je veux example.com" (en clair)──> Proxy ──> Décision
                    ↑
                SNI Header
            (Server Name Indication)
```

Le SNI permet de voir le domaine sans déchiffrer. C'est moins intrusif mais aussi moins précis (on ne voit que le domaine, pas l'URL complète).

## Partie 5 : L'Éthique et la Légalité ⚖️

### Les Questions Éthiques 🤔

Le SSL Bumping soulève des questions profondes :

1. **Vie Privée vs Sécurité** 🔐 vs 👁️
   - Où placer le curseur ?
   - La sécurité justifie-t-elle tout ?
   - Quid du consentement éclairé ?

2. **Confiance Brisée** 💔
   - HTTPS promettait du "bout en bout"
   - Le SSL Bumping brise cette promesse
   - Impact sur la confiance numérique globale

3. **Pente Glissante** 🎿
   - Commencer par la sécurité...
   - Finir par la surveillance ?
   - Qui surveille les surveillants ?

### Le Cadre Légal 📜

Le SSL Bumping n'est pas illégal en soi, MAIS :

- **Consentement** : Les utilisateurs doivent être informés
- **Proportionnalité** : L'usage doit être justifié
- **Protection des données** : RGPD et autres réglementations s'appliquent
- **Contexte** : Entreprise ≠ Café public ≠ Domicile

> [!WARNING] **Rappel Important : La Légalité Varie ! 🌍**
>
> Les lois diffèrent selon les pays et les contextes :
> - En entreprise : Généralement OK avec information
> - Dans le public : Très réglementé voire interdit
> - À la maison : Complexe (enfants, invités...)
>
> Consultez toujours un juriste avant déploiement !

## Conclusion : Le Cadenas Vert a-t-il Encore un Sens ? 🔒❓

Alors, après tout ça, que penser de notre petit cadenas vert ? Est-il devenu obsolète ? Pas du tout !

Le SSL Bumping nous rappelle que :

1. **La Sécurité est Relative** 🎭
   - Aucune technologie n'est infaillible
   - Le contexte est crucial
   - La vigilance reste de mise

2. **La Technologie est Neutre** ⚖️
   - SSL Bumping peut protéger ou espionner
   - L'intention fait la différence
   - L'éthique guide l'usage

3. **L'Éducation est Essentielle** 📚
   - Comprendre pour mieux se protéger
   - Démystifier pour mieux décider
   - Partager pour progresser ensemble

Le cadenas vert reste un excellent indicateur de sécurité. Mais comme toute technologie, il a ses limites. Le SSL Bumping nous montre qu'entre vous et le serveur, il peut y avoir des intermédiaires... parfois légitimes, parfois moins.

**Mon conseil ?** Restez curieux, restez vigilants, et n'hésitez pas à expérimenter (dans un lab, bien sûr !). La sécurité informatique est un domaine fascinant où l'attaque et la défense s'enrichissent mutuellement.

---

**Ressources pour aller plus loin :**
- 🧪 [Mon Lab SSL Bumping](https://github.com/darnodo/squid-ssl-bumping-lab)
- 📖 [Documentation Squid SSL Bump](http://www.squid-cache.org/Doc/config/ssl_bump/)
- 🔐 [OWASP sur l'Inspection TLS](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Security_Cheat_Sheet.html)

*Stay safe, stay curious!* 🚀🔒
