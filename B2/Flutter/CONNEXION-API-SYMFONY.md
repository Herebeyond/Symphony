# 🌐 Connexion Flutter avec API Symfony

Ce guide explique comment connecter votre application Flutter avec une API REST Symfony (API Platform).

## 🎯 Architecture Client-Serveur

```
Flutter App (Frontend)
    ↓
Requêtes HTTP (JSON)
    ↓
Symfony API Platform (Backend)
    ↓
Base de Données PostgreSQL
```

## 📋 Prérequis

### Côté Flutter
```yaml
dependencies:
  http: ^1.1.0  # Package pour les requêtes HTTP
```

```powershell
# Installer le package
flutter pub add http
```

### Côté Symfony
- ✅ API Platform installée (voir [API-PLATFORM.md](../../B2/Symfony/API-PLATFORM.md))
- ✅ CORS configuré pour autoriser Flutter
- ✅ Entités exposées via l'API

## 🛠️ Étape 1 : Créer un Service HTTP

Créer le fichier `lib/services/api_service.dart` :

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

class ApiService {
  // URL de base de votre API Symfony
  static const String baseUrl = 'https://localhost/api';
  
  // Headers par défaut
  static Map<String, String> get headers => {
    'Content-Type': 'application/json',
    'Accept': 'application/json',
  };

  // GET - Récupérer une liste d'éléments
  static Future<List<dynamic>> getItems(String endpoint) async {
    try {
      final response = await http.get(
        Uri.parse('$baseUrl/$endpoint'),
        headers: headers,
      );

      if (response.statusCode == 200) {
        return json.decode(response.body);
      } else {
        throw Exception('Erreur ${response.statusCode}: ${response.body}');
      }
    } catch (e) {
      throw Exception('Erreur de connexion: $e');
    }
  }

  // GET - Récupérer un élément par ID
  static Future<Map<String, dynamic>> getItem(String endpoint, int id) async {
    try {
      final response = await http.get(
        Uri.parse('$baseUrl/$endpoint/$id'),
        headers: headers,
      );

      if (response.statusCode == 200) {
        return json.decode(response.body);
      } else {
        throw Exception('Erreur ${response.statusCode}: ${response.body}');
      }
    } catch (e) {
      throw Exception('Erreur de connexion: $e');
    }
  }

  // POST - Créer un nouvel élément
  static Future<Map<String, dynamic>> createItem(
    String endpoint,
    Map<String, dynamic> data,
  ) async {
    try {
      final response = await http.post(
        Uri.parse('$baseUrl/$endpoint'),
        headers: headers,
        body: json.encode(data),
      );

      if (response.statusCode == 201 || response.statusCode == 200) {
        return json.decode(response.body);
      } else {
        throw Exception('Erreur ${response.statusCode}: ${response.body}');
      }
    } catch (e) {
      throw Exception('Erreur de création: $e');
    }
  }

  // PUT - Mettre à jour un élément
  static Future<Map<String, dynamic>> updateItem(
    String endpoint,
    int id,
    Map<String, dynamic> data,
  ) async {
    try {
      final response = await http.put(
        Uri.parse('$baseUrl/$endpoint/$id'),
        headers: headers,
        body: json.encode(data),
      );

      if (response.statusCode == 200) {
        return json.decode(response.body);
      } else {
        throw Exception('Erreur ${response.statusCode}: ${response.body}');
      }
    } catch (e) {
      throw Exception('Erreur de mise à jour: $e');
    }
  }

  // PATCH - Mise à jour partielle
  static Future<Map<String, dynamic>> patchItem(
    String endpoint,
    int id,
    Map<String, dynamic> data,
  ) async {
    try {
      final response = await http.patch(
        Uri.parse('$baseUrl/$endpoint/$id'),
        headers: headers,
        body: json.encode(data),
      );

      if (response.statusCode == 200) {
        return json.decode(response.body);
      } else {
        throw Exception('Erreur ${response.statusCode}: ${response.body}');
      }
    } catch (e) {
      throw Exception('Erreur de mise à jour: $e');
    }
  }

  // DELETE - Supprimer un élément
  static Future<void> deleteItem(String endpoint, int id) async {
    try {
      final response = await http.delete(
        Uri.parse('$baseUrl/$endpoint/$id'),
        headers: headers,
      );

      if (response.statusCode != 204 && response.statusCode != 200) {
        throw Exception('Erreur ${response.statusCode}: ${response.body}');
      }
    } catch (e) {
      throw Exception('Erreur de suppression: $e');
    }
  }
}
```

## 📦 Étape 2 : Créer un Modèle de Données

Créer le fichier `lib/models/user.dart` :

```dart
class User {
  final int id;
  final String email;
  final String nom;
  final String prenom;

  User({
    required this.id,
    required this.email,
    required this.nom,
    required this.prenom,
  });

  // Créer un User depuis JSON (API Symfony)
  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      id: json['id'],
      email: json['email'],
      nom: json['nom'],
      prenom: json['prenom'],
    );
  }

  // Convertir un User en JSON (pour envoyer à l'API)
  Map<String, dynamic> toJson() {
    return {
      'email': email,
      'nom': nom,
      'prenom': prenom,
    };
  }

  // Copie avec modification
  User copyWith({
    int? id,
    String? email,
    String? nom,
    String? prenom,
  }) {
    return User(
      id: id ?? this.id,
      email: email ?? this.email,
      nom: nom ?? this.nom,
      prenom: prenom ?? this.prenom,
    );
  }
}
```

## 🎨 Étape 3 : Utiliser l'API dans un Widget

Créer le fichier `lib/screens/users_screen.dart` :

```dart
import 'package:flutter/material.dart';
import '../services/api_service.dart';
import '../models/user.dart';

class UsersScreen extends StatefulWidget {
  const UsersScreen({super.key});

  @override
  State<UsersScreen> createState() => _UsersScreenState();
}

class _UsersScreenState extends State<UsersScreen> {
  List<User> users = [];
  bool isLoading = true;
  String errorMessage = '';

  @override
  void initState() {
    super.initState();
    loadUsers();
  }

  Future<void> loadUsers() async {
    setState(() {
      isLoading = true;
      errorMessage = '';
    });

    try {
      final data = await ApiService.getItems('users');
      setState(() {
        users = data.map((json) => User.fromJson(json)).toList();
        isLoading = false;
      });
    } catch (e) {
      setState(() {
        errorMessage = e.toString();
        isLoading = false;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Liste des Utilisateurs'),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: loadUsers,
          ),
        ],
      ),
      body: _buildBody(),
      floatingActionButton: FloatingActionButton(
        onPressed: _showAddUserDialog,
        child: const Icon(Icons.add),
      ),
    );
  }

  Widget _buildBody() {
    if (isLoading) {
      return const Center(child: CircularProgressIndicator());
    }

    if (errorMessage.isNotEmpty) {
      return Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.error, size: 60, color: Colors.red),
            const SizedBox(height: 16),
            Text(
              errorMessage,
              textAlign: TextAlign.center,
              style: const TextStyle(color: Colors.red),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: loadUsers,
              child: const Text('Réessayer'),
            ),
          ],
        ),
      );
    }

    if (users.isEmpty) {
      return const Center(child: Text('Aucun utilisateur'));
    }

    return ListView.builder(
      itemCount: users.length,
      itemBuilder: (context, index) {
        final user = users[index];
        return ListTile(
          leading: CircleAvatar(
            child: Text(user.prenom[0] + user.nom[0]),
          ),
          title: Text('${user.prenom} ${user.nom}'),
          subtitle: Text(user.email),
          trailing: Row(
            mainAxisSize: MainAxisSize.min,
            children: [
              IconButton(
                icon: const Icon(Icons.edit),
                onPressed: () => _showEditUserDialog(user),
              ),
              IconButton(
                icon: const Icon(Icons.delete),
                onPressed: () => _deleteUser(user.id),
              ),
            ],
          ),
        );
      },
    );
  }

  Future<void> _showAddUserDialog() async {
    final emailController = TextEditingController();
    final nomController = TextEditingController();
    final prenomController = TextEditingController();

    await showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('Ajouter un utilisateur'),
        content: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            TextField(
              controller: emailController,
              decoration: const InputDecoration(labelText: 'Email'),
            ),
            TextField(
              controller: nomController,
              decoration: const InputDecoration(labelText: 'Nom'),
            ),
            TextField(
              controller: prenomController,
              decoration: const InputDecoration(labelText: 'Prénom'),
            ),
          ],
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('Annuler'),
          ),
          ElevatedButton(
            onPressed: () async {
              try {
                await ApiService.createItem('users', {
                  'email': emailController.text,
                  'nom': nomController.text,
                  'prenom': prenomController.text,
                });
                if (mounted) {
                  Navigator.pop(context);
                  await loadUsers();
                  ScaffoldMessenger.of(context).showSnackBar(
                    const SnackBar(content: Text('Utilisateur créé')),
                  );
                }
              } catch (e) {
                if (mounted) {
                  ScaffoldMessenger.of(context).showSnackBar(
                    SnackBar(content: Text('Erreur: $e')),
                  );
                }
              }
            },
            child: const Text('Créer'),
          ),
        ],
      ),
    );
  }

  Future<void> _showEditUserDialog(User user) async {
    final emailController = TextEditingController(text: user.email);
    final nomController = TextEditingController(text: user.nom);
    final prenomController = TextEditingController(text: user.prenom);

    await showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('Modifier l\'utilisateur'),
        content: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            TextField(
              controller: emailController,
              decoration: const InputDecoration(labelText: 'Email'),
            ),
            TextField(
              controller: nomController,
              decoration: const InputDecoration(labelText: 'Nom'),
            ),
            TextField(
              controller: prenomController,
              decoration: const InputDecoration(labelText: 'Prénom'),
            ),
          ],
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('Annuler'),
          ),
          ElevatedButton(
            onPressed: () async {
              try {
                await ApiService.updateItem('users', user.id, {
                  'email': emailController.text,
                  'nom': nomController.text,
                  'prenom': prenomController.text,
                });
                if (mounted) {
                  Navigator.pop(context);
                  await loadUsers();
                  ScaffoldMessenger.of(context).showSnackBar(
                    const SnackBar(content: Text('Utilisateur modifié')),
                  );
                }
              } catch (e) {
                if (mounted) {
                  ScaffoldMessenger.of(context).showSnackBar(
                    SnackBar(content: Text('Erreur: $e')),
                  );
                }
              }
            },
            child: const Text('Modifier'),
          ),
        ],
      ),
    );
  }

  Future<void> _deleteUser(int id) async {
    final confirmed = await showDialog<bool>(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('Confirmer la suppression'),
        content: const Text('Voulez-vous vraiment supprimer cet utilisateur ?'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context, false),
            child: const Text('Annuler'),
          ),
          ElevatedButton(
            onPressed: () => Navigator.pop(context, true),
            style: ElevatedButton.styleFrom(backgroundColor: Colors.red),
            child: const Text('Supprimer'),
          ),
        ],
      ),
    );

    if (confirmed == true) {
      try {
        await ApiService.deleteItem('users', id);
        await loadUsers();
        if (mounted) {
          ScaffoldMessenger.of(context).showSnackBar(
            const SnackBar(content: Text('Utilisateur supprimé')),
          );
        }
      } catch (e) {
        if (mounted) {
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(content: Text('Erreur: $e')),
          );
        }
      }
    }
  }
}
```

## ⚙️ Étape 4 : Configuration CORS dans Symfony

Pour permettre à Flutter de communiquer avec votre API Symfony, configurez CORS.

### Installer le Bundle CORS

```bash
# Dans votre projet Symfony
docker compose exec php composer require nelmio/cors-bundle
```

### Configurer CORS

Modifier `config/packages/nelmio_cors.yaml` :

```yaml
nelmio_cors:
    defaults:
        origin_regex: true
        allow_origin: ['*']  # En production, spécifier les domaines autorisés
        allow_methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'OPTIONS']
        allow_headers: ['Content-Type', 'Authorization', 'Accept']
        expose_headers: ['Link']
        max_age: 3600
    paths:
        '^/api': ~
```

**⚠️ En production**, spécifiez les domaines autorisés :

```yaml
nelmio_cors:
    defaults:
        origin_regex: true
        allow_origin: ['https://monapp.com', 'https://www.monapp.com']
        allow_methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'OPTIONS']
        allow_headers: ['Content-Type', 'Authorization', 'Accept']
        expose_headers: ['Link']
        max_age: 3600
    paths:
        '^/api': ~
```

## 🔐 Gestion de l'Authentification JWT

### Configuration Flutter

```dart
class ApiService {
  static String? _token;

  static void setToken(String token) {
    _token = token;
  }

  static void clearToken() {
    _token = null;
  }

  static Map<String, String> get headers => {
    'Content-Type': 'application/json',
    'Accept': 'application/json',
    if (_token != null) 'Authorization': 'Bearer $_token',
  };

  // Login
  static Future<String> login(String email, String password) async {
    final response = await http.post(
      Uri.parse('$baseUrl/login'),
      headers: {'Content-Type': 'application/json'},
      body: json.encode({'email': email, 'password': password}),
    );

    if (response.statusCode == 200) {
      final data = json.decode(response.body);
      final token = data['token'];
      setToken(token);
      
      // Sauvegarder dans le stockage sécurisé
      final storage = FlutterSecureStorage();
      await storage.write(key: 'jwt_token', value: token);
      
      return token;
    } else {
      throw Exception('Erreur de connexion');
    }
  }

  // Logout
  static Future<void> logout() async {
    clearToken();
    final storage = FlutterSecureStorage();
    await storage.delete(key: 'jwt_token');
  }
}
```

## 📊 Gestion des Erreurs HTTP

```dart
class ApiException implements Exception {
  final int statusCode;
  final String message;

  ApiException(this.statusCode, this.message);

  @override
  String toString() => 'ApiException: $statusCode - $message';
}

// Dans ApiService
static Future<List<dynamic>> getItems(String endpoint) async {
  try {
    final response = await http.get(
      Uri.parse('$baseUrl/$endpoint'),
      headers: headers,
    );

    switch (response.statusCode) {
      case 200:
        return json.decode(response.body);
      case 401:
        throw ApiException(401, 'Non autorisé - Token invalide');
      case 403:
        throw ApiException(403, 'Accès interdit');
      case 404:
        throw ApiException(404, 'Ressource non trouvée');
      case 500:
        throw ApiException(500, 'Erreur serveur');
      default:
        throw ApiException(
          response.statusCode,
          'Erreur ${response.statusCode}: ${response.body}',
        );
    }
  } on SocketException {
    throw ApiException(0, 'Pas de connexion Internet');
  } on TimeoutException {
    throw ApiException(0, 'Délai d\'attente dépassé');
  } catch (e) {
    throw ApiException(0, 'Erreur inconnue: $e');
  }
}
```

## 🎯 Bonnes Pratiques

### 1. Utiliser un Service Centralisé
✅ Créer un seul service API pour toute l'application

### 2. Gérer les Timeouts
```dart
final response = await http.get(
  Uri.parse('$baseUrl/$endpoint'),
  headers: headers,
).timeout(const Duration(seconds: 10));
```

### 3. Logger les Requêtes (Développement)
```dart
print('Request: GET $baseUrl/$endpoint');
print('Headers: $headers');
print('Response: ${response.statusCode} ${response.body}');
```

### 4. Utiliser des Constantes
```dart
class ApiConstants {
  static const String baseUrl = 'https://localhost/api';
  static const Duration timeout = Duration(seconds: 10);
}
```

## 📚 Fichiers de Documentation Associés

- **[INSTALLATION.md](INSTALLATION.md)** - Installation de Flutter
- **[PACKAGES-DEPENDANCES.md](PACKAGES-DEPENDANCES.md)** - Gestion des packages
- **[COMMANDES-UTILES.md](COMMANDES-UTILES.md)** - Commandes Flutter courantes
- **[DEPANNAGE.md](DEPANNAGE.md)** - Problèmes courants et solutions
