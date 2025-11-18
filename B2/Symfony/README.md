# 🎼 Documentation Symfony

Bienvenue dans la documentation Symfony ! Ce dossier contient tous les guides nécessaires pour développer des applications backend Symfony avec Docker et API Platform.

## 📚 Liste des Documents

### 🚀 [INSTALLATION.md](INSTALLATION.md)
Guide complet d'installation de Symfony avec Docker.
- Installation de Docker Desktop
- Configuration du projet Symfony
- Démarrage des conteneurs
- Accès aux services (PHP, PostgreSQL, Nginx)
- Structure d'un projet Symfony

### 🏗️ [ENTITES.md](ENTITES.md)
Création et gestion des entités Doctrine.
- Comprendre les entités et ORM
- Créer des entités avec Maker
- Relations entre entités (OneToMany, ManyToOne, etc.)
- Migrations de base de données
- Fixtures et données de test

### 🗄️ [DONNEES-BDD.md](DONNEES-BDD.md)
Gestion des données et de la base de données.
- Configuration de PostgreSQL
- Commandes Doctrine
- Importer/Exporter des données
- Requêtes personnalisées
- Bonnes pratiques SQL

### 🌐 [API-PLATFORM.md](API-PLATFORM.md)
Création d'API REST avec API Platform.
- Installation et configuration
- Exposer des entités en API
- Personnaliser les endpoints
- Filtres et pagination
- Sérialisation et groupes
- Documentation OpenAPI automatique

### 🔧 [AJOUT-ADMINER.md](AJOUT-ADMINER.md)
Interface web pour gérer la base de données.
- Accès à Adminer
- Explorer et modifier les tables
- Exécuter des requêtes SQL
- Import/Export de données

### 📊 [DIAGRAMMES.md](DIAGRAMMES.md)
Visualisation de l'architecture et des relations.
- Diagrammes UML
- Schémas de base de données
- Architecture du projet
- Flux de données

### 🛠️ [NODEJS.md](NODEJS.md)
Intégration de Node.js dans Symfony.
- Installation et configuration
- Webpack Encore
- Assets JavaScript/CSS
- Compilation frontend

### ⚠️ [DEPANNAGE.md](DEPANNAGE.md)
Solutions aux problèmes courants.
- Problèmes Docker
- Erreurs de conteneurs
- Problèmes de base de données
- Erreurs Symfony
- Problèmes de cache

### 🌍 [NGROK.md](NGROK.md)
Exposer votre API Symfony sur Internet avec ngrok.
- Installation et configuration de ngrok
- Créer un tunnel sécurisé
- Obtenir une URL publique
- Tester l'API depuis n'importe où
- Partager l'API avec des clients ou webhooks

## 🎯 Par où commencer ?

### 1️⃣ Débutant complet
Commencez par **[INSTALLATION.md](INSTALLATION.md)** pour installer Docker et créer votre projet Symfony.

### 2️⃣ Symfony installé, créer des entités
Consultez **[ENTITES.md](ENTITES.md)** pour créer vos modèles de données et **[DONNEES-BDD.md](DONNEES-BDD.md)** pour gérer la base.

### 3️⃣ Créer une API REST
Suivez **[API-PLATFORM.md](API-PLATFORM.md)** pour exposer vos entités en API et les connecter avec Flutter.

### 4️⃣ Gérer la base de données
Utilisez **[AJOUT-ADMINER.md](AJOUT-ADMINER.md)** pour accéder à l'interface web de gestion de base de données.

### 5️⃣ Problème rencontré
Référez-vous à **[DEPANNAGE.md](DEPANNAGE.md)** pour trouver des solutions aux erreurs courantes.

### 6️⃣ Visualiser l'architecture
Consultez **[DIAGRAMMES.md](DIAGRAMMES.md)** pour comprendre la structure du projet.

### 7️⃣ Exposer l'API sur Internet
Suivez **[NGROK.md](NGROK.md)** pour rendre votre API accessible depuis n'importe où avec ngrok.

## 🔗 Lien avec Flutter

Symfony et Flutter fonctionnent ensemble dans une architecture **client-serveur** :

```
┌─────────────────────────┐
│   Flutter Application   │  ← Frontend (Mobile/Web/Desktop)
│  (Interface Utilisateur)│
└───────────┬─────────────┘
            │
            │ HTTP/HTTPS (REST API)
            │ Format: JSON
            ↓
┌─────────────────────────┐
│    Symfony Backend      │  ← Backend (Serveur)
│    (API Platform)       │
└───────────┬─────────────┘
            │
            ↓
┌─────────────────────────┐
│   Base de Données       │  ← PostgreSQL/MySQL
│   (PostgreSQL)          │
└─────────────────────────┘
```

**Symfony** expose des API REST via API Platform tandis que **Flutter** consomme ces API pour afficher les données.

## 📖 Documentation Flutter

Pour la partie frontend Flutter, consultez le dossier **[B2/Flutter](../Flutter/)** :
- Installation Flutter
- Connexion à l'API Symfony
- Gestion des packages
- Commandes utiles

## 🐳 Architecture Docker

Le projet utilise Docker avec plusieurs conteneurs :

- **PHP-FPM** : Exécution de Symfony
- **Nginx** : Serveur web
- **PostgreSQL** : Base de données
- **Adminer** : Interface web pour la base de données
- **Node.js** : Compilation des assets (optionnel)

## 🌟 Ressources Officielles

- **Site officiel** : https://symfony.com
- **Documentation** : https://symfony.com/doc/current/
- **API Platform** : https://api-platform.com
- **Doctrine ORM** : https://www.doctrine-project.org
- **Communauté** : https://symfony.com/community

## 💡 Conseils

- ✅ Lisez les documents dans l'ordre recommandé
- ✅ Testez chaque commande dans le conteneur Docker
- ✅ Consultez [DEPANNAGE.md](DEPANNAGE.md) en cas d'erreur
- ✅ Utilisez `docker-compose ps` pour vérifier l'état des conteneurs
- ✅ Gardez Symfony à jour avec `composer update`
- ✅ Activez le mode debug pendant le développement
- ✅ Utilisez le Profiler Symfony pour déboguer

## 🆘 Besoin d'Aide ?

1. Consultez **[DEPANNAGE.md](DEPANNAGE.md)**
2. Vérifiez les logs Docker avec `docker-compose logs`
3. Utilisez le Profiler Symfony en mode debug
4. Cherchez sur Stack Overflow (tag `symfony`)
5. Rejoignez la communauté Symfony

Bonne chance avec Symfony ! 🎼
