# Installation et Configuration de ngrok pour Symfony

## 📋 Table des matières
1. [Introduction](#introduction)
2. [Prérequis](#prérequis)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Utilisation](#utilisation)
6. [Fonctionnement](#fonctionnement)
7. [Dépannage](#dépannage)

---

## 🎯 Introduction

**ngrok** est un outil qui permet de créer un tunnel sécurisé entre Internet et votre application locale. Il expose votre API Symfony qui tourne en local (sur Docker) au monde entier via une URL publique HTTPS.

### Cas d'usage
- Tester votre API avec des webhooks externes (Stripe, PayPal, etc.)
- Partager votre API en développement avec des collègues ou clients
- Tester votre API sur des appareils mobiles
- Développer et tester des intégrations avec des services tiers

---

## ✅ Prérequis

- Docker et Docker Compose installés
- Un projet Symfony fonctionnel avec Docker (déjà configuré ✓)
- Un compte ngrok (gratuit) : [https://ngrok.com/](https://ngrok.com/)

---

## 🚀 Installation

### Étape 1 : Créer un compte ngrok

1. Rendez-vous sur [https://ngrok.com/signup](https://ngrok.com/signup)
2. Créez un compte gratuit (avec Google, GitHub ou email)
3. Une fois connecté, allez dans le dashboard
4. Copiez votre **authtoken** depuis [https://dashboard.ngrok.com/get-started/your-authtoken](https://dashboard.ngrok.com/get-started/your-authtoken)

### Étape 2 : Modification du docker-compose.yaml

Le service ngrok a été ajouté au fichier `compose.yaml` :

```yaml
###> ngrok ###
  ngrok:
    image: ngrok/ngrok:latest
    restart: unless-stopped
    command:
      - "start"
      - "--all"
      - "--config"
      - "/etc/ngrok.yml"
    volumes:
      - ./ngrok.yml:/etc/ngrok.yml
    ports:
      - 4040:4040
    depends_on:
      - php
###< ngrok ###
```

**Explications** :
- `image: ngrok/ngrok:latest` : Utilise l'image officielle Docker de ngrok
- `restart: unless-stopped` : Redémarre automatiquement le conteneur si il s'arrête
- `command` : Démarre tous les tunnels définis dans la configuration
- `volumes` : Monte le fichier de configuration ngrok dans le conteneur
- `ports: 4040:4040` : Expose l'interface web de ngrok sur le port 4040
- `depends_on: php` : Assure que le service PHP démarre avant ngrok

### Étape 3 : Configuration de ngrok.yml

Un fichier `ngrok.yml` a été créé à la racine du projet :

```yaml
version: "2"
authtoken: YOUR_NGROK_AUTHTOKEN
tunnels:
  symfony-api:
    addr: php:80
    proto: http
    inspect: true
```

**Explications des paramètres** :
- `version: "2"` : Version du format de configuration ngrok
- `authtoken` : Votre token d'authentification ngrok (à remplacer)
- `tunnels` : Liste des tunnels à créer
  - `symfony-api` : Nom du tunnel (personnalisable)
  - `addr: php:80` : Adresse du service à exposer (le conteneur PHP sur le port 80)
  - `proto: http` : Protocole à utiliser (HTTP)
  - `inspect: true` : Active l'inspection du trafic dans l'interface web

---

## ⚙️ Configuration

### Étape 1 : Ajouter votre authtoken

Ouvrez le fichier `ngrok.yml` et remplacez `YOUR_NGROK_AUTHTOKEN` par votre véritable token :

```yaml
authtoken: votre_token_ici
```

### Étape 2 : Redémarrer les conteneurs

```powershell
# Arrêter les conteneurs
docker compose down

# Reconstruire et démarrer avec ngrok
docker compose up -d
```

---

## 🎮 Utilisation

### Démarrer le tunnel

Si ce n'est pas déjà fait :

```powershell
cd c:\Users\baill\Docker\Test_Installation\mon-projet-symfony
docker compose up -d
```

### Vérifier que ngrok fonctionne

```powershell
docker compose ps
```

Vous devriez voir le conteneur `ngrok` en état "Up".

### Accéder à l'interface web de ngrok

Ouvrez votre navigateur et allez sur : **http://localhost:4040**

Cette interface vous permet de :
- ✅ Voir l'URL publique générée (ex: `https://xxxx-xx-xx-xx.ngrok-free.app`)
- 🔍 Inspecter toutes les requêtes HTTP en temps réel
- 📊 Analyser les headers, body, réponses
- 🔄 Rejouer des requêtes

### Obtenir l'URL publique via la ligne de commande

```powershell
docker compose logs ngrok
```

Cherchez une ligne comme :
```
started tunnel  url=https://xxxx-xx-xx-xx.ngrok-free.app
```

### Tester votre API

Une fois l'URL obtenue, vous pouvez tester votre API :

```powershell
# Avec curl (si installé)
curl https://votre-url-ngrok.ngrok-free.app/api/users

# Ou dans votre navigateur
https://votre-url-ngrok.ngrok-free.app
```

### Arrêter ngrok

```powershell
# Arrêter uniquement ngrok
docker compose stop ngrok

# Ou arrêter tous les services
docker compose down
```

---

## 🔧 Fonctionnement

### Architecture du tunnel

```
Internet
    ↓
[Serveurs ngrok]
    ↓ (tunnel sécurisé)
[Conteneur ngrok]
    ↓ (réseau Docker)
[Conteneur PHP/Symfony]
    ↓
[Votre API]
```

### Processus détaillé

1. **Connexion** : Le conteneur ngrok se connecte aux serveurs ngrok avec votre authtoken
2. **Tunnel** : Un tunnel TCP/IP sécurisé est établi entre votre machine et les serveurs ngrok
3. **URL publique** : ngrok génère une URL publique aléatoire (ou personnalisée si compte payant)
4. **Routage** : Toute requête vers l'URL publique est routée via le tunnel vers votre conteneur PHP
5. **Réponse** : La réponse de votre API repart dans l'autre sens jusqu'au client

### Sécurité

- ✅ Tunnel chiffré en HTTPS automatiquement
- ✅ Authentification via authtoken
- ⚠️ L'URL est publique : n'importe qui peut y accéder
- ⚠️ Ne pas exposer de données sensibles en développement
- 💡 Possibilité d'ajouter une authentification basique (compte payant)

### Limitations du compte gratuit

- 🔄 URL aléatoire qui change à chaque redémarrage
- ⏱️ Session limitée à 2 heures
- 🔢 1 processus ngrok simultané
- 📊 40 connexions/minute

---

## 🐛 Dépannage

### Configuration Caddyfile pour ngrok

**Important** : Pour que ngrok fonctionne correctement avec FrankenPHP/Caddy, vous devez modifier le fichier `frankenphp/Caddyfile` :

```plaintext
{
    skip_install_trust
    auto_https off  # Désactive la redirection HTTPS automatique
    
    servers {
        trusted_proxies static 0.0.0.0/0 ::/0  # Fait confiance aux proxies
    }
    
    # ... reste de la configuration
}
```

Ces directives sont **essentielles** pour éviter les boucles de redirection.

### Problème : Le conteneur ngrok ne démarre pas

**Solution** : Vérifiez que vous avez bien remplacé `YOUR_NGROK_AUTHTOKEN` dans `ngrok.yml`

```powershell
docker compose logs ngrok
```

### Problème : "ERR_NGROK_108"

**Cause** : Authtoken invalide ou expiré

**Solution** : 
1. Récupérez un nouveau token sur https://dashboard.ngrok.com/get-started/your-authtoken
2. Mettez à jour `ngrok.yml`
3. Redémarrez : `docker compose restart ngrok`

### Problème : ERR_TOO_MANY_REDIRECTS - Boucle de redirection

**Erreur** : "Cette page vous a redirigé à de trop nombreuses reprises"

**Cause** : Caddy force la redirection HTTPS même si ngrok envoie déjà du HTTPS

**Solution** : Ajoutez `auto_https off` dans le bloc global du Caddyfile

```plaintext
{
    skip_install_trust
    auto_https off
    
    servers {
        trusted_proxies static 0.0.0.0/0 ::/0
    }
}
```

Puis redémarrez : `docker compose restart php`

**Explication** : ngrok gère déjà le HTTPS en externe, donc Caddy n'a pas besoin de forcer la redirection.

### Problème : ERR_NGROK_8012 - Connection refused

**Erreur** : "dial tcp 172.22.0.3:80: connect: connection refused"

**Cause** : Le conteneur PHP ne démarre pas correctement, souvent à cause d'une erreur dans le Caddyfile

**Solution** :
1. Vérifiez les logs PHP : `docker compose logs php`
2. Cherchez les erreurs de configuration Caddy
3. Si vous voyez "unrecognized directive: servers", la directive `trusted_proxies` doit être dans le bloc global `{}` du Caddyfile
4. Redémarrez : `docker compose restart php`

### Problème : L'URL ngrok ne répond pas

**Vérifications** :
1. Le service PHP est bien démarré : `docker compose ps`
2. L'application Symfony répond localement : http://localhost
3. Regardez les logs : `docker compose logs ngrok`

### Problème : Erreur "tunnel session failed"

**Solution** : Vous avez peut-être déjà une session ngrok active ailleurs

```powershell
# Arrêter tous les conteneurs ngrok
docker compose down
docker compose up -d
```

### Problème : "This site can't be reached"

**Vérifiez** :
1. Que l'URL est correcte (copiez depuis http://localhost:4040)
2. Que votre firewall ne bloque pas ngrok
3. Que vous êtes connecté à Internet

### Voir les logs en temps réel

```powershell
docker compose logs -f ngrok
```

---

## 📝 Commandes utiles

```powershell
# Démarrer tous les services
docker compose up -d

# Voir le statut
docker compose ps

# Voir les logs de ngrok
docker compose logs ngrok

# Voir les logs en temps réel
docker compose logs -f ngrok

# Redémarrer uniquement ngrok
docker compose restart ngrok

# Arrêter ngrok
docker compose stop ngrok

# Supprimer le conteneur ngrok
docker compose rm -f ngrok

# Tout arrêter
docker compose down
```

---

## 🎓 Configuration avancée

### Utiliser HTTPS vers le service PHP

Si votre conteneur PHP utilise HTTPS (port 443) :

```yaml
tunnels:
  symfony-api:
    addr: https://php:443
    proto: http
    inspect: true
```

### Utiliser un sous-domaine personnalisé (compte payant)

```yaml
tunnels:
  symfony-api:
    addr: php:80
    proto: http
    subdomain: mon-api-symfony
    inspect: true
```

Votre URL sera : `https://mon-api-symfony.ngrok-free.app`

### Ajouter une authentification basique (compte payant)

```yaml
tunnels:
  symfony-api:
    addr: php:80
    proto: http
    auth: "utilisateur:motdepasse"
    inspect: true
```

### Désactiver l'avertissement ngrok

Le compte gratuit affiche un avertissement avant d'accéder à l'API. Pour le retirer, il faut passer au compte payant.

---

## 🔗 Ressources

- [Documentation officielle ngrok](https://ngrok.com/docs)
- [ngrok Dashboard](https://dashboard.ngrok.com/)
- [Plans et tarifs ngrok](https://ngrok.com/pricing)
- [Configuration ngrok en YAML](https://ngrok.com/docs/agent/config/)

---

## ✅ Checklist de vérification

- [ ] Compte ngrok créé
- [ ] Authtoken récupéré et ajouté dans `ngrok.yml`
- [ ] Service ngrok ajouté dans `compose.yaml`
- [ ] Conteneurs redémarrés : `docker compose up -d`
- [ ] Interface ngrok accessible : http://localhost:4040
- [ ] URL publique obtenue
- [ ] API testée et fonctionnelle

---

**🎉 Félicitations ! Votre API Symfony est maintenant accessible depuis Internet via ngrok !**
