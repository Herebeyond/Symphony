# 🛠️ Guide de Dépannage - Symfony Docker

Ce guide résout **TOUS** les problèmes courants d'installation et d'utilisation.

> **ℹ️ NOTE IMPORTANTE :** Ce guide contient des exemples avec des entités `Game` et `Platform`. Ces entités ne sont **PAS** incluses dans l'installation de base. Elles sont utilisées comme exemples pour illustrer les commandes et troubleshooting. Remplacez `Game`/`Platform` par vos propres entités selon vos besoins.

## 🚨 PROBLÈMES D'INSTALLATION

### 1. "Port 80 already in use" / "Port is already allocated"

**CAUSE :** Un autre projet Docker utilise les mêmes ports.

**SOLUTION IMMÉDIATE :**
```powershell
# Arrêter TOUS les containers Docker
docker stop $(docker ps -aq)

# Relancer le projet
docker compose up -d --wait
```

### 2. "Container is unhealthy" / Conteneur PHP ne démarre pas

**CAUSE :** Problèmes de migration base de données ou volumes persistants.

**SOLUTION :**
```powershell
# Nettoyer complètement l'environnement
docker compose down --volumes
docker system prune -f --volumes

# Reconstruire et redémarrer
docker compose build --no-cache
docker compose up -d --wait
```

### 3. "Table already exists" (Erreur Migration)

**CAUSE :** Base de données contient déjà des tables d'une installation précédente.

**SOLUTION :**
```powershell
docker compose down --volumes
docker volume rm $(docker volume ls -q)
docker compose up -d --wait
```

### 4. Installation très lente (5-7 minutes)

**CAUSE NORMALE :** La première construction Docker prend du temps.

**QUE FAIRE :**
- ✅ **PATIENTER** - Ne fermez pas la fenêtre
- ✅ **Vérifier progression** - Les messages défilent
- ❌ **NE PAS INTERROMPRE** - Cela corromprait l'installation

## 🌐 PROBLÈMES D'ACCÈS WEB

### 5. "This site can't be reached" sur http://localhost

**SOLUTIONS :**
1. **Accepter le certificat HTTPS** - Cliquer "Avancé" puis "Continuer"
2. **Tester http://localhost:8080** (Adminer) d'abord
3. **Vérifier Docker :** `docker compose ps`

### 8. Base de données vide / Pas de données

**VÉRIFICATION :** 
```powershell
# Les données sont normalement déjà incluses
docker compose exec php bin/console app:inspect-entities
```

**Si vraiment vide :**
```powershell
# Ajouter des jeux manuellement
docker compose exec php bin/console app:add-game "Nom du Jeu"
docker compose exec php bin/console app:add-platform "Nom de la Plateforme"
```

## 🗄️ PROBLÈMES BASE DE DONNÉES

### 6. "could not find driver" - Extension PostgreSQL manquante

**CAUSE :** L'extension PHP `pdo_pgsql` n'est pas installée dans le conteneur.

**SYMPTÔMES :**
- Erreur : `could not find driver`
- Conteneur PHP avec statut "unhealthy"
- Échec de connexion PostgreSQL

**SOLUTION :**
```powershell
# Installer l'extension PostgreSQL
docker compose exec php install-php-extensions pdo_pgsql

# Redémarrer le conteneur PHP pour appliquer
docker compose restart php

# Vérifier que l'extension est chargée
docker compose exec php php -m | findstr pdo_pgsql
```

### 7. "SQLSTATE[HY000] [2002] Connection refused" - Mauvaise configuration

**CAUSE :** Le fichier `.env` utilise `127.0.0.1` au lieu du nom de service Docker.

**SYMPTÔMES :**
- Impossible de se connecter à PostgreSQL
- Erreurs de connexion dans les logs

**SOLUTION :**
```powershell
# Éditer le fichier .env et remplacer :
# DATABASE_URL="postgresql://app:!ChangeMe!@127.0.0.1:5432/app..."
# par :
# DATABASE_URL="postgresql://app:!ChangeMe!@database:5432/app..."

# Puis redémarrer
docker compose restart php
```

### 8. Adminer : "Cannot connect to database"

**Paramètres corrects :**
- **Système :** PostgreSQL  
- **Serveur :** `database` (PAS `localhost`)
- **Utilisateur :** `app`
- **Mot de passe :** `!ChangeMe!`
- **Base de données :** `app`

**Si la connexion échoue :**
```powershell
# Vérifier que la base de données est prête
docker compose logs database

# Tester la connexion directement
docker compose exec php bin/console dbal:run-sql "SELECT 1"
```

## 🔧 PROBLÈMES SPÉCIFIQUES À ADMINER

### "service adminer depends on undefined service database"

**CAUSE :** Doctrine ORM n'est pas installé (le service `database` n'existe pas).

**SOLUTION :**
```powershell
# Installer Doctrine ORM (ajoute automatiquement PostgreSQL)
docker compose up -d php --no-deps
docker compose exec php composer require symfony/orm-pack
docker compose down
```

### "Port 8080 already in use"

**CAUSE :** Un autre service utilise déjà le port 8080.

**SOLUTION :**
```powershell
# Modifier le port d'Adminer dans compose.override.yaml :
# ports:
#   - target: 8080
#     published: 8081  # Changer ici
#     protocol: tcp

# Puis redémarrer
docker compose down
docker compose up -d --wait
```

### Adminer : Interface vide ou erreur au chargement

**SOLUTION :**
```powershell
# Vérifier les logs d'Adminer
docker compose logs adminer

# Redémarrer le service
docker compose restart adminer
```

## 🗄️ PROBLÈMES DE DONNÉES

### 9. Base de données vide / Pas de données

```powershell
# 1. Tout arrêter et nettoyer
docker stop $(docker ps -aq)
docker system prune -a -f --volumes

# 2. Recommencer l'installation
docker compose build --no-cache
docker compose up -d --wait
docker compose exec php bin/console app:insert-game-data --clear
```

---

**💡 90% des problèmes viennent de ne pas suivre exactement les étapes !**

## 🚨 Diagnostic Rapide

### Vérifier le Statut du Système

```powershell
# Vérifier tous les conteneurs
docker compose ps

# Vérifier la santé des conteneurs
docker compose ps --format "table {{.Name}}\t{{.Status}}\t{{.Ports}}"

# Voir les journaux pour les problèmes
docker compose logs --tail=50 php
docker compose logs --tail=50 database
```

### Vérification des Exigences Système

```powershell
# Vérifier la version de Docker
docker --version
docker compose version

# Vérifier l'espace disque disponible
docker system df

# Vérifier les conteneurs en cours d'exécution
docker ps -a
```

## 🐳 Problèmes Docker

### Le Conteneur ne Démarre Pas

**Symptômes :**
- `docker compose up` échoue
- Les conteneurs affichent le statut "Exited"
- Erreurs de liaison de port

**Solutions :**

```powershell
# Arrêter tous les conteneurs et nettoyer
docker compose down --remove-orphans

# Nettoyer les volumes si nécessaire
docker compose down --volumes

# Reconstruire les images à partir de zéro
docker compose build --pull --no-cache

# Démarrer avec une sortie détaillée
docker compose up --wait
```

### Port Déjà Utilisé

**Erreur :** `Port 80 is already allocated`

**Solutions :**

```powershell
# Trouver le processus utilisant le port
netstat -ano | findstr :80
netstat -ano | findstr :443

# Tuer le processus utilisant le port (remplacer PID)
taskkill /PID <PID> /F

# Ou changer les ports dans compose.yaml
# HTTP_PORT=8080 HTTPS_PORT=8443
```

### Manque d'Espace Disque

**Erreur :** `no space left on device`

**Solutions :**

```powershell
# Nettoyer le système Docker
docker system prune -f

# Supprimer les images inutilisées
docker image prune -a -f

# Supprimer les volumes inutilisés
docker volume prune -f

# Nettoyer le cache de construction
docker builder prune -f
```

## 🗄️ Problèmes de Base de Données

### Connexion Refusée

**Erreur :** `Connection refused` ou `SQLSTATE[HY000] [2002]`

**Étapes de Diagnostic :**

```powershell
# Vérifier si le conteneur de base de données fonctionne
docker compose ps database

# Vérifier les journaux de la base de données
docker compose logs database

# Tester la connexion à la base de données
docker compose exec php bin/console dbal:run-sql "SELECT 1"

# Vérifier la santé du conteneur de base de données
docker compose exec database mysql -u root -p -e "SELECT 1"
```

**Solutions :**

```powershell
# Redémarrer le service de base de données
docker compose restart database

# Attendre que la base de données soit prête
docker compose up database --wait

# Vérifier la chaîne de connexion dans .env
docker compose exec php cat .env
```

### Problèmes de Migration

**Erreur :** `Table 'app.game' doesn't exist` ou `Table already exists`

**Solutions :**

```powershell
# Option 1 : Réinitialiser les migrations
docker compose exec php bin/console doctrine:migrations:version --add --all --no-interaction

# Option 2 : Supprimer et recréer la base de données
docker compose exec php bin/console doctrine:database:drop --force --if-exists
docker compose exec php bin/console doctrine:database:create
docker compose exec php bin/console doctrine:migrations:migrate --no-interaction

# Option 3 : Réinitialiser les volumes (option nucléaire)
docker compose down --volumes
docker compose up -d --wait
docker compose exec php bin/console doctrine:database:create
docker compose exec php bin/console doctrine:migrations:migrate --no-interaction
```

### Permission Refusée

**Erreur :** `Access denied for user 'Nox'@'%'`

**Solutions :**

```powershell
# Utiliser l'utilisateur root temporairement
DATABASE_URL="mysql://root:root@database:3306/app?serverVersion=8.0&charset=utf8mb4"

# Ou recréer l'utilisateur dans MySQL
docker compose exec database mysql -u root -p
```

```sql
-- Dans la console MySQL
DROP USER IF EXISTS 'Nox'@'%';
CREATE USER 'Nox'@'%' IDENTIFIED BY 'Aglaboule14';
GRANT ALL PRIVILEGES ON app.* TO 'Nox'@'%';
FLUSH PRIVILEGES;
```

## 🌐 Problèmes d'Accès Web

### Avertissements de Certificat HTTPS

**Problème :** Le navigateur affiche "Non sécurisé" ou erreur de certificat

**Solutions :**

1. **Accepter le certificat auto-signé :**
   - Cliquer sur "Avancé" dans le navigateur
   - Cliquer sur "Continuer vers localhost (non sécurisé)"

2. **Installer le certificat dans Windows :**
   ```powershell
   # Exporter le certificat depuis le navigateur
   # Importer dans le magasin de certificats Windows
   ```

3. **Utiliser HTTP à la place (développement uniquement) :**
   - Accéder à http://localhost à la place
   - Ou configurer les ports : `HTTP_PORT=80`

### 404 Non Trouvé

**Problème :** Les pages affichent 404 ou erreurs de routage

**Diagnostic :**

```powershell
# Vérifier les routes
docker compose exec php bin/console debug:router

# Vérifier les journaux du serveur web
docker compose logs php

# Tester la fonctionnalité PHP de base
docker compose exec php bin/console about
```

**Solutions :**

```powershell
# Vider le cache Symfony
docker compose exec php bin/console cache:clear

# Reconstruire les assets
docker compose exec php bin/console asset-map:compile

# Redémarrer le conteneur PHP
docker compose restart php
```

### Chargement Lent

**Problème :** Les pages se chargent très lentement

**Solutions :**

```powershell
# Vérifier les ressources des conteneurs
docker stats

# Optimiser les paramètres PHP
# Éditer frankenphp/conf.d/20-app.dev.ini
opcache.enable=1
opcache.memory_consumption=256

# Vider le cache
docker compose exec php bin/console cache:clear --env=prod
```

## ⚡ Problèmes de Performance

### Utilisation Élevée de la Mémoire

**Symptômes :**
- Les conteneurs utilisent trop de RAM
- Le système devient lent
- Erreurs de mémoire insuffisante

**Solutions :**

```powershell
# Vérifier l'utilisation de la mémoire
docker stats --no-stream

# Limiter la mémoire des conteneurs dans compose.yaml
services:
  php:
    deploy:
      resources:
        limits:
          memory: 1g

# Optimiser la mémoire PHP
# Dans frankenphp/conf.d/20-app.dev.ini
memory_limit = 512M
```

### Requêtes de Base de Données Lentes

**Symptômes :**
- Les pages se chargent lentement
- Timeouts de base de données

**Diagnostic :**

```powershell
# Vérifier les requêtes lentes
docker compose exec database mysql -u root -p -e "SHOW PROCESSLIST"

# Activer le journal des requêtes lentes
docker compose exec database mysql -u root -p -e "SET GLOBAL slow_query_log = 'ON'"
```

**Solutions :**

```sql
-- Ajouter des index de base de données
CREATE INDEX idx_game_name ON game(name);
CREATE INDEX idx_platform_name ON platform(name);

-- Optimiser les requêtes
ANALYZE TABLE game;
ANALYZE TABLE platform;
```

## 🔧 Problèmes de Commande

### Commande Non Trouvée

**Erreur :** `Command "app:insert-game-data" is not defined`

**Solutions :**

```powershell
# Vider le cache
docker compose exec php bin/console cache:clear

# Vérifier si la commande existe
docker compose exec php bin/console list app

# Reconstruire l'autoloader
docker compose exec php composer dump-autoload
```

### Erreurs de Permission dans les Commandes

**Erreur :** `Permission denied` lors de l'exécution de commandes

**Solutions :**

```powershell
# Vérifier les permissions de fichier
docker compose exec php ls -la src/Command/

# Corriger la propriété si nécessaire
docker compose exec php chown -R www-data:www-data /app

# Exécuter en tant que root si nécessaire
docker compose exec --user root php bin/console your-command
```

## 📝 Problèmes Spécifiques à Symfony

### Problèmes de Cache

**Symptômes :**
- Les changements ne sont pas reflétés
- Comportement étrange après les mises à jour

**Solutions :**

```powershell
# Vider tous les caches
docker compose exec php bin/console cache:clear

# Vider des caches spécifiques
docker compose exec php bin/console cache:clear --env=dev
docker compose exec php bin/console cache:clear --env=prod

# Préchauffer le cache
docker compose exec php bin/console cache:warmup
```

### Problèmes de Composer

**Erreur :** `SSL certificate problem` ou échec d'installation de package

**Solutions :**

```powershell
# Mettre à jour composer
docker compose exec php composer self-update

# Vider le cache de composer
docker compose exec php composer clear-cache

# Installer avec différentes options
docker compose exec php composer install --no-dev --optimize-autoloader

# Désactiver la vérification SSL (dernier recours)
docker compose exec php composer config -g secure-http false
```

### Problèmes de Configuration de Bundle

**Erreur :** Bundle non trouvé ou configuration invalide

**Solutions :**

```powershell
# Vérifier les bundles installés
docker compose exec php bin/console debug:container --show-private

# Régénérer la configuration de bundle
docker compose exec php composer recipes

# Vider le cache après les changements de configuration
docker compose exec php bin/console cache:clear
```

## 🚀 Problèmes de Développement

### Rechargement à Chaud Non Fonctionnel

**Problème :** Les changements de code ne sont pas reflétés immédiatement

**Solutions :**

```powershell
# Vérifier si le mode watch est activé
# Dans Dockerfile: ENV FRANKENPHP_WORKER_CONFIG=watch

# Redémarrer avec watch
docker compose down
docker compose up -d

# Vider le cache manuellement
docker compose exec php bin/console cache:clear
```

### Débogage Non Fonctionnel

**Problème :** Xdebug ou outils de débogage non fonctionnels

**Solutions :**

```powershell
# Vérifier si Xdebug est installé
docker compose exec php php -m | grep xdebug

# Activer Xdebug
docker compose exec php php -dxdebug.mode=debug script.php

# Vérifier la configuration Xdebug
docker compose exec php php -i | grep xdebug
```

## 🆘 Récupération d'Urgence

### Réinitialisation Complète

Quand tout se casse :

```powershell
# Option nucléaire - réinitialisation complète
docker compose down --volumes --remove-orphans
docker system prune -a -f
docker volume prune -f

# Tout reconstruire
docker compose build --pull --no-cache
docker compose up -d --wait

# Recréer la base de données et les données
docker compose exec php bin/console doctrine:database:create
docker compose exec php bin/console doctrine:migrations:migrate --no-interaction
docker compose exec php bin/console app:insert-game-data --clear
```

### Sauvegarde Avant Récupération

```powershell
# Sauvegarder la base de données avant les changements majeurs
docker compose exec php bin/console app:database:backup --name="before-reset"

# Exporter l'état du conteneur
docker commit symfony-docker-php-1 my-backup-image
```

## 📊 Surveillance et Journaux

### Commandes de Journal Essentielles

```powershell
# Journaux en temps réel
docker compose logs -f php
docker compose logs -f database

# N dernières lignes
docker compose logs --tail=100 php

# Journaux avec horodatage
docker compose logs -t php

# Journaux de tous les services
docker compose logs --tail=50
```

### Surveillance du Système

```powershell
# Utilisation des ressources des conteneurs
docker stats

# Utilisation du disque
docker system df

# Informations réseau
docker network ls
docker compose ps --format "table {{.Name}}\t{{.Status}}\t{{.Ports}}"
```

---

**📝 Documentation Connexe :**
- [INSTALLATION.md](INSTALLATION.md) - Guide de Démarrage Rapide
- [ENTITES.md](ENTITES.md) - Guide de Gestion des Entités
- [DONNEES-BDD.md](DONNEES-BDD.md) - Manipulation des données via commandes
- [AJOUT-ADMINER.md](AJOUT-ADMINER.md) - Installation PostgreSQL + Adminer