# 📊 Visualisation de Diagrammes de Base de Données avec dbdiagram.io

Ce guide explique comment visualiser votre base de données Symfony Docker en utilisant dbdiagram.io, un outil en ligne puissant pour créer des diagrammes de bases de données.

## 🎯 Qu'est-ce que dbdiagram.io ?

**dbdiagram.io** est un outil gratuit en ligne de diagramme de base de données qui vous permet de :
- ✅ Créer des diagrammes visuels de base de données à partir de DBML (Database Markup Language)
- ✅ Exporter les diagrammes en PDF, PNG, ou SQL
- ✅ Partager les diagrammes avec votre équipe
- ✅ Collaborer en temps réel

## 🚀 Démarrage Rapide

### Méthode 1 : Utiliser le DBML Auto-Généré (Recommandé)

```powershell
# Générer le DBML à partir de votre base de données actuelle
docker compose exec php npm run db-diagram

# Ceci va :
# 1. Se connecter à votre base de données PostgreSQL
# 2. Générer le fichier database.dbml
# 3. Afficher les instructions pour dbdiagram.io

# ✅ Les packages NPM sont pré-installés - pas besoin de npm install manuel !
```

### Méthode 2 : Utiliser le Fichier DBML Statique

Si vous préférez utiliser le fichier DBML pré-créé :

```powershell
# Le fichier database.dbml existe déjà dans la racine de votre projet
# Vous pouvez l'utiliser directement avec dbdiagram.io
```

## 📋 Instructions Étape par Étape

### 1. Générer ou Mettre à Jour le Fichier DBML

```powershell
# S'assurer que vos conteneurs Docker fonctionnent
docker compose up -d

# Générer un nouveau DBML depuis la base de données
docker compose exec php npm run generate-dbml
```

### 2. Copier le Contenu DBML

Ouvrez le fichier `database.dbml` généré et copiez tout son contenu.

### 3. Créer un Diagramme sur dbdiagram.io

1. **Allez sur [dbdiagram.io](https://dbdiagram.io)**
2. **Cliquez sur "Go to App" ou "Get Started"**
3. **Collez votre contenu DBML** dans le panneau d'édition de gauche
4. **Visualisez votre diagramme** dans le panneau de droite instantanément !

### 4. Personnaliser Votre Diagramme

Sur dbdiagram.io, vous pouvez :
- **Réorganiser les tables** en les faisant glisser
- **Zoomer** pour une meilleure visibilité
- **Changer les couleurs** et thèmes
- **Ajouter des notes** et de la documentation

### 5. Exporter Votre Diagramme

Cliquez sur le bouton **Export** pour télécharger au format :
- **PDF** - Pour la documentation
- **PNG** - Pour les présentations
- **SQL** - Pour la création de base de données
- **DBML** - Pour le partage

## 🛠️ Commandes NPM Disponibles

| Commande | Description |
|---------|-------------|
| `npm run generate-dbml` | Générer un fichier DBML à partir de la base de données |
| `npm run db-diagram` | Générer DBML + afficher les instructions |

### Exemple d'Utilisation

```powershell
# Génération rapide de diagramme de base de données
docker compose exec php npm run db-diagram

# Sortie :
# ✅ Fichier DBML généré avec succès : /app/database.dbml
# 
# 📋 Étapes suivantes :
# 1. Ouvrir https://dbdiagram.io
# 2. Copier le contenu de database.dbml  
# 3. Le coller dans l'éditeur dbdiagram.io
# 4. Visualiser votre diagramme de base de données !
```

## 📊 Ce qui est Inclus dans Votre Diagramme

Votre diagramme de base de données Symfony Docker inclut :

### 🏗️ Tables Principales
- **`game`** - Entrées du catalogue de jeux (exemple)
- **`platform`** - Plateformes de jeux (PlayStation, Xbox, PC, etc.) (exemple)
- **`game_platform`** - Table de jonction many-to-many (exemple)

> **Note :** Ces tables sont des exemples. Le template officiel aura une base de données vide.

### ⚙️ Tables Système
- **`doctrine_migration_versions`** - Suivi des migrations Doctrine
- **`messenger_messages`** - File d'attente Symfony Messenger

### 🔗 Relations
- **Game ↔ Platform** - Relation many-to-many via table de jonction (exemple)
- **Contraintes de clés étrangères** avec règles de suppression en cascade

## 🎨 Fonctionnalités DBML Expliquées

### Définitions de Tables
```dbml
Table game {
  id int [pk, increment]           // Clé primaire, auto-incrémentation
  name varchar(255) [not null]    // Champ requis
}
```

### Relations
```dbml
// Relation many-to-many
Ref: game_platform.game_id > game.id [delete: cascade]
Ref: game_platform.platform_id > platform.id [delete: cascade]
```

### Documentation du Projet
```dbml
Project symfony_docker_app {
  database_type: 'PostgreSQL'
  Note: '''
    Documentation et instructions d'utilisation de la base de données
  '''
}
```

## 🔄 Maintenir les Diagrammes à Jour

### Quand Régénérer le DBML

Exécutez le générateur après :
- ✅ **Ajout de nouvelles entités** à votre application Symfony
- ✅ **Création de nouvelles migrations** 
- ✅ **Modification des structures de tables**
- ✅ **Ajout/suppression de relations**

### Mises à Jour Automatisées

```powershell
# Ajoutez ceci à votre workflow de développement
# Après avoir exécuté les migrations :
docker compose exec php bin/console doctrine:migrations:migrate
docker compose exec php npm run db-diagram
```

## 🆘 Dépannage

### Problème : "Erreur de connexion à la base de données"

```powershell
# Vérifier que les conteneurs fonctionnent
docker compose ps

# Redémarrer si nécessaire
docker compose up -d

# Vérifier la connectivité de la base de données
docker compose exec php bin/console doctrine:query:sql "SELECT 1"
```

### Problème : "Fichier DBML introuvable"

```powershell
# Régénérer le fichier
docker compose exec php npm run generate-dbml

# Vérifier que le fichier existe
docker compose exec php ls -la database.dbml
```

### Problème : "Diagramme vide sur dbdiagram.io"

1. **Vérifier la syntaxe DBML** - Assurez-vous qu'il n'y a pas d'erreurs de syntaxe
2. **Copier le contenu complet** - Assurez-vous d'avoir copié tout le fichier
3. **Actualiser le navigateur** - Essayez un rafraîchissement forcé (Ctrl+F5)

## 💡 Conseils Pro

### 1. Sauvegarder Vos Diagrammes
- **Créer un compte** sur dbdiagram.io pour sauvegarder les diagrammes
- **Partager les URLs** avec votre équipe
- **Contrôle de version** - Garder les fichiers DBML dans Git

### 2. Intégration Documentation
```powershell
# Ajouter à votre README.md du projet
echo "## Diagramme de Base de Données" >> README.md
echo "Voir : [dbdiagram.io](https://dbdiagram.io) + database.dbml" >> README.md
```

### 3. Collaboration d'Équipe
- **Partager les fichiers DBML** via contrôle de version
- **Inclure les diagrammes** dans la documentation
- **Mettre à jour régulièrement** pendant le développement

### 4. Utilisation Avancée
```powershell
# Diagrammes spécifiques à l'environnement
$env:DB_HOST="production-host"; docker compose exec php npm run generate-dbml

# Sélection de base de données personnalisée
$env:DB_NAME="custom_db"; docker compose exec php npm run generate-dbml
```

## 🔗 Ressources Connexes

- **[Documentation dbdiagram.io](https://dbdiagram.io/docs)**
- **[Guide du Langage DBML](https://github.com/holistics/dbml)**
- **[Documentation Symfony Doctrine](https://symfony.com/doc/current/doctrine.html)**

## 📈 Exemple de Workflow

```powershell
# 1. Développer une nouvelle fonctionnalité
docker compose exec php bin/console make:entity Product
docker compose exec php bin/console make:migration
docker compose exec php bin/console doctrine:migrations:migrate

# 2. Mettre à jour le diagramme de base de données  
docker compose exec php npm run db-diagram

# 3. Créer un diagramme visuel
# - Aller sur dbdiagram.io
# - Coller le contenu de database.dbml
# - Exporter en PNG/PDF pour la documentation

# 4. Commiter les changements
git add database.dbml
git commit -m "feat: ajouter entité Product avec diagramme de base de données"
```

**🎉 You're Ready!** Your database is now visualized and ready for team collaboration and documentation!