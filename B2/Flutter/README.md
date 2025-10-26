# 📱 Documentation Flutter

Bienvenue dans la documentation Flutter ! Ce dossier contient tous les guides nécessaires pour développer des applications Flutter et les connecter avec Symfony.

## 📚 Liste des Documents

### 🚀 [INSTALLATION.md](INSTALLATION.md)
Guide complet d'installation de Flutter sur Windows.
- Installation du SDK Flutter
- Configuration des variables d'environnement
- Configuration de VS Code
- Création de votre première application
- Structure d'un projet Flutter

### 📦 [PACKAGES-DEPENDANCES.md](PACKAGES-DEPENDANCES.md)
Gestion des packages et dépendances via `pubspec.yaml`.
- Comprendre le fichier pubspec.yaml
- Installer et gérer les packages
- Packages essentiels pour Symfony
- Bonnes pratiques de versionning

### 🌐 [CONNEXION-API-SYMFONY.md](CONNEXION-API-SYMFONY.md)
Intégration Flutter avec l'API Symfony.
- Architecture client-serveur
- Service HTTP pour communiquer avec Symfony
- Modèles de données
- Configuration CORS
- Authentification JWT
- Gestion des erreurs HTTP

### 🛠️ [COMMANDES-UTILES.md](COMMANDES-UTILES.md)
Référence complète des commandes Flutter.
- Gestion de projet
- Exécution et debug
- Build et déploiement
- Tests
- Configuration Flutter

### ⚠️ [DEPANNAGE.md](DEPANNAGE.md)
Solutions aux problèmes courants.
- Problèmes d'installation
- Problèmes de dépendances
- Problèmes de connexion API
- Problèmes de build
- Problèmes d'exécution

## 🎯 Par où commencer ?

### 1️⃣ Débutant complet
Commencez par **[INSTALLATION.md](INSTALLATION.md)** pour installer Flutter et créer votre première application.

### 2️⃣ Flutter installé, besoin de packages
Consultez **[PACKAGES-DEPENDANCES.md](PACKAGES-DEPENDANCES.md)** pour ajouter des packages à votre projet.

### 3️⃣ Connecter avec Symfony
Suivez **[CONNEXION-API-SYMFONY.md](CONNEXION-API-SYMFONY.md)** pour créer un service API et communiquer avec votre backend Symfony.

### 4️⃣ Problème rencontré
Référez-vous à **[DEPANNAGE.md](DEPANNAGE.md)** pour trouver des solutions aux erreurs courantes.

### 5️⃣ Besoin d'une commande
Cherchez dans **[COMMANDES-UTILES.md](COMMANDES-UTILES.md)** la commande dont vous avez besoin.

## 🔗 Lien avec Symfony

Flutter et Symfony fonctionnent ensemble dans une architecture **client-serveur** :

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

**Flutter** gère l'interface utilisateur tandis que **Symfony** expose des API REST pour la logique métier et l'accès aux données.

## 📖 Documentation Symfony

Pour la partie backend Symfony, consultez le dossier **[B2/Symfony](../Symfony/)** :
- Installation Symfony Docker
- API Platform
- Entités et base de données
- Configuration Adminer

## 🌟 Ressources Officielles

- **Site officiel** : https://flutter.dev
- **Documentation** : https://docs.flutter.dev
- **Packages** : https://pub.dev
- **Communauté** : https://discord.gg/flutter

## 💡 Conseils

- ✅ Lisez les documents dans l'ordre recommandé
- ✅ Testez chaque exemple de code
- ✅ Consultez [DEPANNAGE.md](DEPANNAGE.md) en cas d'erreur
- ✅ Utilisez `flutter doctor` régulièrement pour vérifier votre installation
- ✅ Gardez Flutter à jour avec `flutter upgrade`

## 🆘 Besoin d'Aide ?

1. Consultez **[DEPANNAGE.md](DEPANNAGE.md)**
2. Utilisez `flutter doctor -v` pour diagnostiquer
3. Cherchez sur Stack Overflow (tag `flutter`)
4. Rejoignez le Discord Flutter

Bonne chance avec Flutter ! 🚀
