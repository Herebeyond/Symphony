# 🔐 Authentification JWT avec Symfony

**Ce guide explique comment mettre en place une authentification JWT (JSON Web Token) pour sécuriser votre API Symfony.**

## 📋 Qu'est-ce qu'un JWT ?

Un **JWT** (JSON Web Token) est un petit jeton de données au format JSON utilisé pour prouver l'identité d'un utilisateur sans devoir stocker de session sur le serveur.

### Structure d'un JWT

Un token JWT est divisé en **trois parties** séparées par des points :

```
xxxxx.yyyyy.zzzzz
```

1. **Header** : Indique le type de token et l'algorithme de signature
2. **Payload** : Contient les informations utiles (identifiant, rôle, etc.)
3. **Signature** : Permet de vérifier que le token n'a pas été modifié

### Avantages des JWT

- ✅ **Légers et rapides** - Pas de session serveur
- ✅ **Multi-plateforme** - Fonctionnent sur web, mobile, etc.
- ✅ **Sécurisés** - Avec HTTPS et clés bien configurées
- ✅ **Stateless** - Le serveur ne stocke rien

### Limites des JWT

- ⚠️ **Non révocable immédiatement** - Valide jusqu'à expiration
- ⚠️ **Risque de vol** - Si mal protégé côté client
- ⚠️ **Taille** - Plus volumineux qu'un simple ID de session

## 🔄 Fonctionnement Général

```
1. Client → Serveur : Envoi identifiants (email + password)
2. Serveur → Client : Retour token JWT signé
3. Client : Stockage sécurisé du token
4. Client → Serveur : Requêtes avec header "Authorization: Bearer <token>"
5. Serveur : Vérification token et autorisation accès
```

## ⚠️ PRÉREQUIS

**AVANT DE COMMENCER :**
- ✅ **Symfony Docker** installé et fonctionnel (voir [INSTALLATION.md](INSTALLATION.md))
- ✅ **Containers Docker** en cours d'exécution
- ✅ **PostgreSQL + Adminer** installés et démarrés (voir [AJOUT-ADMINER.md](AJOUT-ADMINER.md)) - **OBLIGATOIRE !**
- ✅ **Doctrine ORM** installé (installé automatiquement avec PostgreSQL)
- ✅ **MakerBundle** installé (sera vérifié à l'étape 0)

## 🎯 INSTALLATION ÉTAPE PAR ÉTAPE

### Étape 0 : Vérifier et Installer les Prérequis (OBLIGATOIRE)

**⚠️ CRITIQUE :** Avant de commencer, vous DEVEZ avoir suivi le guide **[AJOUT-ADMINER.md](AJOUT-ADMINER.md)** qui installe :
- ✅ PostgreSQL (base de données)
- ✅ Extension PHP `pdo_pgsql`
- ✅ Doctrine ORM
- ✅ Configuration correcte de `DATABASE_URL`
- ✅ Adminer (interface graphique - optionnel)

#### 0.1 : Vérification rapide

```powershell
# Vérifier que "database" ET "php" sont actifs
docker compose ps

# Test de connexion PostgreSQL
docker compose exec php bin/console dbal:run-sql "SELECT 1"
```

**Si ces commandes échouent, suivez d'abord [AJOUT-ADMINER.md](AJOUT-ADMINER.md).**

#### 0.2 : Installer MakerBundle (OBLIGATOIRE)

Le template de base ne contient pas le MakerBundle. Vérifions s'il est installé :

```powershell
# Vérifier si make:user existe
docker compose exec php bin/console list make
```

**❌ Si vous voyez l'erreur `There are no commands defined in the "make" namespace` :**

```powershell
# Installer le MakerBundle
docker compose exec php composer require symfony/maker-bundle --dev
```

**⏱️ Temps d'installation :** 1-2 minutes

**✅ Vérification après installation :**
```powershell
# Vérifier que make:user est disponible
docker compose exec php bin/console list make
```

**Résultat attendu :** Liste des commandes `make:*` incluant `make:user`

**Si la commande échoue toujours, consultez la section Dépannage en fin de guide.**

### Étape 1 : Installer le Bundle JWT

```powershell
# Installation du LexikJWTAuthenticationBundle
docker compose exec php composer require lexik/jwt-authentication-bundle
```

**⏱️ Temps d'installation :** 2-3 minutes

**✅ Résultat attendu :**
- Installation des dépendances JWT
- Configuration automatique via Symfony Flex
- Création du fichier `config/packages/lexik_jwt_authentication.yaml`
- Enregistrement du bundle dans `config/bundles.php`

### Étape 2 : Générer les Clés de Chiffrement

```powershell
# Générer les clés privée et publique
docker compose exec php php bin/console lexik:jwt:generate-keypair
```

**✅ Résultat attendu :**
- Création de `config/jwt/private.pem` (clé privée)
- Création de `config/jwt/public.pem` (clé publique)
- Ces clés permettent de signer et vérifier les tokens

**⚠️ IMPORTANT :** Ne JAMAIS commiter les clés dans Git ! Vérifiez que `config/jwt/` est dans `.gitignore`.

### Étape 3 : Créer l'Entité User AVANT de configurer le pare-feu

**⚠️ IMPORTANT :** Vous DEVEZ créer l'entité User AVANT de modifier `security.yaml`, sinon vous aurez une erreur.

```powershell
# Créer l'entité User avec le système de sécurité
docker compose exec php bin/console make:user
```

**Répondre aux questions :**
- Nom de la classe : `User`
- Stockage en base de données : `yes`
- Propriété pour identifier : `email`
- Hasher les mots de passe : `yes`

**✅ Résultat attendu :**
- Création de `src/Entity/User.php`
- Création de `src/Repository/UserRepository.php`
- **Modification automatique de `config/packages/security.yaml`**

**Puis générer et appliquer la migration :**

```powershell
# Créer la migration
docker compose exec php bin/console make:migration

# Appliquer la migration
docker compose exec php bin/console doctrine:migrations:migrate
```

### Étape 4 : Configurer le Pare-feu

**⚠️ IMPORTANT :** Ne copiez-collez PAS tout le fichier ! Suivez les instructions ci-dessous pour modifier uniquement les sections nécessaires.

Ouvrir le fichier `config/packages/security.yaml` et effectuer les modifications suivantes :

#### 4.1 - Vérifier la section `providers`

**La commande `make:user` devrait avoir créé automatiquement :**
```yaml
providers: 
    app_user_provider:
        entity:
            class: App\Entity\User
            property: email
```

**Si ce n'est pas le cas**, localiser la section `providers:` et modifier/ajouter ce contenu.

#### 4.2 - Ajouter deux nouveaux firewalls AVANT `main:`

**Localiser la section `firewalls:`** et **APRÈS le firewall `dev:`**, ajouter ces deux nouveaux firewalls :

```yaml
firewalls:
    dev:
        pattern: ^/(_(profiler|wdt)|css|images|js)/
        security: false
    
    # ⬇️ AJOUTER CES DEUX FIREWALLS ICI ⬇️
    login:
        pattern: ^/api/login
        stateless: true
        json_login:
            check_path: /api/login_check
            success_handler: lexik_jwt_authentication.handler.authentication_success
            failure_handler: lexik_jwt_authentication.handler.authentication_failure

    api:
        pattern: ^/api
        stateless: true
        jwt: ~
    # ⬆️ FIN DE L'AJOUT ⬆️
    
    main:
        lazy: true
        provider: app_user_provider  # ⬅️ Vérifier que c'est bien app_user_provider
```

#### 4.3 - Modifier la section `access_control`

**Localiser cette section :**
```yaml
access_control:
    - { path: ^/admin, roles: ROLE_ADMIN }
    - { path: ^/profile, roles: ROLE_USER }
```

**Remplacer par (ou ajouter AVANT les autres règles) :**
```yaml
access_control:
    - { path: ^/api/login, roles: PUBLIC_ACCESS }
    - { path: ^/api,       roles: IS_AUTHENTICATED_FULLY }
```

**📝 Résumé des modifications :**
1. ✅ Créer l'entité User AVANT toute modification de security.yaml
2. ✅ Provider : Vérifié automatiquement par `make:user`
3. ✅ Firewalls : Ajouter `login` et `api` avant `main`
4. ✅ Access control : Autoriser `/api/login` publiquement, protéger `/api` avec authentification

### Étape 5 : Configurer les Routes

Ouvrir le fichier `config/routes.yaml` et **ajouter à la fin du fichier** (après la section `controllers:`) :

```yaml
# Route pour l'authentification JWT
api_login_check:
    path: /api/login_check
```

**Exemple de fichier complet :**
```yaml
controllers:
    resource:
        path: ../src/Controller/
        namespace: App\Controller
    type: attribute

# ⬇️ AJOUTER CETTE ROUTE ICI ⬇️
api_login_check:
    path: /api/login_check
```

**📝 Note :** Cette route est gérée automatiquement par le bundle JWT. Vous n'avez pas besoin de créer de contrôleur.

### Étape 6 : Créer un Utilisateur de Test

**Option 1 : Via la commande de hashage**

```powershell
# Hasher un mot de passe
docker compose exec php bin/console security:hash-password
```

Puis insérer manuellement l'utilisateur via Adminer ou SQL.

**Option 2 : Créer une commande personnalisée**

Créer le fichier `src/Command/CreateUserCommand.php` :

```php
<?php

namespace App\Command;

use App\Entity\User;
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Component\Console\Attribute\AsCommand;
use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputArgument;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
use Symfony\Component\PasswordHasher\Hasher\UserPasswordHasherInterface;

#[AsCommand(
    name: 'app:create-user',
    description: 'Créer un nouvel utilisateur',
)]
class CreateUserCommand extends Command
{
    public function __construct(
        private EntityManagerInterface $entityManager,
        private UserPasswordHasherInterface $passwordHasher
    ) {
        parent::__construct();
    }

    protected function configure(): void
    {
        $this
            ->addArgument('email', InputArgument::REQUIRED, 'Email de l\'utilisateur')
            ->addArgument('password', InputArgument::REQUIRED, 'Mot de passe');
    }

    protected function execute(InputInterface $input, OutputInterface $output): int
    {
        $user = new User();
        $user->setEmail($input->getArgument('email'));
        $user->setRoles(['ROLE_USER']);
        
        $hashedPassword = $this->passwordHasher->hashPassword(
            $user,
            $input->getArgument('password')
        );
        $user->setPassword($hashedPassword);

        $this->entityManager->persist($user);
        $this->entityManager->flush();

        $output->writeln('✅ Utilisateur créé avec succès !');

        return Command::SUCCESS;
    }
}
```

**Utiliser la commande :**

```powershell
# Créer un utilisateur
docker compose exec php bin/console app:create-user user@example.com password123
```

### Étape 7 : Tester l'Authentification

```powershell
# Test avec curl (adapter email/password selon vos données)
curl -X POST https://localhost/api/login_check ^
  -H "Content-Type: application/json" ^
  -d "{\"email\":\"user@example.com\",\"password\":\"password123\"}"
```

**✅ Réponse attendue :**

```json
{
  "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9..."
}
```

### Étape 8 : Utiliser le Token pour Accéder aux Routes Protégées

**Créer un contrôleur de test** `src/Controller/ApiController.php` :

```php
<?php

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\Routing\Annotation\Route;

#[Route('/api', name: 'api_')]
class ApiController extends AbstractController
{
    #[Route('/users', name: 'users', methods: ['GET'])]
    public function getUsers(): JsonResponse
    {
        return $this->json([
            'message' => 'Accès autorisé !',
            'user' => $this->getUser()->getEmail(),
        ]);
    }
}
```

**Tester la route protégée :**

```powershell
# Requête authentifiée avec le token (remplacer <TOKEN> par votre token)
curl -X GET https://localhost/api/users ^
  -H "Authorization: Bearer <TOKEN>"
```

**✅ Réponse attendue :**

```json
{
  "message": "Accès autorisé !",
  "user": "user@example.com"
}
```

## 🔧 CONFIGURATION AVANCÉE

### Personnaliser la Durée de Vie du Token

Modifier le fichier `config/packages/lexik_jwt_authentication.yaml` :

```yaml
lexik_jwt_authentication:
    secret_key: '%env(resolve:JWT_SECRET_KEY)%'
    public_key: '%env(resolve:JWT_PUBLIC_KEY)%'
    pass_phrase: '%env(JWT_PASSPHRASE)%'
    token_ttl: 3600  # Durée de vie en secondes (1h par défaut)
```

### Ajouter des Données Personnalisées au Token

Créer un écouteur d'événements `src/EventListener/JWTCreatedListener.php` :

```php
<?php

namespace App\EventListener;

use Lexik\Bundle\JWTAuthenticationBundle\Event\JWTCreatedEvent;
use Symfony\Component\Security\Core\User\UserInterface;

class JWTCreatedListener
{
    public function onJWTCreated(JWTCreatedEvent $event): void
    {
        $user = $event->getUser();
        
        if (!$user instanceof UserInterface) {
            return;
        }

        $payload = $event->getData();
        
        // Ajouter des données personnalisées
        $payload['id'] = $user->getId();
        $payload['email'] = $user->getEmail();
        
        $event->setData($payload);
    }
}
```

**Enregistrer le service dans `config/services.yaml` :**

```yaml
services:
    App\EventListener\JWTCreatedListener:
        tags:
            - { name: kernel.event_listener, event: lexik_jwt_authentication.on_jwt_created, method: onJWTCreated }
```

### Implémenter un Refresh Token

```powershell
# Installer le bundle de refresh token
docker compose exec php composer require gesdinet/jwt-refresh-token-bundle
```

**Générer la table des refresh tokens :**

```powershell
docker compose exec php bin/console doctrine:schema:update --force
```

**Configurer dans `config/packages/gesdinet_jwt_refresh_token.yaml` :**

```yaml
gesdinet_jwt_refresh_token:
    ttl: 2592000  # 30 jours
    token_parameter_name: refresh_token
```

**Ajouter la route dans `config/routes.yaml` :**

```yaml
api_refresh_token:
    path: /api/token/refresh
```

### Configurer CORS

Si vous n'avez pas encore installé le bundle CORS :

```powershell
docker compose exec php composer require nelmio/cors-bundle
```

**Configurer dans `config/packages/nelmio_cors.yaml` :**

```yaml
nelmio_cors:
    defaults:
        origin_regex: true
        allow_origin: ['*']  # À restreindre en production (ex: ['https://monapp.com'])
        allow_methods: ['GET', 'OPTIONS', 'POST', 'PUT', 'PATCH', 'DELETE']
        allow_headers: ['Content-Type', 'Authorization']
        expose_headers: ['Link']
        max_age: 3600
    paths:
        '^/api/':
            allow_origin: ['*']
            allow_headers: ['Content-Type', 'Authorization']
            allow_methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE']
            max_age: 3600
```

## 🧪 TESTS ET DÉBOGAGE

### Décoder un Token JWT

Vous pouvez décoder votre token sur [jwt.io](https://jwt.io) pour voir son contenu.

**Ou via la commande Symfony :**

```powershell
# Voir les informations d'un token
docker compose exec php bin/console lexik:jwt:decode <TOKEN>
```

### Vérifier les Erreurs Courantes

**Erreur 401 - Unauthorized :**
- ✅ Vérifier que le token est bien envoyé dans le header `Authorization`
- ✅ Vérifier que le format est exactement `Bearer <token>`
- ✅ Vérifier que le token n'est pas expiré
- ✅ Vérifier que les clés JWT sont correctes

**Erreur 403 - Forbidden :**
- ✅ Vérifier les rôles de l'utilisateur
- ✅ Vérifier les règles `access_control` dans `security.yaml`

**Token non reconnu :**
- ✅ Vérifier que les fichiers `private.pem` et `public.pem` existent dans `config/jwt/`
- ✅ Vérifier la configuration dans `lexik_jwt_authentication.yaml`
- ✅ Régénérer les clés si nécessaire

**Erreur "Unable to find key" :**
- ✅ Vérifier les permissions des fichiers de clés
- ✅ Vérifier que le chemin vers les clés est correct

**Erreur "There are no commands defined in the 'make' namespace" :**

Cette erreur signifie que le **MakerBundle n'est pas installé** ou **pas enregistré**.

**Solution 1 : Installer le MakerBundle**
```powershell
docker compose exec php composer require symfony/maker-bundle --dev
```

**Solution 2 : Vérifier l'enregistrement dans config/bundles.php**

Si après installation l'erreur persiste, vérifiez que le bundle est enregistré :

```powershell
# Vérifier le contenu de config/bundles.php
docker compose exec php cat config/bundles.php
```

Le fichier doit contenir cette ligne :
```php
Symfony\Bundle\MakerBundle\MakerBundle::class => ['dev' => true, 'test' => true],
```

**Si la ligne est absente**, ajoutez-la manuellement dans `config/bundles.php` :

```php
<?php

return [
    // ... autres bundles ...
    Symfony\Bundle\MakerBundle\MakerBundle::class => ['dev' => true, 'test' => true],
];
```

Puis videz le cache :
```powershell
docker compose exec php bin/console cache:clear
```

### Commandes Utiles

```powershell
# Voir les informations d'un token
docker compose exec php bin/console lexik:jwt:decode <TOKEN>

# Régénérer les clés (⚠️ invalide tous les tokens existants)
docker compose exec php bin/console lexik:jwt:generate-keypair --overwrite

# Vérifier la configuration du pare-feu
docker compose exec php bin/console debug:firewall

# Lister toutes les routes
docker compose exec php bin/console debug:router

# Vérifier la configuration de sécurité
docker compose exec php bin/console debug:config security
```

## 🛡️ BONNES PRATIQUES DE SÉCURITÉ

### Configuration de Production

1. **Utiliser HTTPS obligatoirement**
   - Configure ton serveur web avec un certificat SSL valide

2. **Limiter la durée de vie des tokens**
   ```yaml
   token_ttl: 3600  # 1 heure recommandé
   ```

3. **Implémenter un refresh token**
   - Permet de renouveler l'accès sans redemander les identifiants

4. **Restreindre CORS**
   ```yaml
   allow_origin: ['https://monapp.com']  # Seulement les domaines autorisés
   ```

5. **Protéger les clés JWT**
   - Ne jamais les commiter dans Git
   - Utiliser des variables d'environnement en production
   - Limiter les permissions fichiers (chmod 600)

6. **Valider les données d'entrée**
   - Utiliser les contraintes Symfony pour valider email/password

7. **Logger les tentatives de connexion**
   - Surveiller les échecs d'authentification

8. **Implémenter un rate limiting**
   - Limiter le nombre de tentatives de connexion

### Variables d'Environnement

Dans le fichier `.env.local` (ne pas commiter) :

```env
JWT_SECRET_KEY=%kernel.project_dir%/config/jwt/private.pem
JWT_PUBLIC_KEY=%kernel.project_dir%/config/jwt/public.pem
JWT_PASSPHRASE=votre_passphrase_securisee
```

## 📚 Ressources Complémentaires

- 📖 [Documentation LexikJWTAuthenticationBundle](https://github.com/lexik/LexikJWTAuthenticationBundle)
- 📖 [JWT.io - Décoder et tester les tokens](https://jwt.io)
- 📖 [Guide Symfony Security](https://symfony.com/doc/current/security.html)
- 📖 [RFC 7519 - JWT Standard](https://tools.ietf.org/html/rfc7519)

## 🔗 Guides Connexes

- [API-PLATFORM.md](API-PLATFORM.md) - Créer une API REST avec API Platform
- [INSTALLATION.md](INSTALLATION.md) - Installation de Symfony Docker
- [ENTITES.md](ENTITES.md) - Gestion des entités Doctrine
- [DEPANNAGE.md](DEPANNAGE.md) - Solutions aux problèmes courants
- [JWT.md](../Flutter/JWT.md) - Utiliser JWT dans Flutter

## 📊 Récapitulatif des Commandes

```powershell
# Prérequis : Installer MakerBundle (si pas déjà installé)
docker compose exec php composer require symfony/maker-bundle --dev

# Installation JWT
docker compose exec php composer require lexik/jwt-authentication-bundle

# Génération des clés
docker compose exec php php bin/console lexik:jwt:generate-keypair

# Créer l'entité User
docker compose exec php bin/console make:user

# Migrations
docker compose exec php bin/console make:migration
docker compose exec php bin/console doctrine:migrations:migrate

# Créer un utilisateur
docker compose exec php bin/console app:create-user user@example.com password123

# Test
curl -X POST https://localhost/api/login_check -H "Content-Type: application/json" -d "{\"email\":\"user@example.com\",\"password\":\"password123\"}"
```

---

**✅ Votre API Symfony est maintenant sécurisée avec JWT ! Consultez le guide Flutter pour l'utiliser côté client.**
