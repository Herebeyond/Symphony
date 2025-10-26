# 🛠️ Commandes Utiles Flutter

Ce guide rassemble toutes les commandes Flutter couramment utilisées pour le développement, le build et le déploiement.

## 📦 Gestion de Projet

### Créer un Nouveau Projet

```powershell
# Créer un nouveau projet Flutter
flutter create nom_projet

# Créer avec une organisation spécifique
flutter create --org com.monentreprise nom_projet

# Créer avec un template spécifique
flutter create --template=plugin mon_plugin
flutter create --template=package mon_package

# Créer seulement pour certaines plateformes
flutter create --platforms=android,ios,web mon_app
```

### Analyser le Code

```powershell
# Analyser tout le code (lint)
flutter analyze

# Analyser avec détails
flutter analyze --verbose

# Analyser un fichier spécifique
flutter analyze lib/main.dart
```

### Formater le Code

```powershell
# Formater tout le code
flutter format .

# Formater un dossier spécifique
flutter format lib/

# Formater avec ligne de 120 caractères
flutter format --line-length 120 .

# Vérifier sans modifier
flutter format --set-exit-if-changed .
```

### Nettoyer le Projet

```powershell
# Nettoyer les fichiers de build
flutter clean

# Nettoyer et réinstaller les dépendances
flutter clean; flutter pub get
```

## 🏃 Exécution et Debug

### Lancer l'Application

```powershell
# Lister les périphériques disponibles
flutter devices

# Lancer sur le périphérique par défaut
flutter run

# Lancer sur un périphérique spécifique
flutter run -d chrome
flutter run -d windows
flutter run -d emulator-5554

# Lancer en mode release
flutter run --release

# Lancer en mode profile (pour analyser les performances)
flutter run --profile

# Lancer avec hot reload désactivé
flutter run --no-hot-reload
```

### Options de Démarrage

```powershell
# Lancer avec un port d'observation spécifique
flutter run --observatory-port=8888

# Lancer en mode verbose
flutter run --verbose

# Lancer avec une cible différente
flutter run -t lib/main_dev.dart

# Lancer avec arguments
flutter run --dart-define=ENV=dev
```

### Commandes pendant l'Exécution

Pendant que l'app est en cours d'exécution :
- `r` - Hot reload (recharger le code)
- `R` - Hot restart (redémarrer complètement)
- `p` - Afficher la grille de pixels
- `o` - Basculer entre Android et iOS
- `w` - Dump widget hierarchy
- `t` - Dump rendering tree
- `q` - Quitter

### Logs et Debugging

```powershell
# Afficher les logs en temps réel
flutter logs

# Afficher les logs d'un périphérique spécifique
flutter logs -d chrome

# Lancer Flutter DevTools
flutter pub global activate devtools
flutter pub global run devtools
```

## 📦 Gestion des Dépendances

### Installer les Packages

```powershell
# Installer toutes les dépendances
flutter pub get

# Ajouter un package
flutter pub add http
flutter pub add provider

# Ajouter un package en dev_dependencies
flutter pub add --dev flutter_lints

# Supprimer un package
flutter pub remove http
```

### Mettre à Jour les Packages

```powershell
# Vérifier les packages obsolètes
flutter pub outdated

# Mettre à jour tous les packages
flutter pub upgrade

# Mettre à jour un package spécifique
flutter pub upgrade http

# Mettre à jour vers les versions majeures
flutter pub upgrade --major-versions
```

### Analyser les Dépendances

```powershell
# Afficher l'arbre de dépendances
flutter pub deps

# Afficher seulement les dépendances directes
flutter pub deps --no-dev

# Vérifier les dépendances (dry run)
flutter pub get --dry-run
```

## 🏗️ Build et Déploiement

### Build Android

```powershell
# Build APK (debug)
flutter build apk

# Build APK (release)
flutter build apk --release

# Build APK pour une ABI spécifique
flutter build apk --target-platform android-arm64

# Build App Bundle (Google Play)
flutter build appbundle --release

# Build avec split per ABI
flutter build apk --split-per-abi
```

### Build iOS

```powershell
# Build pour iOS
flutter build ios --release

# Build sans code signing
flutter build ios --no-codesign

# Build pour simulateur
flutter build ios --simulator
```

### Build Web

```powershell
# Build pour Web
flutter build web --release

# Build avec renderer HTML
flutter build web --web-renderer html

# Build avec renderer CanvasKit
flutter build web --web-renderer canvaskit

# Build avec base-href spécifique
flutter build web --base-href /mon-app/
```

### Build Windows

```powershell
# Build pour Windows
flutter build windows --release

# Build en mode debug
flutter build windows
```

### Build Linux

```powershell
# Build pour Linux
flutter build linux --release
```

### Build macOS

```powershell
# Build pour macOS
flutter build macos --release
```

## 🧪 Tests

### Lancer les Tests

```powershell
# Lancer tous les tests
flutter test

# Lancer avec coverage
flutter test --coverage

# Lancer un fichier de test spécifique
flutter test test/widget_test.dart

# Lancer avec un nom spécifique
flutter test --name="UserScreen"

# Lancer en mode verbose
flutter test --verbose
```

### Tests d'Intégration

```powershell
# Lancer les tests d'intégration
flutter test integration_test

# Lancer sur un périphérique spécifique
flutter test integration_test/app_test.dart -d chrome
```

### Générer le Rapport de Coverage

```powershell
# Générer coverage
flutter test --coverage

# Afficher le rapport (nécessite lcov)
genhtml coverage/lcov.info -o coverage/html
```

## 🔧 Configuration Flutter

### Configuration Générale

```powershell
# Afficher la configuration Flutter
flutter config

# Activer le support Web
flutter config --enable-web

# Activer le support Windows Desktop
flutter config --enable-windows-desktop

# Activer le support Linux Desktop
flutter config --enable-linux-desktop

# Activer le support macOS Desktop
flutter config --enable-macos-desktop

# Désactiver analytics
flutter config --no-analytics
```

### Mettre à Jour Flutter

```powershell
# Vérifier les mises à jour
flutter upgrade --dry-run

# Mettre à jour Flutter
flutter upgrade

# Mettre à jour vers une version spécifique
flutter upgrade --force

# Passer au canal stable
flutter channel stable
flutter upgrade

# Passer au canal beta
flutter channel beta
flutter upgrade
```

### Informations Flutter

```powershell
# Afficher la version de Flutter
flutter --version

# Diagnostic complet
flutter doctor

# Diagnostic détaillé
flutter doctor -v

# Accepter les licences Android
flutter doctor --android-licenses
```

## 🎨 Génération de Code

### Assets et Icons

```powershell
# Générer les icônes de l'app
flutter pub run flutter_launcher_icons

# Générer les splash screens
flutter pub run flutter_native_splash:create
```

### Code Generation (Build Runner)

```powershell
# Générer le code une fois
flutter pub run build_runner build

# Générer en supprimant les conflits
flutter pub run build_runner build --delete-conflicting-outputs

# Watch mode (regénère automatiquement)
flutter pub run build_runner watch

# Nettoyer les fichiers générés
flutter pub run build_runner clean
```

## 📱 Gestion des Périphériques

### Périphériques

```powershell
# Lister les périphériques connectés
flutter devices

# Lister avec détails
flutter devices -v

# Vérifier si un périphérique est connecté
flutter devices --device-id=chrome
```

### Émulateurs

```powershell
# Lister les émulateurs disponibles
flutter emulators

# Lancer un émulateur
flutter emulators --launch Pixel_5_API_33

# Créer un nouvel émulateur
flutter emulators --create
```

## 🔍 Performance et Profiling

### Profiling

```powershell
# Lancer en mode profile
flutter run --profile

# Afficher les statistiques de performance
flutter run --profile --trace-skia

# Analyser la taille de l'app
flutter build apk --analyze-size
flutter build appbundle --analyze-size
```

### Screenshots

```powershell
# Prendre une capture d'écran
flutter screenshot

# Sauvegarder dans un fichier
flutter screenshot --out=screenshot.png
```

## 🌍 Internationalisation

```powershell
# Générer les fichiers de traduction
flutter pub run intl_translation:extract_to_arb --output-dir=lib/l10n lib/main.dart

# Générer les fichiers Dart depuis ARB
flutter pub run intl_translation:generate_from_arb --output-dir=lib/l10n \
  --no-use-deferred-loading lib/main.dart lib/l10n/intl_*.arb
```

## 🔐 Signing et Certificats

### Android

```powershell
# Générer un keystore
keytool -genkey -v -keystore ~/upload-keystore.jks -keyalg RSA -keysize 2048 -validity 10000 -alias upload

# Vérifier le keystore
keytool -list -v -keystore ~/upload-keystore.jks
```

## 📊 Analyse et Optimisation

```powershell
# Analyser la taille de build
flutter build apk --analyze-size
flutter build appbundle --analyze-size
flutter build ios --analyze-size

# Afficher l'arbre des dépendances
flutter pub deps --style=compact

# Vérifier les packages inutilisés
flutter pub deps | grep -i unused
```

## 🧹 Maintenance

```powershell
# Nettoyer complètement le projet
flutter clean
rm -rf pubspec.lock
flutter pub get

# Réinitialiser Flutter
flutter upgrade --force
flutter pub cache repair

# Supprimer le cache de packages
flutter pub cache clean
```

## 📚 Documentation et Ressources

```powershell
# Générer la documentation API
flutter pub global activate dartdoc
dartdoc

# Ouvrir la documentation Flutter
flutter docs

# Ouvrir les samples Flutter
flutter create --sample=<sample_id> my_sample
```

## 🎯 Raccourcis Utiles

```powershell
# Cycle complet de développement
flutter clean && flutter pub get && flutter run

# Build release pour Android
flutter build apk --release --split-per-abi

# Tests avec coverage
flutter test --coverage && genhtml coverage/lcov.info -o coverage/html

# Mettre à jour et nettoyer
flutter upgrade && flutter clean && flutter pub get
```

## 📚 Fichiers de Documentation Associés

- **[INSTALLATION.md](INSTALLATION.md)** - Installation de Flutter
- **[PACKAGES-DEPENDANCES.md](PACKAGES-DEPENDANCES.md)** - Gestion des packages
- **[CONNEXION-API-SYMFONY.md](CONNEXION-API-SYMFONY.md)** - Intégration avec l'API Symfony
- **[DEPANNAGE.md](DEPANNAGE.md)** - Problèmes courants et solutions
