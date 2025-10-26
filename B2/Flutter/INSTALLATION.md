# 📱 Installation de Flutter

Ce guide vous explique comment installer Flutter sur Windows pour créer des applications multiplateformes.

## 🎯 Qu'est-ce que Flutter ?

Flutter est un framework open-source développé par **Google** qui permet de créer des applications natives pour :
- 📱 **Mobile** : iOS et Android
- 🌐 **Web** : Applications web progressives (PWA)
- 💻 **Desktop** : Windows, macOS, Linux
- 🔗 **Embedded** : Systèmes embarqués

**Caractéristiques principales :**
- ✅ **Un seul code source** pour toutes les plateformes
- ✅ **Performance native** grâce au compilateur Dart
- ✅ **Hot Reload** pour un développement rapide
- ✅ **Widgets personnalisables** pour créer des UI modernes
- ✅ **Large écosystème** de packages sur pub.dev

## 🔗 Lien avec Symfony

Flutter et Symfony forment une architecture **client-serveur** moderne :

```
┌─────────────────────────┐
│   Flutter Application   │  ← Frontend (Mobile/Web/Desktop)
│  (Interface Utilisateur)│
└───────────┬─────────────┘
            │
            │ HTTP/HTTPS (REST API ou GraphQL)
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

**Rôles :**
- **Flutter** : Gère l'interface utilisateur et l'expérience utilisateur (UX)
- **Symfony** : Expose des API REST, gère la logique métier et la base de données
- **Communication** : Requêtes HTTP (GET, POST, PUT, DELETE) avec JSON

## ⚠️ Prérequis

### Configuration Minimale
- **Système d'exploitation** : Windows 10/11 (64-bit), macOS, ou Linux
- **Espace disque** : Au moins 2.5 GB (sans IDE/Android Studio)
- **RAM** : Minimum 4 GB (8 GB recommandés)
- **Git** : Pour Windows ([télécharger ici](https://git-scm.com/download/win))

### Outils Optionnels (selon plateforme cible)
- **Android Studio** : Pour développement Android
- **Xcode** : Pour développement iOS (macOS uniquement)
- **Visual Studio** : Pour développement Windows Desktop
- **Chrome** : Pour développement Web

## 🚀 Installation de Flutter (Windows)

### Étape 1 : Télécharger Flutter SDK

```powershell
# Créer un dossier pour Flutter (éviter Program Files)
New-Item -Path "C:\src" -ItemType Directory -Force
cd C:\src

# Télécharger Flutter SDK (version stable)
# Aller sur https://flutter.dev/docs/get-started/install/windows
# Télécharger le fichier ZIP et l'extraire dans C:\src\flutter
```

**💡 Alternative - Utiliser Git :**
```powershell
# Cloner le dépôt Flutter (méthode recommandée)
cd C:\src
git clone https://github.com/flutter/flutter.git -b stable
```

### Étape 2 : Configurer les Variables d'Environnement

```powershell
# Ajouter Flutter au PATH (méthode temporaire pour la session actuelle)
$env:Path += ";C:\src\flutter\bin"

# Vérifier l'installation
flutter --version
```

**📝 Configuration Permanente du PATH :**

1. Ouvrir **Paramètres Système** → **Variables d'environnement**
2. Dans **Variables système**, sélectionner **Path** → **Modifier**
3. Cliquer sur **Nouveau** et ajouter : `C:\src\flutter\bin`
4. Cliquer sur **OK** pour valider
5. **Redémarrer PowerShell** pour appliquer les changements

### Étape 3 : Vérifier l'Installation

```powershell
# Exécuter le diagnostic Flutter
flutter doctor
```

**🔍 Sortie attendue :**
```
Doctor summary (to see all details, run flutter doctor -v):
[√] Flutter (Channel stable, 3.24.0, on Microsoft Windows)
[!] Android toolchain - develop for Android devices
[!] Chrome - develop for the web
[√] Visual Studio - develop Windows apps
[!] Android Studio (not installed)
[√] VS Code (version 1.95.0)
[√] Connected device (1 available)
[√] Network resources

! Doctor found issues in 3 categories.
```

**✅ C'est normal d'avoir des avertissements** - vous n'avez besoin que des outils pour vos plateformes cibles.

### Étape 4 : Installer les Outils de Développement (Optionnel)

#### Pour Android :

```powershell
# Télécharger et installer Android Studio
# https://developer.android.com/studio

# Après installation, lancer Android Studio et installer :
# - Android SDK
# - Android SDK Command-line Tools
# - Android SDK Build-Tools
# - Android Emulator

# Accepter les licences Android
flutter doctor --android-licenses
```

#### Pour Web :

```powershell
# Installer/Vérifier Chrome
# Flutter utilise Chrome pour le développement web
# https://www.google.com/chrome/
```

#### Pour Windows Desktop :

```powershell
# Installer Visual Studio 2022 avec :
# - Desktop development with C++
# - Windows 10/11 SDK

# Activer le support Windows
flutter config --enable-windows-desktop
```

### Étape 5 : Configurer un Éditeur (VS Code)

```powershell
# Installer VS Code si pas déjà fait
# https://code.visualstudio.com/

# Installer les extensions Flutter et Dart depuis VS Code
# Extension ID: Dart-Code.flutter
# Extension ID: Dart-Code.dart-code
```

**📝 Extensions VS Code recommandées :**
- **Flutter** (Dart-Code.flutter)
- **Dart** (Dart-Code.dart-code)
- **Awesome Flutter Snippets** (Nash.awesome-flutter-snippets)
- **Pubspec Assist** (jeroen-meijer.pubspec-assist)

## 🎨 Créer Votre Première Application Flutter

### Créer un Nouveau Projet

```powershell
# Naviguer vers votre dossier de projets
cd C:\Users\baill\Docker

# Créer une nouvelle application Flutter
flutter create mon_app_flutter

# Entrer dans le projet
cd mon_app_flutter

# Ouvrir dans VS Code
code .
```

**🎯 Structure du projet créé :**
```
mon_app_flutter/
├── android/              # Code spécifique Android
├── ios/                  # Code spécifique iOS
├── linux/                # Code spécifique Linux
├── macos/                # Code spécifique macOS
├── web/                  # Code spécifique Web
├── windows/              # Code spécifique Windows
├── lib/
│   └── main.dart        # Point d'entrée de l'application
├── test/                 # Tests unitaires
├── pubspec.yaml         # Dépendances et configuration
└── README.md
```

### Exécuter l'Application

```powershell
# Lister les périphériques disponibles
flutter devices

# Lancer l'app sur Chrome (Web)
flutter run -d chrome

# Lancer l'app sur Windows Desktop
flutter run -d windows

# Lancer l'app sur un émulateur Android
flutter run -d emulator-5554
```

**💡 Mode Hot Reload :**
- Pendant que l'app tourne, tapez `r` pour recharger
- Tapez `R` pour un redémarrage complet
- Tapez `q` pour quitter

### Structure de base de main.dart

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Mon App Flutter',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const MyHomePage(title: 'Page d\'Accueil Flutter'),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key, required this.title});

  final String title;

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  int _counter = 0;

  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        title: Text(widget.title),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            const Text(
              'Vous avez appuyé sur le bouton :',
            ),
            Text(
              '$_counter',
              style: Theme.of(context).textTheme.headlineMedium,
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter,
        tooltip: 'Incrémenter',
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

## 📚 Fichiers de Documentation Associés

- **[PACKAGES-DEPENDANCES.md](PACKAGES-DEPENDANCES.md)** - Gestion des packages et dépendances
- **[CONNEXION-API-SYMFONY.md](CONNEXION-API-SYMFONY.md)** - Intégration avec l'API Symfony
- **[COMMANDES-UTILES.md](COMMANDES-UTILES.md)** - Commandes Flutter courantes
- **[DEPANNAGE.md](DEPANNAGE.md)** - Problèmes courants et solutions

## 🎯 Prochaines Étapes

1. ✅ **Apprendre Dart** : Langage de programmation utilisé par Flutter
2. ✅ **Widgets** : Comprendre StatelessWidget vs StatefulWidget
3. ✅ **Packages** : Installer et utiliser des packages (voir [PACKAGES-DEPENDANCES.md](PACKAGES-DEPENDANCES.md))
4. ✅ **API Integration** : Connexion avec Symfony (voir [CONNEXION-API-SYMFONY.md](CONNEXION-API-SYMFONY.md))
5. ✅ **State Management** : Provider, Riverpod, Bloc, GetX
6. ✅ **Navigation** : Routes et navigation entre écrans

## 📝 Notes Importantes

- **Hot Reload** ne fonctionne qu'en mode debug (pas en release)
- Utilisez `const` autant que possible pour optimiser les performances
- Les widgets sont **immuables** - utilisez `setState()` pour mettre à jour l'UI
- Préférez **StatelessWidget** quand l'état n'est pas nécessaire
- Testez toujours sur de vrais appareils avant le déploiement
