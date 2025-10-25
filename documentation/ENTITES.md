# 🏗️ Guide de Gestion des Entités

Guide simplifié pour créer et gérer les entités Doctrine dans le projet Symfony Docker.

> **⚠️ IMPORTANT :** Les entités `Game` et `Platform` mentionnées dans ce guide sont des **EXEMPLES SEULEMENT**. L'installation de base du template officiel ne contient aucune entité. Vous devez créer vos propres entités selon vos besoins.

**🔗 Guides Connexes :**
- [AJOUT-ADMINER.md](AJOUT-ADMINER.md) - Installation PostgreSQL + Adminer
- [DONNEES-BDD.md](DONNEES-BDD.md) - Manipuler les données via commandes
- [DEPANNAGE.md](DEPANNAGE.md) - Solutions aux problèmes courants

> **🔧 PRÉREQUIS :** Ce guide nécessite d'avoir installé PostgreSQL et Adminer en suivant [AJOUT-ADMINER.md](AJOUT-ADMINER.md).

## 🚀 DÉMARRAGE RAPIDE

### Étape 1 : Installer les Outils Nécessaires

```powershell
# Installer le MakerBundle (indispensable pour créer des entités)
docker compose exec php composer require symfony/maker-bundle --dev

# Vérifier que les commandes sont disponibles
docker compose exec php bin/console list make
```

### Étape 2 : Créer votre Première Entité

```powershell
# Créer votre première entité (exemple : Product)
docker compose exec php bin/console make:entity Product

# Suivre les instructions pour ajouter des propriétés :
# - name (string, 255)
# - price (float, nullable)
# - description (text, nullable)
```


### Étape 3 : Générer les Migrations

```powershell
# Créer la migration après création/modification d'entités
docker compose exec php bin/console make:migration

# Appliquer les migrations à la base de données
docker compose exec php bin/console doctrine:migrations:migrate
```

**⚠️ IMPORTANT :** Faire une sauvegarde de la base de données avant une migration car cela peut vider les tables de leur contenu.

**💡 ASTUCE :** Une fois vos entités créées, vous pouvez facilement ajouter des données via les commandes. Consultez [DONNEES-BDD.md](DONNEES-BDD.md) pour toutes les méthodes disponibles.

## ️ Gestion des Entités

### Ajouter des Champs à une Entité Existante

```powershell
# Ajouter des champs à l'entité Product existante
docker compose exec php bin/console make:entity Product
# Entrer les noms et types de propriétés quand demandé
```

### Créer des Relations - Guide Étape par Étape

**Exemple complet :** Système de gestion de produits avec toutes les relations

#### Étape 1 : Créer les entités de base

```powershell
# Créer l'entité Category (indépendante)
docker compose exec php bin/console make:entity Category
# Propriétés : name (string, 255), description (text, nullable)

# Créer l'entité Tag (indépendante)  
docker compose exec php bin/console make:entity Tag
# Propriétés : name (string, 100), color (string, 7, nullable)

# Créer l'entité ProductDetail (indépendante)
docker compose exec php bin/console make:entity ProductDetail
# Propriétés : specifications (text, nullable), warranty (string, 100, nullable)

# L'entité Product existe déjà avec : name (string), price (float, nullable)
```

#### Étape 2 : Ajouter les relations à Product

```powershell
# Ajouter toutes les relations dans Product
docker compose exec php bin/console make:entity Product
```

**Dialogue 1 - Relation ManyToOne (Product → Category) :**
```
New property name: category
Field type: relation
What class should this entity be related to? Category
Relation type: ManyToOne
Is the Product.category property allowed to be null? no
Do you want to add a new property to Category so that you can access/update Product objects from it? yes
New field name inside Category: products
```

**Dialogue 2 - Relation ManyToMany (Product ↔ Tags) :**
```
New property name: tags
Field type: relation  
What class should this entity be related to? Tag
Relation type: ManyToMany
Do you want to add a new property to Tag so that you can access/update Product objects from it? yes
New field name inside Tag: products
```

**Dialogue 3 - Relation OneToOne (Product ↔ ProductDetail) :**
```
New property name: detail
Field type: relation
What class should this entity be related to? ProductDetail
Relation type: OneToOne
Is the Product.detail property allowed to be null? yes
Do you want to add a new property to ProductDetail so that you can access/update Product objects from it? yes
New field name inside ProductDetail: product
```

#### Étape 3 : Finaliser

```powershell
# Créer la migration pour toutes les relations
docker compose exec php bin/console make:migration

# Appliquer la migration
docker compose exec php bin/console doctrine:migrations:migrate

# Régénérer les getters/setters si nécessaire
docker compose exec php bin/console make:entity --regenerate
```

#### 🔍 Comprendre les Types de Relations

| Type | Signification | Exemple concret | Table créée |
|------|---------------|-----------------|-------------|
| **ManyToOne** | Plusieurs → Un | Un produit a UNE catégorie | Colonne `category_id` dans `product` |
| **OneToMany** | Un → Plusieurs | Une catégorie a PLUSIEURS produits | Côté inverse de ManyToOne |
| **ManyToMany** | Plusieurs ↔ Plusieurs | Un produit a PLUSIEURS tags, un tag sur PLUSIEURS produits | Table `product_tag` |
| **OneToOne** | Un ↔ Un | Un produit a UN détail unique | Colonne `product_detail_id` dans `product` |


#### ⚠️ Règles Importantes

**1. Ordre de création :**
1. Entités "indépendantes" d'abord (Category, Tag, ProductDetail)
2. Entité avec relations ensuite (Product)

**2. Vérification après création :**
```powershell
docker compose exec php bin/console doctrine:schema:validate
```

### Régénérer les Getters/Setters

```powershell
# Régénérer les getters/setters manquants
docker compose exec php bin/console make:entity --regenerate
```

## 📊 Gestion de la Base de Données

### Vérifier et Valider

```powershell
# Vérifier la validité du mapping d'entité
docker compose exec php bin/console doctrine:schema:validate

# Obtenir des informations sur les entités
docker compose exec php bin/console doctrine:mapping:info
```

### Mise à Jour Directe (Développement)

```powershell
# Pour le développement rapide - mettre à jour le schéma directement
docker compose exec php bin/console doctrine:schema:update --force

# Prévisualiser les changements sans les appliquer
docker compose exec php bin/console doctrine:schema:update --dump-sql
```


## 🛑 Problèmes Courants

### "There are no commands defined in the 'make' namespace"
```powershell
# Solution : Installer le MakerBundle
docker compose exec php composer require symfony/maker-bundle --dev
```

### Erreurs de connexion base de données
```powershell
# Vérifier que PostgreSQL fonctionne
docker compose ps

# Vérifier la configuration DATABASE_URL dans .env
# Elle doit contenir : DATABASE_URL="postgresql://app:!ChangeMe!@database:5432/app..."
```

### Erreurs de Migration
```powershell
# Vider le cache et régénérer
docker compose exec php bin/console cache:clear
docker compose exec php bin/console make:migration
```

---

**� Guides Connexes :**
- [AJOUT-ADMINER.md](AJOUT-ADMINER.md) - Installation PostgreSQL + Adminer
- [DEPANNAGE.md](DEPANNAGE.md) - Solutions aux problèmes courants