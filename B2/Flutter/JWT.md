# 🔐 Authentification JWT avec Flutter

**Ce guide explique comment utiliser les tokens JWT dans une application Flutter pour s'authentifier auprès d'une API Symfony.**

## 📋 Qu'est-ce qu'un JWT ?

Un **JWT** (JSON Web Token) est un petit jeton de données utilisé pour prouver l'identité d'un utilisateur.

### Comment ça fonctionne dans Flutter ?

```
1. L'utilisateur saisit email + mot de passe
2. Flutter envoie les identifiants à l'API Symfony
3. Symfony retourne un token JWT
4. Flutter stocke le token de manière sécurisée
5. Flutter ajoute le token à chaque requête : "Authorization: Bearer <token>"
6. L'API valide le token et autorise l'accès
```

## ⚠️ PRÉREQUIS

**AVANT DE COMMENCER :**
- ✅ **API Symfony** avec JWT configuré (voir [JWT.md](../Symfony/JWT.md))
- ✅ **Flutter** installé et fonctionnel (voir [INSTALLATION.md](INSTALLATION.md))
- ✅ **Package http** pour les requêtes HTTP

## 🎯 INSTALLATION ÉTAPE PAR ÉTAPE

### Étape 1 : Installer les Packages Nécessaires

Ajouter les dépendances dans `pubspec.yaml` :

```yaml
dependencies:
  flutter:
    sdk: flutter
  http: ^1.1.0
  flutter_secure_storage: ^9.0.0
```

```powershell
# Installer les packages
flutter pub get
```

**📦 Packages utilisés :**
- **http** : Pour envoyer des requêtes HTTP à l'API
- **flutter_secure_storage** : Pour stocker le token de manière sécurisée

### Étape 2 : Créer la Structure de Dossiers

```
lib/
├── main.dart
├── models/
│   └── user.dart
├── services/
│   ├── auth_service.dart
│   └── api_service.dart
└── screens/
    ├── login_screen.dart
    └── home_screen.dart
```

## 🔧 IMPLÉMENTATION

### Étape 3 : Créer le Service d'Authentification

Créer le fichier `lib/services/auth_service.dart` :

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

class AuthService {
  // URL de votre API Symfony
  static const String baseUrl = 'https://localhost/api';
  
  // Instance de stockage sécurisé
  final storage = const FlutterSecureStorage();
  
  /// Connexion : envoie les identifiants et récupère le token
  Future<bool> login(String email, String password) async {
    try {
      final response = await http.post(
        Uri.parse('$baseUrl/login_check'),
        headers: {'Content-Type': 'application/json'},
        body: jsonEncode({
          'email': email,
          'password': password,
        }),
      );
      
      if (response.statusCode == 200) {
        final data = jsonDecode(response.body);
        final token = data['token'];
        
        // Sauvegarder le token de manière sécurisée
        await storage.write(key: 'jwt_token', value: token);
        
        // Optionnel : sauvegarder l'email
        await storage.write(key: 'user_email', value: email);
        
        return true;
      } else {
        print('Erreur de connexion: ${response.statusCode}');
        return false;
      }
    } catch (e) {
      print('Exception lors de la connexion: $e');
      return false;
    }
  }
  
  /// Récupérer le token stocké
  Future<String?> getToken() async {
    return await storage.read(key: 'jwt_token');
  }
  
  /// Récupérer l'email de l'utilisateur
  Future<String?> getUserEmail() async {
    return await storage.read(key: 'user_email');
  }
  
  /// Déconnexion : supprimer le token
  Future<void> logout() async {
    await storage.delete(key: 'jwt_token');
    await storage.delete(key: 'user_email');
  }
  
  /// Vérifier si l'utilisateur est connecté
  Future<bool> isLoggedIn() async {
    final token = await getToken();
    return token != null && token.isNotEmpty;
  }
  
  /// Vérifier si le token est valide (optionnel)
  Future<bool> isTokenValid() async {
    final token = await getToken();
    if (token == null) return false;
    
    try {
      // Tenter une requête vers une route protégée
      final response = await http.get(
        Uri.parse('$baseUrl/users'),
        headers: {'Authorization': 'Bearer $token'},
      );
      
      return response.statusCode == 200;
    } catch (e) {
      return false;
    }
  }
}
```

### Étape 4 : Créer le Service API Authentifié

Créer le fichier `lib/services/api_service.dart` :

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;
import 'auth_service.dart';

class ApiService {
  static const String baseUrl = 'https://localhost/api';
  final AuthService _authService = AuthService();
  
  /// Récupérer les headers avec le token JWT
  Future<Map<String, String>> _getHeaders() async {
    final token = await _authService.getToken();
    return {
      'Content-Type': 'application/json',
      'Accept': 'application/json',
      if (token != null) 'Authorization': 'Bearer $token',
    };
  }
  
  /// Exemple : Récupérer une liste de données (GET)
  Future<List<dynamic>> fetchData(String endpoint) async {
    try {
      final headers = await _getHeaders();
      final response = await http.get(
        Uri.parse('$baseUrl/$endpoint'),
        headers: headers,
      );
      
      if (response.statusCode == 200) {
        return jsonDecode(response.body) as List<dynamic>;
      } else if (response.statusCode == 401) {
        // Token expiré ou invalide -> déconnecter l'utilisateur
        await _authService.logout();
        throw Exception('Session expirée, veuillez vous reconnecter');
      } else {
        throw Exception('Erreur ${response.statusCode}: ${response.body}');
      }
    } catch (e) {
      throw Exception('Erreur API: $e');
    }
  }
  
  /// Exemple : Créer une ressource (POST)
  Future<Map<String, dynamic>> createData(String endpoint, Map<String, dynamic> data) async {
    try {
      final headers = await _getHeaders();
      final response = await http.post(
        Uri.parse('$baseUrl/$endpoint'),
        headers: headers,
        body: jsonEncode(data),
      );
      
      if (response.statusCode == 201 || response.statusCode == 200) {
        return jsonDecode(response.body);
      } else if (response.statusCode == 401) {
        await _authService.logout();
        throw Exception('Session expirée, veuillez vous reconnecter');
      } else {
        throw Exception('Erreur ${response.statusCode}: ${response.body}');
      }
    } catch (e) {
      throw Exception('Erreur API: $e');
    }
  }
  
  /// Exemple : Mettre à jour une ressource (PUT)
  Future<Map<String, dynamic>> updateData(String endpoint, Map<String, dynamic> data) async {
    try {
      final headers = await _getHeaders();
      final response = await http.put(
        Uri.parse('$baseUrl/$endpoint'),
        headers: headers,
        body: jsonEncode(data),
      );
      
      if (response.statusCode == 200) {
        return jsonDecode(response.body);
      } else if (response.statusCode == 401) {
        await _authService.logout();
        throw Exception('Session expirée, veuillez vous reconnecter');
      } else {
        throw Exception('Erreur ${response.statusCode}: ${response.body}');
      }
    } catch (e) {
      throw Exception('Erreur API: $e');
    }
  }
  
  /// Exemple : Supprimer une ressource (DELETE)
  Future<void> deleteData(String endpoint) async {
    try {
      final headers = await _getHeaders();
      final response = await http.delete(
        Uri.parse('$baseUrl/$endpoint'),
        headers: headers,
      );
      
      if (response.statusCode != 204 && response.statusCode != 200) {
        if (response.statusCode == 401) {
          await _authService.logout();
          throw Exception('Session expirée, veuillez vous reconnecter');
        }
        throw Exception('Erreur ${response.statusCode}: ${response.body}');
      }
    } catch (e) {
      throw Exception('Erreur API: $e');
    }
  }
}
```

### Étape 5 : Créer la Page de Connexion

Créer le fichier `lib/screens/login_screen.dart` :

```dart
import 'package:flutter/material.dart';
import '../services/auth_service.dart';

class LoginScreen extends StatefulWidget {
  const LoginScreen({Key? key}) : super(key: key);

  @override
  State<LoginScreen> createState() => _LoginScreenState();
}

class _LoginScreenState extends State<LoginScreen> {
  final _formKey = GlobalKey<FormState>();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  final _authService = AuthService();
  
  bool _isLoading = false;
  bool _obscurePassword = true;
  
  Future<void> _login() async {
    if (!_formKey.currentState!.validate()) {
      return;
    }
    
    setState(() => _isLoading = true);
    
    final success = await _authService.login(
      _emailController.text.trim(),
      _passwordController.text,
    );
    
    setState(() => _isLoading = false);
    
    if (success && mounted) {
      // Connexion réussie -> redirection vers la page d'accueil
      Navigator.pushReplacementNamed(context, '/home');
    } else if (mounted) {
      // Afficher un message d'erreur
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(
          content: Text('Identifiants incorrects'),
          backgroundColor: Colors.red,
        ),
      );
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Connexion'),
        centerTitle: true,
      ),
      body: Center(
        child: SingleChildScrollView(
          padding: const EdgeInsets.all(24.0),
          child: Form(
            key: _formKey,
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              crossAxisAlignment: CrossAxisAlignment.stretch,
              children: [
                // Logo ou titre
                const Icon(
                  Icons.lock_outline,
                  size: 80,
                  color: Colors.blue,
                ),
                const SizedBox(height: 32),
                
                // Champ email
                TextFormField(
                  controller: _emailController,
                  decoration: const InputDecoration(
                    labelText: 'Email',
                    prefixIcon: Icon(Icons.email),
                    border: OutlineInputBorder(),
                  ),
                  keyboardType: TextInputType.emailAddress,
                  autocorrect: false,
                  validator: (value) {
                    if (value == null || value.isEmpty) {
                      return 'Veuillez saisir votre email';
                    }
                    if (!value.contains('@')) {
                      return 'Email invalide';
                    }
                    return null;
                  },
                ),
                const SizedBox(height: 16),
                
                // Champ mot de passe
                TextFormField(
                  controller: _passwordController,
                  decoration: InputDecoration(
                    labelText: 'Mot de passe',
                    prefixIcon: const Icon(Icons.lock),
                    border: const OutlineInputBorder(),
                    suffixIcon: IconButton(
                      icon: Icon(
                        _obscurePassword ? Icons.visibility : Icons.visibility_off,
                      ),
                      onPressed: () {
                        setState(() => _obscurePassword = !_obscurePassword);
                      },
                    ),
                  ),
                  obscureText: _obscurePassword,
                  validator: (value) {
                    if (value == null || value.isEmpty) {
                      return 'Veuillez saisir votre mot de passe';
                    }
                    return null;
                  },
                ),
                const SizedBox(height: 24),
                
                // Bouton de connexion
                SizedBox(
                  height: 50,
                  child: _isLoading
                      ? const Center(child: CircularProgressIndicator())
                      : ElevatedButton(
                          onPressed: _login,
                          child: const Text(
                            'Se connecter',
                            style: TextStyle(fontSize: 16),
                          ),
                        ),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
  
  @override
  void dispose() {
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }
}
```

### Étape 6 : Créer la Page d'Accueil

Créer le fichier `lib/screens/home_screen.dart` :

```dart
import 'package:flutter/material.dart';
import '../services/auth_service.dart';
import '../services/api_service.dart';

class HomeScreen extends StatefulWidget {
  const HomeScreen({Key? key}) : super(key: key);

  @override
  State<HomeScreen> createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> {
  final _authService = AuthService();
  final _apiService = ApiService();
  
  String? _userEmail;
  
  @override
  void initState() {
    super.initState();
    _loadUserInfo();
  }
  
  Future<void> _loadUserInfo() async {
    final email = await _authService.getUserEmail();
    setState(() => _userEmail = email);
  }
  
  Future<void> _logout() async {
    await _authService.logout();
    if (mounted) {
      Navigator.pushReplacementNamed(context, '/login');
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Accueil'),
        actions: [
          IconButton(
            icon: const Icon(Icons.logout),
            onPressed: _logout,
            tooltip: 'Déconnexion',
          ),
        ],
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(
              Icons.check_circle_outline,
              size: 100,
              color: Colors.green,
            ),
            const SizedBox(height: 24),
            const Text(
              'Connexion réussie !',
              style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 16),
            if (_userEmail != null)
              Text(
                'Connecté en tant que: $_userEmail',
                style: const TextStyle(fontSize: 16),
              ),
          ],
        ),
      ),
    );
  }
}
```

### Étape 7 : Configurer le Fichier Principal

Modifier le fichier `lib/main.dart` :

```dart
import 'package:flutter/material.dart';
import 'services/auth_service.dart';
import 'screens/login_screen.dart';
import 'screens/home_screen.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'JWT Authentication',
      theme: ThemeData(
        primarySwatch: Colors.blue,
        useMaterial3: true,
      ),
      home: const AuthCheck(),
      routes: {
        '/login': (context) => const LoginScreen(),
        '/home': (context) => const HomeScreen(),
      },
    );
  }
}

/// Widget pour vérifier l'état de connexion au démarrage
class AuthCheck extends StatelessWidget {
  const AuthCheck({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    final authService = AuthService();
    
    return FutureBuilder<bool>(
      future: authService.isLoggedIn(),
      builder: (context, snapshot) {
        // Pendant le chargement
        if (snapshot.connectionState == ConnectionState.waiting) {
          return const Scaffold(
            body: Center(
              child: CircularProgressIndicator(),
            ),
          );
        }
        
        // Si connecté -> Page d'accueil
        if (snapshot.data == true) {
          return const HomeScreen();
        }
        
        // Sinon -> Page de connexion
        return const LoginScreen();
      },
    );
  }
}
```

## 🧪 TESTS

### Tester l'Application

```powershell
# Lancer l'application
flutter run
```

**Scénario de test :**

1. ✅ L'app s'ouvre sur la page de connexion
2. ✅ Saisir un email et mot de passe valides
3. ✅ Cliquer sur "Se connecter"
4. ✅ Redirection vers la page d'accueil
5. ✅ Fermer et relancer l'app
6. ✅ L'app s'ouvre directement sur la page d'accueil (token mémorisé)
7. ✅ Cliquer sur déconnexion
8. ✅ Retour à la page de connexion

### Déboguer les Erreurs

**Erreur de connexion :**
```dart
// Activer les logs pour voir les erreurs
print('Status: ${response.statusCode}');
print('Body: ${response.body}');
```

**Problème de certificat SSL (en développement uniquement) :**

⚠️ **NE JAMAIS FAIRE EN PRODUCTION**

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
  HttpOverrides.global = MyHttpOverrides(); // ⚠️ Développement uniquement
  runApp(const MyApp());
}
```

## 🔧 FONCTIONNALITÉS AVANCÉES

### Gérer le Refresh Token

Si votre API supporte les refresh tokens :

```dart
Future<bool> refreshToken(String refreshToken) async {
  try {
    final response = await http.post(
      Uri.parse('$baseUrl/token/refresh'),
      headers: {'Content-Type': 'application/json'},
      body: jsonEncode({'refresh_token': refreshToken}),
    );
    
    if (response.statusCode == 200) {
      final data = jsonDecode(response.body);
      await storage.write(key: 'jwt_token', value: data['token']);
      await storage.write(key: 'refresh_token', value: data['refresh_token']);
      return true;
    }
    return false;
  } catch (e) {
    return false;
  }
}
```

### Intercepter les Erreurs 401 Automatiquement

Utiliser le package `dio` avec des intercepteurs :

```yaml
dependencies:
  dio: ^5.3.0
```

```dart
import 'package:dio/dio.dart';

class AuthInterceptor extends Interceptor {
  final AuthService _authService = AuthService();
  
  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) async {
    final token = await _authService.getToken();
    if (token != null) {
      options.headers['Authorization'] = 'Bearer $token';
    }
    handler.next(options);
  }
  
  @override
  void onError(DioException err, ErrorInterceptorHandler handler) async {
    if (err.response?.statusCode == 401) {
      // Token expiré -> déconnecter
      await _authService.logout();
      // Naviguer vers login
    }
    handler.next(err);
  }
}
```

## 🛡️ BONNES PRATIQUES DE SÉCURITÉ

### Côté Flutter

1. **Toujours utiliser flutter_secure_storage**
   - Ne JAMAIS utiliser SharedPreferences pour le token
   - Les données sont chiffrées sur l'appareil

2. **Ne jamais logger le token en production**
   ```dart
   if (kDebugMode) {
     print('Token: $token'); // Seulement en debug
   }
   ```

3. **Gérer l'expiration du token**
   - Intercepter les erreurs 401
   - Rediriger vers la page de connexion

4. **Effacer le token à la déconnexion**
   ```dart
   await storage.deleteAll(); // Nettoyer tout
   ```

5. **Valider les certificats SSL en production**
   - Ne pas désactiver la vérification SSL

6. **Valider les entrées utilisateur**
   - Utiliser des validators pour les formulaires

## 📚 Ressources Complémentaires

- 📖 [Package http](https://pub.dev/packages/http)
- 📖 [Package flutter_secure_storage](https://pub.dev/packages/flutter_secure_storage)
- 📖 [Package dio (alternative à http)](https://pub.dev/packages/dio)
- 📖 [JWT.io - Décoder les tokens](https://jwt.io)

## 🔗 Guides Connexes

- [JWT.md](../Symfony/JWT.md) - Configuration JWT côté Symfony
- [CONNEXION-API-SYMFONY.md](CONNEXION-API-SYMFONY.md) - Connexion Flutter-Symfony
- [INSTALLATION.md](INSTALLATION.md) - Installation de Flutter
- [DEPANNAGE.md](DEPANNAGE.md) - Solutions aux problèmes courants

## 📊 Récapitulatif des Packages

```yaml
dependencies:
  http: ^1.1.0                      # Requêtes HTTP
  flutter_secure_storage: ^9.0.0   # Stockage sécurisé
```

```powershell
flutter pub get
```

---

**✅ Votre application Flutter peut maintenant s'authentifier avec JWT auprès de votre API Symfony !**
