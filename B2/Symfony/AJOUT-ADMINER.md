# 🗄️ Ajouter Adminer au Template Officiel

Ce guide vous explique comment ajouter Adminer à votre installation Symfony Docker existante avec PostgreSQL.

## 🎯 Pourquoi ce Guide ?

Le template officiel `dunglas/symfony-docker` utilise **PostgreSQL** par défaut et n'inclut **pas** Adminer. Ce guide vous permet d'ajouter :
- ✅ **Adminer** (interface web pour gérer PostgreSQL)

## ⚠️ Prérequis

- Avoir suivi le guide d'installation principal [INSTALLATION.md](INSTALLATION.md)
- Votre projet Symfony Docker fonctionne déjà
- **Important :** Le template de base n'inclut **PAS** Doctrine ORM ni PostgreSQL par défaut - ces composants seront installés dans ce guide

## 🚀 Étapes d'Installation

### Étape 1 : Arrêter les Conteneurs

```powershell
# Aller dans votre projet
cd C:\Users\baill\Docker\Test_Installation\mon-projet-symfony

# Arrêter les conteneurs
docker compose down
```

### Étape 2 : Installer Doctrine ORM (OBLIGATOIRE)

**⚠️ CRITIQUE :** Cette étape est **OBLIGATOIRE** avant d'ajouter Adminer. Le template de base n'inclut pas Doctrine ORM ni le service de base de données.

**Solution : Installation via docker run**

Cette méthode contourne le problème de healthcheck en utilisant l'image Docker construite localement :

```powershell
# Utiliser l'image app-php pour installer Doctrine
docker run --rm -v ${PWD}:/app -w /app --entrypoint composer app-php require symfony/orm-pack
```

**⏱️ Temps d'installation :** 2-3 minutes (téléchargement des packages Symfony et Doctrine)

**✅ Cette installation ajoute automatiquement :**
- Le service `database` PostgreSQL dans `compose.yaml`
- La configuration Doctrine dans `config/packages/doctrine.yaml`
- Les variables d'environnement PostgreSQL

**🔍 Vérification :** Vous devriez maintenant voir une section `###> doctrine/doctrine-bundle ###` avec un service `database` à la fin de votre fichier `compose.yaml`.

### Étape 3 : Configuration Manuelle Requise

**⚠️ CORRECTIONS MANUELLES IMPORTANTES :**

#### 3.1 : Corriger le fichier .env (OBLIGATOIRE)

Ouvrez le fichier `.env` à la racine de votre projet et modifiez la ligne `DATABASE_URL` :

```bash
# AVANT (incorrect - causera des erreurs de connexion)
DATABASE_URL="postgresql://app:!ChangeMe!@127.0.0.1:5432/app?serverVersion=16&charset=utf8"

# APRÈS (correct - utilise le nom de service Docker)
DATABASE_URL="postgresql://app:!ChangeMe!@database:5432/app?serverVersion=16&charset=utf8"
```

**📝 Explication :** Docker utilise des noms de services (`database`) et non `localhost` ou `127.0.0.1`.

#### 3.2 : Installer l'Extension PostgreSQL PHP (OBLIGATOIRE)

```powershell
# Démarrer tous les conteneurs
docker compose up -d --wait

# Installer l'extension PostgreSQL requise
docker compose exec php install-php-extensions pdo_pgsql

# Arrêter les conteneurs pour la reconstruction
docker compose down
```

**📝 Pourquoi :** Le template de base n'inclut pas `pdo_pgsql` par défaut, causant des erreurs "could not find driver".

### Étape 4 : Ajouter Adminer au compose.override.yaml

Ajoutez seulement cette section à votre fichier `compose.override.yaml` (dans la section `services:` au même niveau que `php:`):

```yaml
  # Ajouter Adminer
  adminer:
    image: adminer
    restart: unless-stopped
    ports:
      - target: 8080
        published: 8080
        protocol: tcp
    depends_on:
      - database
```

### Étape 5 : Reconstruire et Redémarrer les Services

**⚠️ IMPORTANT :** Nous devons reconstruire l'image Docker pour inclure l'extension pdo_pgsql de manière permanente.

```powershell
# Reconstruire l'image Docker avec l'extension PostgreSQL
docker compose build --no-cache php

# Démarrer avec Adminer et PostgreSQL
docker compose up -d --wait
```

**Si la construction échoue ou si un conteneur n'arrive pas à démarrer :**

```powershell
# Option 1: Réinstaller l'extension (temporaire - sera perdue au prochain rebuild)
docker compose exec php install-php-extensions pdo_pgsql
docker compose restart php

# Option 2: Forcer une reconstruction complète
docker compose down --volumes --remove-orphans
docker compose build --no-cache
docker compose up -d --wait
```

**Vérifier l'intégrité des conteneurs**

```powershell
# Vérifier que tous les conteneurs sont sains
docker compose ps
```

**✅ Tous les services doivent afficher "healthy" ou "Up"**

## ✅ Test de Fonctionnement

### 1. Tester l'Application
- **URL :** http://localhost
- **Résultat attendu :** Page d'accueil Symfony

### 2. Tester Adminer avec PostgreSQL
- **URL :** http://localhost:8080
- **Système :** PostgreSQL
- **Serveur :** `database`
- **Utilisateur :** `app`
- **Mot de passe :** `!ChangeMe!`
- **Base de données :** `app`

**Résultat attendu :** Interface Adminer avec accès à PostgreSQL

## 🛑 En Cas de Problème

Si vous rencontrez des erreurs lors de l'installation, consultez le **[Guide de Dépannage](DEPANNAGE.md)** qui contient toutes les solutions aux problèmes courants :
- Extension PostgreSQL manquante (`could not find driver`)
- Erreurs de connexion (`Connection refused`)
- Conflits de ports
- Problèmes de volumes Docker

## 🎯 Étapes Suivantes

Maintenant qu'Adminer est installé :

1. **Créer vos entités** - Consultez [ENTITES.md](ENTITES.md)
2. **Manipuler les données** - Consultez [DONNEES-BDD.md](DONNEES-BDD.md)
3. **Utiliser Adminer** - Interface accessible sur http://localhost:8080

## 💡 Notes Importantes

- **PostgreSQL :** Ce guide utilise PostgreSQL (moderne et performant)
- **Persistence :** Les données PostgreSQL sont sauvegardées dans un volume Docker
- **Production :** Pour la production, modifiez `compose.prod.yaml` également
- **Sécurité :** Changez les mots de passe par défaut en production
- **Adminer universel :** Adminer supporte PostgreSQL, MySQL, SQLite et bien d'autres SGBD

---

**🔗 Guides Connexes :**
- [INSTALLATION.md](INSTALLATION.md) - Installation principale
- [ENTITES.md](ENTITES.md) - Création d'entités Symfony
- [DONNEES-BDD.md](DONNEES-BDD.md) - Manipulation des données via commandes
- [DEPANNAGE.md](DEPANNAGE.md) - Solutions aux problèmes courants