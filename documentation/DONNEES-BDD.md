# 🗃️ Gestion des Données via Ligne de Commande

Guide complet pour ajouter, modifier, supprimer et consulter les données de votre base de données PostgreSQL via les commandes Symfony.

> **🔧 PRÉREQUIS :** Ce guide nécessite d'avoir installé PostgreSQL et Adminer en suivant [AJOUT-ADMINER.md](AJOUT-ADMINER.md) et créé des entités selon [ENTITES.md](ENTITES.md).

## 🚀 APERÇU DES MÉTHODES

| Méthode | Usage | Avantages | Inconvénients |
|---------|-------|-----------|---------------|
| **SQL Direct** | Requêtes brutes | Rapide, flexible | Pas de validation ORM |
| **DQL** | Requêtes via entités | Orienté objet, portable | Plus complexe |
| **Commandes Custom** | Scripts personnalisés | Réutilisable, logique métier | Temps de développement |
| **Fixtures** | Données de test | Reproductible, versionnable | Uniquement pour tests |

## 📊 1. COMMANDES SQL DIRECTES

### Consulter les Données

```powershell
# Lister tous les produits
docker compose exec php bin/console dbal:run-sql "SELECT * FROM product"

# Compter les produits par catégorie
docker compose exec php bin/console dbal:run-sql "SELECT c.name, COUNT(p.id) as nb_products FROM category c LEFT JOIN product p ON p.category_id = c.id GROUP BY c.name"

# Produits avec leurs catégories
docker compose exec php bin/console dbal:run-sql "SELECT p.name as product, c.name as category, p.price FROM product p JOIN category c ON p.category_id = c.id"
```

### Ajouter des Données

```powershell
# Ajouter une catégorie
docker compose exec php bin/console dbal:run-sql "INSERT INTO category (name, description) VALUES ('Électronique', 'Appareils électroniques')"

# Ajouter un produit (récupérer l'ID de catégorie d'abord)
docker compose exec php bin/console dbal:run-sql "SELECT id FROM category WHERE name = 'Électronique'"
docker compose exec php bin/console dbal:run-sql "INSERT INTO product (name, price, category_id) VALUES ('iPhone 15', 999.99, 1)"

# Ajouter plusieurs produits en une fois
docker compose exec php bin/console dbal:run-sql "INSERT INTO product (name, price, category_id) VALUES ('MacBook Pro', 2499.99, 1), ('iPad Air', 599.99, 1)"
```

### Modifier des Données

```powershell
# Modifier le prix d'un produit
docker compose exec php bin/console dbal:run-sql "UPDATE product SET price = 899.99 WHERE name = 'iPhone 15'"

# Modifier plusieurs produits d'une catégorie
docker compose exec php bin/console dbal:run-sql "UPDATE product SET price = price * 0.9 WHERE category_id = (SELECT id FROM category WHERE name = 'Électronique')"

# Changer la catégorie d'un produit
docker compose exec php bin/console dbal:run-sql "UPDATE product SET category_id = 2 WHERE name = 'iPhone 15'"
```

### Supprimer des Données

```powershell
# Supprimer un produit spécifique
docker compose exec php bin/console dbal:run-sql "DELETE FROM product WHERE name = 'iPhone 15'"

# Supprimer tous les produits d'une catégorie
docker compose exec php bin/console dbal:run-sql "DELETE FROM product WHERE category_id = (SELECT id FROM category WHERE name = 'Électronique')"

# Supprimer les produits sans catégorie
docker compose exec php bin/console dbal:run-sql "DELETE FROM product WHERE category_id IS NULL"
```

## 🎯 2. COMMANDES DQL (DOCTRINE QUERY LANGUAGE)

### Consulter avec DQL

```powershell
# Lister tous les produits (entités)
docker compose exec php bin/console doctrine:query:dql "SELECT p FROM App\Entity\Product p"

# Produits d'une catégorie spécifique
docker compose exec php bin/console doctrine:query:dql "SELECT p FROM App\Entity\Product p JOIN p.category c WHERE c.name = 'Électronique'"

# Produits avec prix entre deux valeurs
docker compose exec php bin/console doctrine:query:dql "SELECT p FROM App\Entity\Product p WHERE p.price BETWEEN 500 AND 1000"

# Compter les produits
docker compose exec php bin/console doctrine:query:dql "SELECT COUNT(p) FROM App\Entity\Product p"
```

### Avantages du DQL

- **Orienté entités** : Travaille avec vos classes PHP
- **Portable** : Fonctionne avec tous les SGBD supportés
- **Sécurisé** : Protection contre l'injection SQL
- **Relations automatiques** : Gestion des jointures facilitée

## 🛠️ 3. CRÉER DES COMMANDES PERSONNALISÉES

### Exemple 1 : Commande d'Ajout de Produit

```powershell
# Créer la commande
docker compose exec php bin/console make:command app:add-product
```

**Contenu du fichier `src/Command/AddProductCommand.php` :**

```php
<?php

namespace App\Command;

use App\Entity\Product;
use App\Entity\Category;
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Component\Console\Attribute\AsCommand;
use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputArgument;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
use Symfony\Component\Console\Style\SymfonyStyle;

#[AsCommand(
    name: 'app:add-product',
    description: 'Ajouter un nouveau produit avec sa catégorie',
)]
class AddProductCommand extends Command
{
    private EntityManagerInterface $entityManager;

    public function __construct(EntityManagerInterface $entityManager)
    {
        $this->entityManager = $entityManager;
        parent::__construct();
    }

    protected function configure(): void
    {
        $this
            ->addArgument('name', InputArgument::REQUIRED, 'Nom du produit')
            ->addArgument('price', InputArgument::REQUIRED, 'Prix du produit')
            ->addArgument('category', InputArgument::REQUIRED, 'Nom de la catégorie')
        ;
    }

    protected function execute(InputInterface $input, OutputInterface $output): int
    {
        $io = new SymfonyStyle($input, $output);
        
        $name = $input->getArgument('name');
        $price = (float) $input->getArgument('price');
        $categoryName = $input->getArgument('category');

        // Chercher ou créer la catégorie
        $category = $this->entityManager->getRepository(Category::class)
            ->findOneBy(['name' => $categoryName]);
        
        if (!$category) {
            $category = new Category();
            $category->setName($categoryName);
            $this->entityManager->persist($category);
        }

        // Créer le produit
        $product = new Product();
        $product->setName($name);
        $product->setPrice($price);
        $product->setCategory($category);

        $this->entityManager->persist($product);
        $this->entityManager->flush();

        $io->success("Produit '{$name}' ajouté avec succès dans la catégorie '{$categoryName}' !");

        return Command::SUCCESS;
    }
}
```

**Utilisation :**

```powershell
# Ajouter un produit avec la commande
docker compose exec php bin/console app:add-product "Samsung Galaxy S24" 899.99 "Smartphones"
docker compose exec php bin/console app:add-product "Dell XPS 13" 1299.99 "Ordinateurs"
```

### Exemple 2 : Commande de Listing

```powershell
# Créer la commande de listing
docker compose exec php bin/console make:command app:list-products
```

**Contenu simplifié :**

```php
protected function execute(InputInterface $input, OutputInterface $output): int
{
    $io = new SymfonyStyle($input, $output);
    
    $products = $this->entityManager->getRepository(Product::class)->findAll();
    
    $rows = [];
    foreach ($products as $product) {
        $rows[] = [
            $product->getId(),
            $product->getName(),
            $product->getPrice() . ' €',
            $product->getCategory() ? $product->getCategory()->getName() : 'Aucune'
        ];
    }
    
    $io->table(['ID', 'Nom', 'Prix', 'Catégorie'], $rows);
    
    return Command::SUCCESS;
}
```

**Utilisation :**

```powershell
docker compose exec php bin/console app:list-products
```

## 🎲 4. FIXTURES - DONNÉES DE TEST

### Installation des Fixtures

```powershell
# Installer le bundle fixtures
docker compose exec php composer require --dev orm-fixtures
```

### Créer des Fixtures

```powershell
# Générer une classe de fixtures
docker compose exec php bin/console make:fixtures ProductFixtures
```

**Contenu du fichier `src/DataFixtures/ProductFixtures.php` :**

```php
<?php

namespace App\DataFixtures;

use App\Entity\Product;
use App\Entity\Category;
use Doctrine\Bundle\FixturesBundle\Fixture;
use Doctrine\Persistence\ObjectManager;

class ProductFixtures extends Fixture
{
    public function load(ObjectManager $manager): void
    {
        // Créer des catégories
        $categories = [
            ['name' => 'Smartphones', 'description' => 'Téléphones intelligents'],
            ['name' => 'Ordinateurs', 'description' => 'Ordinateurs portables et fixes'],
            ['name' => 'Tablettes', 'description' => 'Tablettes tactiles'],
        ];

        $categoryObjects = [];
        foreach ($categories as $cat) {
            $category = new Category();
            $category->setName($cat['name']);
            $category->setDescription($cat['description']);
            $manager->persist($category);
            $categoryObjects[] = $category;
        }

        // Créer des produits
        $products = [
            ['iPhone 15', 999.99, 0], // Index de catégorie
            ['Samsung Galaxy S24', 899.99, 0],
            ['MacBook Pro M3', 2499.99, 1],
            ['Dell XPS 13', 1299.99, 1],
            ['iPad Pro', 1099.99, 2],
            ['Surface Pro', 1199.99, 2],
        ];

        foreach ($products as [$name, $price, $catIndex]) {
            $product = new Product();
            $product->setName($name);
            $product->setPrice($price);
            $product->setCategory($categoryObjects[$catIndex]);
            $manager->persist($product);
        }

        $manager->flush();
    }
}
```

### Charger les Fixtures

```powershell
# Charger toutes les fixtures (supprime les données existantes)
docker compose exec php bin/console doctrine:fixtures:load

# Charger sans demander confirmation
docker compose exec php bin/console doctrine:fixtures:load --no-interaction

# Ajouter aux données existantes (sans supprimer)
docker compose exec php bin/console doctrine:fixtures:load --append
```

## 🔍 5. COMMANDES D'INSPECTION

### Analyser la Structure

```powershell
# Voir toutes les entités
docker compose exec php bin/console doctrine:mapping:info

# Voir le schéma SQL actuel
docker compose exec php bin/console doctrine:schema:create --dump-sql

# Comparer avec la base de données
docker compose exec php bin/console doctrine:schema:update --dump-sql
```

### Statistiques Rapides

```powershell
# Compter les enregistrements de toutes les tables
docker compose exec php bin/console dbal:run-sql "SELECT 'product' as table_name, COUNT(*) as count FROM product UNION ALL SELECT 'category', COUNT(*) FROM category UNION ALL SELECT 'tag', COUNT(*) FROM tag"

# Voir les derniers produits ajoutés
docker compose exec php bin/console dbal:run-sql "SELECT name, price, created_at FROM product ORDER BY id DESC LIMIT 5"
```

## 🚨 6. COMMANDES DE MAINTENANCE

### Nettoyer les Données

```powershell
# Supprimer tous les produits
docker compose exec php bin/console dbal:run-sql "DELETE FROM product"

# Supprimer toutes les données (en respectant les contraintes)
docker compose exec php bin/console dbal:run-sql "DELETE FROM product; DELETE FROM category; DELETE FROM tag"

# Remettre à zéro les auto-increment
docker compose exec php bin/console dbal:run-sql "ALTER SEQUENCE product_id_seq RESTART WITH 1"
docker compose exec php bin/console dbal:run-sql "ALTER SEQUENCE category_id_seq RESTART WITH 1"
```

### Sauvegarder et Restaurer

```powershell
# Exporter les données vers un fichier SQL
docker compose exec database pg_dump -U app app > backup.sql

# Restaurer depuis un fichier SQL
docker compose exec -T database psql -U app app < backup.sql
```

## 🛑 CONSEILS ET BONNES PRATIQUES

### Sécurité

- **Toujours sauvegarder** avant des suppressions massives
- **Tester d'abord** avec `SELECT` avant `DELETE`
- **Utiliser des transactions** pour les opérations complexes

### Performance

- **Éviter les SELECT *** sur de grosses tables
- **Utiliser LIMIT** pour les tests
- **Préférer les commandes custom** pour les traitements répétitifs

### Organisation

```powershell
# Créer des commandes par domaine
docker compose exec php bin/console make:command app:product:add
docker compose exec php bin/console make:command app:product:list
docker compose exec php bin/console make:command app:category:create
```

---

**🔗 Guides Connexes :**
- [ENTITES.md](ENTITES.md) - Création et gestion des entités
- [AJOUT-ADMINER.md](AJOUT-ADMINER.md) - Interface graphique PostgreSQL
- [DEPANNAGE.md](DEPANNAGE.md) - Solutions aux problèmes courants