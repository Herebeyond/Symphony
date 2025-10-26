# 📦 Gestion des Packages et Dépendances Flutter

Ce guide explique comment gérer les packages (bibliothèques) dans votre projet Flutter via le fichier `pubspec.yaml`.

## 🎯 Qu'est-ce qu'un Package ?

Un **package** est une bibliothèque réutilisable de code Dart qui ajoute des fonctionnalités à votre application :
- 🌐 **Requêtes HTTP** : Communiquer avec des API (Symfony)
- 🎨 **UI Components** : Widgets personnalisés
- 💾 **Stockage local** : Sauvegarder des données
- 🔐 **Authentification** : Gérer les connexions utilisateurs
- 📊 **State Management** : Gérer l'état de l'application

**📚 Source des packages :** https://pub.dev

## 📄 Fichier pubspec.yaml

Le fichier `pubspec.yaml` à la racine de votre projet contient toutes les dépendances et configurations.

### Structure de Base

```yaml
name: mon_app_flutter
description: "Ma première application Flutter"
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: '>=3.2.0 <4.0.0'

dependencies:
  flutter:
    sdk: flutter
  
  # Vos packages de production ici

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^3.0.0
  
  # Vos packages de développement ici

flutter:
  uses-material-design: true
```

### Exemple Complet avec Packages Courants

```yaml
name: mon_app_flutter
description: "Application Flutter avec API Symfony"
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: '>=3.2.0 <4.0.0'

dependencies:
  flutter:
    sdk: flutter
  
  # Package pour les requêtes HTTP (connexion API Symfony)
  http: ^1.1.0
  
  # State management
  provider: ^6.1.1
  
  # Navigation avancée
  go_router: ^13.0.0
  
  # Stockage local (préférences)
  shared_preferences: ^2.2.2
  
  # Base de données locale
  sqflite: ^2.3.0
  
  # Gestion des formulaires
  flutter_form_builder: ^9.1.1
  
  # Loading indicators
  flutter_spinkit: ^5.2.0
  
  # Toast messages
  fluttertoast: ^8.2.4
  
  # Icons supplémentaires
  font_awesome_flutter: ^10.6.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^3.0.0
  
  # Tests
  mockito: ^5.4.4
  
  # Build runner (pour code generation)
  build_runner: ^2.4.7

flutter:
  uses-material-design: true
  
  # Assets (images, fonts, etc.)
  # assets:
  #   - assets/images/
  #   - assets/icons/
  
  # Fonts personnalisées
  # fonts:
  #   - family: CustomFont
  #     fonts:
  #       - asset: fonts/CustomFont-Regular.ttf
  #       - asset: fonts/CustomFont-Bold.ttf
  #         weight: 700
```

## 🛠️ Commandes de Gestion des Packages

### Installer les Dépendances

```powershell
# Installer toutes les dépendances du pubspec.yaml
flutter pub get

# Équivalent (raccourci)
flutter pub get
```

**📝 Quand utiliser :**
- Après avoir cloné un projet
- Après avoir modifié `pubspec.yaml` manuellement
- Si les dépendances sont manquantes

### Ajouter un Package

```powershell
# Ajouter un package en dependencies
flutter pub add http
flutter pub add provider
flutter pub add go_router

# Ajouter un package en dev_dependencies
flutter pub add --dev flutter_lints
flutter pub add --dev mockito

# Ajouter une version spécifique
flutter pub add http:1.1.0
```

**✅ Avantage :** Cette commande :
1. Ajoute le package dans `pubspec.yaml`
2. Télécharge le package automatiquement
3. Met à jour `pubspec.lock`

### Supprimer un Package

```powershell
# Supprimer un package
flutter pub remove http
flutter pub remove provider

# Nettoyer les packages inutilisés
flutter pub get
```

### Mettre à Jour les Packages

```powershell
# Vérifier les packages obsolètes
flutter pub outdated

# Mettre à jour tous les packages (dans les contraintes)
flutter pub upgrade

# Mettre à jour un package spécifique
flutter pub upgrade http

# Mettre à jour vers la dernière version compatible
flutter pub upgrade --major-versions
```

### Analyser les Dépendances

```powershell
# Afficher l'arbre de dépendances
flutter pub deps

# Afficher uniquement les dépendances directes
flutter pub deps --no-dev

# Vérifier les dépendances
flutter pub get --dry-run
```

## 📦 Packages Essentiels pour Connexion avec Symfony

### 1. HTTP - Requêtes API

```yaml
dependencies:
  http: ^1.1.0
```

```dart
import 'package:http/http.dart' as http;

// GET request
final response = await http.get(Uri.parse('https://localhost/api/users'));

// POST request
final response = await http.post(
  Uri.parse('https://localhost/api/users'),
  headers: {'Content-Type': 'application/json'},
  body: json.encode({'nom': 'Doe', 'prenom': 'John'}),
);
```

### 2. Dio - Alternative HTTP avancée

```yaml
dependencies:
  dio: ^5.4.0
```

```dart
import 'package:dio/dio.dart';

final dio = Dio();

// GET request avec intercepteurs
final response = await dio.get('https://localhost/api/users');

// POST request
final response = await dio.post(
  'https://localhost/api/users',
  data: {'nom': 'Doe', 'prenom': 'John'},
);
```

**💡 Avantages de Dio :**
- Intercepteurs (pour JWT, logs, etc.)
- Gestion automatique des timeouts
- Upload/Download de fichiers
- Meilleure gestion des erreurs

### 3. Provider - State Management

```yaml
dependencies:
  provider: ^6.1.1
```

```dart
import 'package:provider/provider.dart';

// Utilisation
class UserProvider extends ChangeNotifier {
  List<User> _users = [];
  
  List<User> get users => _users;
  
  Future<void> loadUsers() async {
    _users = await ApiService.getUsers();
    notifyListeners();
  }
}

// Dans l'app
MultiProvider(
  providers: [
    ChangeNotifierProvider(create: (_) => UserProvider()),
  ],
  child: MyApp(),
);
```

### 4. Shared Preferences - Stockage Local

```yaml
dependencies:
  shared_preferences: ^2.2.2
```

```dart
import 'package:shared_preferences/shared_preferences.dart';

// Sauvegarder
final prefs = await SharedPreferences.getInstance();
await prefs.setString('token', 'eyJhbGciOiJIUzI1...');

// Lire
final token = prefs.getString('token');
```

### 5. Flutter Secure Storage - Stockage Sécurisé

```yaml
dependencies:
  flutter_secure_storage: ^9.0.0
```

```dart
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

final storage = FlutterSecureStorage();

// Sauvegarder (crypté)
await storage.write(key: 'jwt_token', value: token);

// Lire
final token = await storage.read(key: 'jwt_token');

// Supprimer
await storage.delete(key: 'jwt_token');
```

## 🎨 Packages UI Populaires

### 1. Flutter Spinkit - Loading Indicators

```yaml
dependencies:
  flutter_spinkit: ^5.2.0
```

```dart
import 'package:flutter_spinkit/flutter_spinkit.dart';

SpinKitRotatingCircle(
  color: Colors.blue,
  size: 50.0,
)
```

### 2. Cached Network Image - Images avec Cache

```yaml
dependencies:
  cached_network_image: ^3.3.0
```

```dart
import 'package:cached_network_image/cached_network_image.dart';

CachedNetworkImage(
  imageUrl: "https://example.com/image.jpg",
  placeholder: (context, url) => CircularProgressIndicator(),
  errorWidget: (context, url, error) => Icon(Icons.error),
)
```

### 3. Font Awesome - Icônes Supplémentaires

```yaml
dependencies:
  font_awesome_flutter: ^10.6.0
```

```dart
import 'package:font_awesome_flutter/font_awesome_flutter.dart';

FaIcon(FontAwesomeIcons.github)
FaIcon(FontAwesomeIcons.user)
```

## 🔧 Packages de Développement

### 1. Flutter Lints - Règles de Qualité de Code

```yaml
dev_dependencies:
  flutter_lints: ^3.0.0
```

Crée un fichier `analysis_options.yaml` :

```yaml
include: package:flutter_lints/flutter.yaml

linter:
  rules:
    prefer_const_constructors: true
    avoid_print: true
```

### 2. Build Runner - Génération de Code

```yaml
dev_dependencies:
  build_runner: ^2.4.7
  json_serializable: ^6.7.1
```

```powershell
# Générer le code
flutter pub run build_runner build

# Watch mode (regénère automatiquement)
flutter pub run build_runner watch
```

## ⚠️ Bonnes Pratiques

### 1. Versionning des Packages

```yaml
# ✅ RECOMMANDÉ - Version compatible (^)
dependencies:
  http: ^1.1.0  # Accepte 1.1.x et 1.2.x mais pas 2.0.0

# ⚠️ Version exacte (éviter sauf cas spécifique)
dependencies:
  http: 1.1.0  # Uniquement cette version

# ❌ DÉCONSEILLÉ - N'importe quelle version
dependencies:
  http: any  # Peut causer des conflits
```

### 2. Séparation dependencies vs dev_dependencies

```yaml
dependencies:
  # Packages utilisés dans l'app en production
  http: ^1.1.0
  provider: ^6.1.1

dev_dependencies:
  # Packages utilisés uniquement pour le développement/tests
  flutter_test:
    sdk: flutter
  mockito: ^5.4.4
  flutter_lints: ^3.0.0
```

### 3. Nettoyer Régulièrement

```powershell
# Nettoyer et réinstaller
flutter clean
flutter pub get

# Vérifier les packages obsolètes
flutter pub outdated
```

### 4. Fichier pubspec.lock

- ✅ **Commiter** ce fichier dans Git
- ✅ Garantit les mêmes versions pour tous les développeurs
- ✅ Évite les surprises lors des déploiements

## 🔗 Ressources

- **Pub.dev** : https://pub.dev (recherche de packages)
- **Flutter Packages** : https://docs.flutter.dev/packages-and-plugins
- **Awesome Flutter** : https://github.com/Solido/awesome-flutter

## 📚 Fichiers de Documentation Associés

- **[INSTALLATION.md](INSTALLATION.md)** - Installation de Flutter
- **[CONNEXION-API-SYMFONY.md](CONNEXION-API-SYMFONY.md)** - Intégration avec l'API Symfony
- **[COMMANDES-UTILES.md](COMMANDES-UTILES.md)** - Commandes Flutter courantes
- **[DEPANNAGE.md](DEPANNAGE.md)** - Problèmes courants et solutions
