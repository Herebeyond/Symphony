# ⚠️ Dépannage Flutter

Ce guide rassemble les problèmes courants rencontrés avec Flutter et leurs solutions.

## 🔧 Problèmes d'Installation

### Erreur: "flutter command not found"

**Symptôme :** La commande `flutter` n'est pas reconnue dans PowerShell.

**Solution :**

```powershell
# Vérifier que Flutter est dans le PATH
$env:Path

# Ajouter temporairement pour la session actuelle
$env:Path += ";C:\src\flutter\bin"

# Pour ajouter définitivement :
# 1. Ouvrir Paramètres Système → Variables d'environnement
# 2. Modifier la variable Path
# 3. Ajouter C:\src\flutter\bin
# 4. Redémarrer PowerShell
```

### Erreur: "Unable to locate Android SDK"

**Symptôme :** Flutter ne trouve pas le SDK Android.

**Solution :**

```powershell
# Définir la variable d'environnement ANDROID_HOME
$env:ANDROID_HOME = "C:\Users\$env:USERNAME\AppData\Local\Android\Sdk"

# Ajouter au PATH
$env:Path += ";$env:ANDROID_HOME\platform-tools"
$env:Path += ";$env:ANDROID_HOME\tools"

# Pour rendre permanent, ajouter ces variables dans les variables d'environnement système
```

### Erreur: "cmdline-tools component is missing"

**Symptôme :** Android SDK command-line tools manquants.

**Solution :**

1. Ouvrir Android Studio
2. Aller dans **Tools** → **SDK Manager**
3. Onglet **SDK Tools**
4. Cocher **Android SDK Command-line Tools**
5. Cliquer sur **Apply**

```powershell
# Puis accepter les licences
flutter doctor --android-licenses
```

## 📦 Problèmes de Dépendances

### Erreur: "version solving failed"

**Symptôme :** Conflits de versions entre packages.

**Solution :**

```powershell
# Nettoyer et réinstaller
flutter clean
flutter pub get

# Si le problème persiste, mettre à jour les packages
flutter pub upgrade

# En dernier recours, supprimer pubspec.lock
rm pubspec.lock
flutter pub get
```

### Erreur: "The current Dart SDK version is..."

**Symptôme :** Version de Dart incompatible.

**Solution :**

```yaml
# Dans pubspec.yaml, ajuster la contrainte SDK
environment:
  sdk: '>=3.0.0 <4.0.0'  # Adapter selon votre version
```

```powershell
# Mettre à jour Flutter
flutter upgrade
```

### Erreur: "Because X depends on Y which doesn't exist"

**Symptôme :** Package introuvable.

**Solution :**

```powershell
# Vérifier le nom du package sur pub.dev
# Corriger le nom dans pubspec.yaml

# Nettoyer le cache
flutter pub cache repair

# Réinstaller
flutter pub get
```

## 🌐 Problèmes de Connexion API

### Erreur de certificat SSL (localhost HTTPS)

**Symptôme :** Erreur SSL lors de la connexion à l'API Symfony en HTTPS.

**Solution temporaire (développement uniquement) :**

```dart
import 'dart:io';

class MyHttpOverrides extends HttpOverrides {
  @override
  HttpClient createHttpClient(SecurityContext? context) {
    return super.createHttpClient(context)
      ..badCertificateCallback = (X509Certificate cert, String host, int port) => true;
  }
}

void main() {
  HttpOverrides.global = MyHttpOverrides();
  runApp(const MyApp());
}
```

**⚠️ Ne JAMAIS utiliser en production !**

**Solution pour production :**

1. Utiliser un certificat SSL valide
2. Ou utiliser HTTP (non recommandé)

### Erreur CORS

**Symptôme :** `CORS policy: No 'Access-Control-Allow-Origin' header`

**Solution côté Symfony :**

```bash
# Installer le bundle CORS
docker compose exec php composer require nelmio/cors-bundle
```

```yaml
# config/packages/nelmio_cors.yaml
nelmio_cors:
    defaults:
        origin_regex: true
        allow_origin: ['*']
        allow_methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'OPTIONS']
        allow_headers: ['Content-Type', 'Authorization', 'Accept']
        expose_headers: ['Link']
        max_age: 3600
    paths:
        '^/api': ~
```

### Erreur: "SocketException: Failed host lookup"

**Symptôme :** Impossible de se connecter à l'API.

**Causes possibles :**

1. **URL incorrecte** : Vérifier l'URL de l'API
2. **Pas de connexion Internet** : Vérifier la connexion
3. **API non démarrée** : Démarrer Symfony

**Solution :**

```dart
// Vérifier l'URL dans api_service.dart
static const String baseUrl = 'https://localhost/api';  // Correct ?

// Tester avec curl
curl https://localhost/api/users
```

### Erreur 401 Unauthorized

**Symptôme :** Accès refusé à l'API.

**Solution :**

```dart
// Vérifier que le token JWT est bien envoyé
static Map<String, String> get headers => {
  'Content-Type': 'application/json',
  'Accept': 'application/json',
  'Authorization': 'Bearer $_token',  // Token présent ?
};

// Vérifier la validité du token
print('Token: $_token');
```

## 🏗️ Problèmes de Build

### Erreur: "Gradle build failed"

**Symptôme :** Le build Android échoue.

**Solutions :**

```powershell
# 1. Nettoyer le projet
flutter clean

# 2. Nettoyer Gradle
cd android
./gradlew clean
cd ..

# 3. Rebuild
flutter build apk
```

**Si le problème persiste :**

```gradle
// android/app/build.gradle
android {
    compileSdkVersion 34  // Mettre à jour
    
    defaultConfig {
        minSdkVersion 21  // Augmenter si nécessaire
        targetSdkVersion 34
    }
}
```

### Erreur: "Out of memory" pendant le build

**Symptôme :** Build échoue avec erreur de mémoire.

**Solution :**

```properties
# android/gradle.properties
org.gradle.jvmargs=-Xmx4096m -XX:MaxPermSize=512m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8
```

### Erreur: "Execution failed for task ':app:lintVitalRelease'"

**Symptôme :** Lint échoue pendant le build release.

**Solution :**

```gradle
// android/app/build.gradle
android {
    lintOptions {
        checkReleaseBuilds false
        // Ou ignorer seulement certaines erreurs
        // disable 'InvalidPackage'
    }
}
```

## 📱 Problèmes d'Exécution

### Performance lente en mode debug

**Symptôme :** L'application est très lente.

**Explication :** C'est **normal** en mode debug. Flutter ajoute beaucoup de checks et d'outils de développement.

**Solution :**

```powershell
# Tester les performances réelles en mode release
flutter run --release

# Ou mode profile
flutter run --profile
```

### Hot Reload ne fonctionne pas

**Symptôme :** `r` ne recharge pas les modifications.

**Solutions :**

1. **Redémarrer complètement** : Taper `R` au lieu de `r`
2. **Vérifier les erreurs** : Corriger les erreurs de syntaxe
3. **Relancer l'app** : `q` puis `flutter run`

**Limitations du Hot Reload :**
- Ne fonctionne pas avec les modifications de `main()`
- Ne fonctionne pas avec les enum modifiés
- Ne fonctionne pas en mode release

### Erreur: "Waiting for another flutter command to release the startup lock"

**Symptôme :** Flutter reste bloqué.

**Solution :**

```powershell
# Supprimer le fichier de lock
rm $env:LOCALAPPDATA\Pub\Cache\.flutter_tool_state

# Ou localisation alternative
rm $env:USERPROFILE\.flutter-devtools\.flutter_tool_state

# Puis relancer la commande
flutter pub get
```

## 🎨 Problèmes d'Interface

### Erreur: "RenderFlex overflowed"

**Symptôme :** Débordement de widgets (pixels jaunes et noirs).

**Solution :**

```dart
// Utiliser Expanded ou Flexible
Column(
  children: [
    Expanded(  // Occupe l'espace disponible
      child: ListView(...),
    ),
  ],
)

// Ou SingleChildScrollView
SingleChildScrollView(
  child: Column(
    children: [
      // Widgets
    ],
  ),
)
```

### Erreur: "setState() called after dispose()"

**Symptôme :** Erreur après navigation ou fermeture du widget.

**Solution :**

```dart
Future<void> loadData() async {
  final data = await ApiService.getData();
  
  // Vérifier si le widget est encore monté
  if (mounted) {
    setState(() {
      _data = data;
    });
  }
}
```

### Widgets ne se mettent pas à jour

**Symptôme :** L'interface ne reflète pas les changements de données.

**Solution :**

```dart
// S'assurer d'appeler setState()
void updateData() {
  setState(() {
    _counter++;  // Modifications dans setState
  });
}

// Ou utiliser un StatefulWidget au lieu de StatelessWidget
```

## 🔌 Problèmes de Plugins

### Erreur: "MissingPluginException"

**Symptôme :** Plugin non trouvé lors de l'exécution.

**Solution :**

```powershell
# Nettoyer et rebuild
flutter clean
flutter pub get

# Si sur Android
cd android
./gradlew clean
cd ..

# Puis relancer
flutter run
```

**Si le problème persiste :**

```powershell
# Hot restart complet
# Dans l'app en cours : Taper 'R'

# Ou arrêter et relancer
q
flutter run
```

## 💾 Problèmes de Stockage Local

### Erreur: "PathProviderException"

**Symptôme :** Impossible d'accéder au stockage local.

**Solution Android :**

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<manifest>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
</manifest>
```

**Solution iOS :**

```xml
<!-- ios/Runner/Info.plist -->
<key>NSPhotoLibraryUsageDescription</key>
<string>Nous avons besoin d'accéder à vos photos</string>
```

## 🔄 Problèmes de Mise à Jour

### Erreur après mise à jour de Flutter

**Symptôme :** Le projet ne compile plus après `flutter upgrade`.

**Solution :**

```powershell
# 1. Nettoyer complètement
flutter clean
rm pubspec.lock

# 2. Réinstaller les dépendances
flutter pub get

# 3. Mettre à jour les packages
flutter pub upgrade

# 4. Si problème persiste, revenir en arrière
flutter downgrade
```

## 🌍 Problèmes Web

### Erreur: "CORS policy" sur Web

**Symptôme :** CORS bloque les requêtes en développement web.

**Solution :**

```powershell
# Lancer Chrome avec CORS désactivé (développement uniquement)
flutter run -d chrome --web-browser-flag "--disable-web-security"
```

### Erreur: "Failed to load network image"

**Symptôme :** Images réseau ne s'affichent pas sur web.

**Solution :**

```html
<!-- web/index.html -->
<head>
    <!-- Ajouter la meta tag -->
    <meta name="referrer" content="no-referrer">
</head>
```

## 🖥️ Problèmes Windows Desktop

### Erreur: "Visual Studio not found"

**Symptôme :** Build Windows échoue.

**Solution :**

1. Installer Visual Studio 2022
2. Cocher "Desktop development with C++"
3. Installer Windows 10/11 SDK

```powershell
# Activer le support Windows
flutter config --enable-windows-desktop

# Vérifier
flutter doctor
```

## 📊 Problèmes de Performance

### App consomme trop de mémoire

**Solutions :**

```dart
// 1. Utiliser const pour les widgets statiques
const Text('Hello')  // Au lieu de Text('Hello')

// 2. Disposer les contrôleurs
@override
void dispose() {
  _controller.dispose();
  super.dispose();
}

// 3. Optimiser les images
CachedNetworkImage(
  imageUrl: url,
  maxWidth: 500,  // Limiter la taille
)

// 4. Utiliser ListView.builder au lieu de ListView
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) => ItemWidget(items[index]),
)
```

## 🆘 Commandes de Diagnostic

```powershell
# Diagnostic complet
flutter doctor -v

# Vérifier la configuration
flutter config

# Analyser le code
flutter analyze

# Afficher les logs
flutter logs

# Nettoyer le cache
flutter pub cache repair

# Réinitialiser Flutter
flutter clean
flutter pub get
```

## 📚 Ressources Supplémentaires

### Documentation Officielle
- **Flutter Debugging** : https://docs.flutter.dev/testing/debugging
- **Common Errors** : https://docs.flutter.dev/testing/errors

### Communauté
- **Stack Overflow** : Tag `flutter`
- **GitHub Issues** : https://github.com/flutter/flutter/issues
- **Discord Flutter** : https://discord.gg/flutter

## 🎯 Checklist de Dépannage Générale

Avant de chercher plus loin, toujours essayer :

```powershell
# 1. Nettoyer le projet
flutter clean

# 2. Supprimer pubspec.lock
rm pubspec.lock

# 3. Réinstaller les dépendances
flutter pub get

# 4. Redémarrer l'IDE (VS Code)

# 5. Vérifier les mises à jour
flutter upgrade

# 6. Diagnostic complet
flutter doctor -v
```

## 📚 Fichiers de Documentation Associés

- **[INSTALLATION.md](INSTALLATION.md)** - Installation de Flutter
- **[PACKAGES-DEPENDANCES.md](PACKAGES-DEPENDANCES.md)** - Gestion des packages
- **[CONNEXION-API-SYMFONY.md](CONNEXION-API-SYMFONY.md)** - Intégration avec l'API Symfony
- **[COMMANDES-UTILES.md](COMMANDES-UTILES.md)** - Commandes Flutter courantes
