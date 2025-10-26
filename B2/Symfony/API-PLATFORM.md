# 🚀 Installation et Configuration d'API Platform

**Ce guide vous explique comment installer et configurer API Platform pour créer des APIs REST modernes avec Symfony.**

## 📋 Qu'est-ce qu'API Platform ?

API Platform est un framework puissant pour créer des APIs REST et GraphQL avec Symfony. Il permet de :
- ✅ **Générer automatiquement** des APIs REST à partir des entités Doctrine
- ✅ **Documentation automatique** avec Swagger UI et ReDoc
- ✅ **Validation** et sérialisation automatiques
- ✅ **Pagination, filtrage et tri** prêts à l'emploi
- ✅ **Support JSON-LD, HAL, JSON:API**
- ✅ **Interface d'administration** pour tester les APIs

## ⚠️ PRÉREQUIS

**AVANT DE COMMENCER :**
- ✅ **Symfony Docker** doit être installé et fonctionnel (voir [INSTALLATION.md](INSTALLATION.md))
- ✅ **Containers Docker** en cours d'exécution (`docker compose up -d`)
- ✅ **Entités Doctrine** existantes (optionnel mais recommandé)

## 🎯 INSTALLATION ÉTAPE PAR ÉTAPE

### Étape 1 : Vérifier que les Containers Fonctionnent

```powershell
# Vérifier l'état des containers
docker compose ps

# Si les containers ne sont pas démarrés
docker compose up -d
```

### Étape 2 : Installer API Platform Core

```powershell
# Installation via Composer (DANS le container PHP)
docker compose exec php composer require api-platform/core
```

**⏱️ Temps d'installation :** 2-3 minutes selon votre connexion

**✅ Résultat attendu :**
- Installation des dépendances API Platform
- Configuration automatique via Symfony Flex
- Création du fichier `config/packages/api_platform.yaml`
- Enregistrement du bundle dans `config/bundles.php`

### Étape 3 : Vérifier l'Installation

```powershell
# Vérifier que le package est installé
docker compose exec php composer show api-platform/core

# Vérifier les routes API Platform
docker compose exec php bin/console debug:router | findstr api
```

**✅ Routes créées automatiquement :**
- `/api` - Point d'entrée de l'API
- `/api/docs.{_format}` - Documentation de l'API
- `/api/contexts/{shortName}.{_format}` - Contextes JSON-LD

## 🌐 ACCÉDER À L'API

### 1. **Point d'Entrée de l'API**
- 🔗 **URL :** [https://localhost/api](https://localhost/api)
- **Format :** JSON-LD par défaut
- **Description :** Point d'entrée principal de votre API

### 2. **Documentation Interactive (Swagger UI)**
- 🔗 **URL :** [https://localhost/api/docs](https://localhost/api/docs)
- **Description :** Interface graphique pour tester vos APIs
- **Fonctionnalités :** Tests en direct, exemples de requêtes

### 3. **Documentation Alternative (ReDoc)**
- 🔗 **URL :** [https://localhost/api/docs?ui=re_doc](https://localhost/api/docs?ui=re_doc)
- **Description :** Documentation plus lisible et détaillée

## ⚙️ CONFIGURATION PAR DÉFAUT

Le fichier `config/packages/api_platform.yaml` est créé avec :

```yaml
api_platform:
    title: Hello API Platform
    version: 1.0.0
    defaults:
        stateless: true
        cache_headers:
            vary: ['Content-Type', 'Authorization', 'Origin']
```

### Personnalisation de la Configuration

```yaml
api_platform:
    title: Mon API Symfony
    version: 2.0.0
    description: 'API pour mon application Symfony'
    defaults:
        stateless: true
        cache_headers:
            vary: ['Content-Type', 'Authorization', 'Origin']
        # Pagination par défaut
        pagination_items_per_page: 10
        # Formats supportés
        formats:
            jsonld: ['application/ld+json']
            json: ['application/json']
            html: ['text/html']
```

## 🏗️ CRÉER VOS PREMIÈRES API RESOURCES

> **⚠️ NOTE :** Les exemples ci-dessous utilisent une entité `Product` générique. Adaptez selon vos besoins.

### Transformer une Entité Existante en API Resource

Ajoutez simplement l'attribut `#[ApiResource]` à vos entités :

```php
<?php
// src/Entity/Product.php

use ApiPlatform\Metadata\ApiResource;
use Doctrine\ORM\Mapping as ORM;

#[ORM\Entity]
#[ApiResource] // ← Ajouter cette ligne
class Product
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    private ?int $id = null;

    #[ORM\Column(length: 255)]
    private ?string $name = null;

    // Getters et Setters...
}
```

### Créer une Nouvelle Entité avec API

```powershell
# 1. Générer une entité avec MakerBundle
docker compose exec php bin/console make:entity Product

# 2. Ajouter #[ApiResource] dans le fichier src/Entity/Product.php généré
```

## 🧪 TESTER VOS APIs

### Via l'Interface Swagger (Recommandé)
1. Accéder à [https://localhost/api/docs](https://localhost/api/docs)
2. Développer une ressource (ex: Product)
3. Cliquer sur "Try it out"
4. Exécuter la requête

### Via curl (Terminal)

```powershell
# Récupérer tous les produits
curl -X GET "https://localhost/api/products" -H "accept: application/ld+json" -k

# Créer un nouveau produit
curl -X POST "https://localhost/api/products" -H "Content-Type: application/ld+json" -d '{"name": "Mon Produit"}' -k
```

### Via PowerShell (Windows)

```powershell
# Récupérer tous les produits
Invoke-RestMethod -Uri "https://localhost/api/products" -Method GET -SkipCertificateCheck

# Créer un produit
$body = @{name = "Mon Produit"} | ConvertTo-Json
Invoke-RestMethod -Uri "https://localhost/api/products" -Method POST -Headers @{"Content-Type"="application/ld+json"} -Body $body -SkipCertificateCheck
```

## 🔧 CONFIGURATION AVANCÉE (Optionnel)

### Personnaliser les Opérations

```php
use ApiPlatform\Metadata\ApiResource;
use ApiPlatform\Metadata\Get;
use ApiPlatform\Metadata\GetCollection;
use ApiPlatform\Metadata\Post;

#[ApiResource(
    operations: [
        new GetCollection(),  // GET /api/products
        new Get(),            // GET /api/products/{id}
        new Post()            // POST /api/products
        // Pas de PUT ni DELETE
    ]
)]
class Product { }
```

### Ajouter de la Validation

```php
use Symfony\Component\Validator\Constraints as Assert;

class Product
{
    #[ORM\Column(length: 255)]
    #[Assert\NotBlank(message: "Le nom est obligatoire")]
    #[Assert\Length(min: 2, max: 255)]
    private ?string $name = null;
}
```

### Configurer la Pagination

Dans `config/packages/api_platform.yaml` :

```yaml
api_platform:
    defaults:
        pagination_items_per_page: 10  # Nombre d'items par page
```

## 🛠️ COMMANDES UTILES

```powershell
# Vérifier les routes API
docker compose exec php bin/console debug:router | findstr api

# Nettoyer le cache après modifications
docker compose exec php bin/console cache:clear

# Exporter la documentation OpenAPI
docker compose exec php bin/console api:openapi:export
```

## 🚨 PROBLÈMES COURANTS

### "No API resources found"
Vérifiez que vos entités ont l'attribut `#[ApiResource]`

### "Documentation non accessible"
```powershell
# Vérifier les routes
docker compose exec php bin/console debug:router | findstr api_doc
```

### "Certificat SSL non valide"
Acceptez le certificat auto-signé dans votre navigateur ou utilisez `-k` avec curl

Pour plus de problèmes, consultez **[DEPANNAGE.md](DEPANNAGE.md)**

## 📚 RESSOURCES COMPLÉMENTAIRES

- **Documentation officielle :** [API Platform Docs](https://api-platform.com/docs/)
- **[ENTITES.md](ENTITES.md)** - Créer et gérer vos entités Doctrine
- **[DONNEES-BDD.md](DONNEES-BDD.md)** - Manipuler les données

## ✅ RÉSUMÉ

Vous avez maintenant :
- ✅ API REST fonctionnelle avec documentation automatique
- ✅ Interface de test intégrée (Swagger UI)
- ✅ Validation et sérialisation automatiques
- ✅ Standards modernes (JSON-LD, OpenAPI)

**Prochaines étapes :** Ajoutez `#[ApiResource]` à vos entités et testez via `/api/docs` !