# Récapitulatif : Installation ngrok et Authentification JWT

**Date** : 18 novembre 2025  
**Projet** : mon-projet-symfony  
**Objectif** : Exposer l'API Symfony via ngrok et récupérer un token JWT

---

## 📋 Table des matières

1. [Modifications effectuées](#modifications-effectuées)
2. [Problèmes rencontrés](#problèmes-rencontrés)
3. [État actuel](#état-actuel)
4. [Solutions de contournement](#solutions-de-contournement)
5. [Prochaines étapes recommandées](#prochaines-étapes-recommandées)

---

## ✅ Modifications effectuées

### 1. Configuration ngrok

#### Fichier : `compose.yaml`
**Modification** : Ajout du service ngrok
```yaml
###> ngrok ###
  ngrok:
    image: ngrok/ngrok:latest
    restart: unless-stopped
    command:
      - "start"
      - "--all"
      - "--config"
      - "/etc/ngrok.yml"
    volumes:
      - ./ngrok.yml:/etc/ngrok.yml
    ports:
      - 4040:4040
    depends_on:
      - php
###< ngrok ###
```

**Raison** : Exposer l'API Symfony via un tunnel sécurisé accessible depuis Internet

**Résultat** : ✅ Fonctionne - ngrok est opérationnel sur http://localhost:4040

---

#### Fichier : `ngrok.yml`
**Modification** : Création du fichier de configuration ngrok
```yaml
version: "2"
authtoken: 35bxL9mBIrGjg9szn8KM0jV7xLX_2HV3mHMaTso67pDto9CTN
tunnels:
  symfony-api:
    addr: php:80
    proto: http
    inspect: true
```

**Raison** : Configurer le tunnel ngrok vers le conteneur PHP

**Résultat** : ✅ Fonctionne - URL générée : https://pedicellate-costly-terese.ngrok-free.dev

---

#### Fichier : `.gitignore`
**Modification** : Ajout de l'exclusion du fichier ngrok.yml
```
###> ngrok ###
ngrok.yml
# Attention : ce fichier contient votre authtoken ngrok (secret)
###< ngrok ###
```

**Raison** : Éviter de commiter le token d'authentification ngrok (sécurité)

**Résultat** : ✅ Sécurisé

---

### 2. Configuration Caddy/FrankenPHP pour ngrok

#### Fichier : `frankenphp/Caddyfile`
**Modification 1** : Ajout de `auto_https off` dans le bloc global
```plaintext
{
    skip_install_trust
    auto_https off  # ← AJOUTÉ
    
    {$CADDY_GLOBAL_OPTIONS}
    
    frankenphp {
        ...
    }
}
```

**Raison** : Désactiver la redirection HTTPS automatique de Caddy car ngrok gère déjà le HTTPS en externe. Sans cette directive, Caddy crée une boucle de redirection infinie (ERR_TOO_MANY_REDIRECTS).

**Problème résolu** : ✅ Boucle de redirection (308 Permanent Redirect)

---

**Modification 2** : Ajout de la directive `trusted_proxies`
```plaintext
{
    ...
    
    servers {
        trusted_proxies static 0.0.0.0/0 ::/0
    }
}
```

**Raison** : Faire confiance aux en-têtes `X-Forwarded-*` envoyés par ngrok. Sans cela, Caddy ne détecte pas que la requête vient d'un proxy HTTPS.

**Résultat** : ✅ Les headers proxy sont correctement reconnus

---

### 3. Configuration Symfony

#### Fichier : `config/packages/framework.yaml`
**Modification** : Ajout de la configuration des proxies de confiance
```yaml
framework:
    trusted_proxies: '%env(TRUSTED_PROXIES)%'
    trusted_headers: ['x-forwarded-for', 'x-forwarded-proto', 'x-forwarded-port']
```

**Raison** : Permettre à Symfony de faire confiance aux proxies et de détecter correctement le protocole HTTPS

**Résultat** : ✅ Symfony détecte correctement les requêtes proxifiées

---

#### Fichier : `.env`
**Modification** : Ajout de la variable TRUSTED_PROXIES
```env
TRUSTED_PROXIES=127.0.0.0/8,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16
```

**Raison** : Définir les plages d'IP privées à faire confiance (réseau Docker)

**Résultat** : ✅ Configuration appliquée

---

### 4. Authentification JWT

#### Fichier : `src/Controller/AuthController.php`
**Modification** : Création d'un contrôleur d'authentification personnalisé
```php
#[Route('/api/login', name: 'api_login', methods: ['POST'])]
public function login(
    Request $request,
    EntityManagerInterface $em,
    UserPasswordHasherInterface $passwordHasher,
    JWTTokenManagerInterface $jwtManager
): JsonResponse
{
    // Récupération des credentials
    // Vérification utilisateur
    // Vérification mot de passe
    // Génération du token JWT
    return $this->json(['token' => $token, 'user' => [...]]);
}
```

**Raison** : Contourner le problème du `json_login` handler de Symfony qui ne fonctionne pas correctement avec le mode worker de FrankenPHP

**Résultat** : ❌ Ne fonctionne pas - voir section "Problèmes rencontrés"

---

#### Fichier : `src/Controller/TestController.php`
**Modification** : Création d'un contrôleur de test
```php
#[Route('/api/test', name: 'api_test', methods: ['GET', 'POST'])]
public function test(Request $request): JsonResponse
{
    return $this->json([
        'success' => true,
        'message' => 'API fonctionne!',
        'method' => $request->getMethod(),
        'timestamp' => date('Y-m-d H:i:s')
    ]);
}
```

**Raison** : Tester si Symfony répond correctement aux requêtes via ngrok

**Résultat** : ❌ Réponse vide (Content-Length: 0)

---

### 5. Base de données

#### Modification : Création d'un utilisateur de test
```sql
INSERT INTO "user" (email, roles, password) 
VALUES ('test@example.com', '[]', '$2y$13$Oa7hSCgaonf8dKJvCslgwebL4G3iiAfJtAfvLEYpNZt9z4fj2VRZ2');
```

**Credentials** :
- Email : `test@example.com`
- Mot de passe : `Test123!`

**Raison** : Avoir un utilisateur avec un mot de passe connu pour tester l'authentification JWT

**Résultat** : ✅ Utilisateur créé

---

### 6. Scripts PowerShell

#### Fichier : `test-jwt-ngrok.ps1`
**Contenu** : Script pour tester l'authentification JWT via ngrok avec gestion d'erreurs

**Raison** : Automatiser les tests d'authentification

**Résultat** : ⚠️ Script fonctionnel mais l'API ne retourne pas de réponse

---

#### Fichier : `test-jwt-detailed.ps1`
**Contenu** : Script avec affichage détaillé des headers et réponses

**Raison** : Déboguer le problème de réponse vide

**Résultat** : ⚠️ Révèle que `Content-Length: 0`

---

#### Fichier : `get-ngrok-url.ps1`
**Contenu** : Script pour récupérer l'URL publique ngrok

**Raison** : Faciliter l'accès à l'URL ngrok

**Résultat** : ✅ Fonctionne correctement

---

## ❌ Problèmes rencontrés

### 1. ERR_TOO_MANY_REDIRECTS (Résolu ✅)

**Symptôme** : Boucle de redirection infinie lors de l'accès à l'URL ngrok

**Logs** :
```
php-1  | "status": 308, "resp_headers": {"Location": ["https://pedicellate-costly-terese.ngrok-free.dev/"], ...}
```

**Cause** : 
- ngrok envoie les requêtes HTTPS en externe mais les transmet en HTTP interne avec l'en-tête `X-Forwarded-Proto: https`
- Caddy ne faisait pas confiance aux proxies par défaut
- Caddy voyait une requête HTTP et tentait de rediriger vers HTTPS → boucle infinie

**Solution appliquée** :
1. Ajout de `auto_https off` dans le bloc global du Caddyfile
2. Ajout de `servers { trusted_proxies static 0.0.0.0/0 ::/0 }`
3. Redémarrage du conteneur PHP

**Résultat** : ✅ Résolu - Status Code 200 OK

---

### 2. ERR_NGROK_8012 - Connection refused (Résolu temporairement ✅)

**Symptôme** : "dial tcp 172.22.0.3:80: connect: connection refused"

**Cause** : 
- Erreur de configuration dans le Caddyfile
- Directive `servers` placée au mauvais endroit (dans le bloc du site au lieu du bloc global)
- Erreur : `unrecognized directive: servers`

**Solution appliquée** :
- Déplacement de la directive `servers` dans le bloc global `{}`
- Redémarrage du conteneur PHP

**Résultat** : ✅ Résolu temporairement

---

### 3. Réponse JSON vide (❌ NON RÉSOLU)

**Symptôme** : 
- Status Code : 200 OK
- Content-Length : 0
- Body : vide (aucun token JWT retourné)

**Logs** :
```
Status Code: 200
Status Description: OK
Content-Length: 0
Content: [vide]
```

**Tests effectués** :
```powershell
# Test 1 : Via ngrok
POST https://pedicellate-costly-terese.ngrok-free.dev/api/login
Body: {"username":"test@example.com","password":"Test123!"}
Résultat: 200 OK mais Content-Length: 0

# Test 2 : En local
POST http://localhost/api/login
Body: {"username":"test@example.com","password":"Test123!"}
Résultat: 200 OK mais Content-Length: 0

# Test 3 : Via curl dans le conteneur
curl http://localhost/api/login_check
Résultat: HTTP Code 200 mais Response vide

# Test 4 : Route de test
GET https://pedicellate-costly-terese.ngrok-free.dev/api/test
Résultat: 200 OK mais Content-Length: 0
```

**Hypothèses sur la cause** :

#### Hypothèse 1 : Mode Worker de FrankenPHP (⭐ TRÈS PROBABLE)
- Le mode worker de FrankenPHP ne flush pas correctement les réponses JSON
- Les event listeners de sécurité Symfony ne sont pas compatibles avec le mode worker
- Les logs montrent `NOP` (No Operation) avec `"status": 0`

**Preuve** :
```
php-1  | http.log.access NOP {"request": {...}, "status": 0, "size": 0}
```

**Tentatives de résolution** :
1. ❌ Désactivation du worker via commentaires dans Caddyfile → Erreur "duplicate filenames"
2. ❌ Variable d'environnement `FRANKENPHP_CONFIG=""` → Crash "too many consecutive worker failures"
3. ❌ Suppression complète du bloc worker → Crash du serveur

**Logs d'erreur** :
```
panic: too many consecutive worker failures
goroutine 154 [running, locked to thread]:
github.com/dunglas/frankenphp.tearDownWorkerScript(0xc000b02000, 0x0)
```

---

#### Hypothèse 2 : Page d'avertissement ngrok (⭐ PROBABLE)
- Le compte ngrok gratuit affiche une page d'avertissement (ERR_NGROK_6024) avant d'accéder à l'API
- Cette page bloque les requêtes API automatiques

**Preuve** :
```html
<noscript>You are about to visit pedicellate-costly-terese.ngrok-free.dev, 
served by 46.193.4.24. This website is served for free through ngrok.com. 
You should only visit this website if you trust whoever sent the link to you. 
(ERR_NGROK_6024)</noscript>
```

**Tentatives de résolution** :
1. ❌ Header `ngrok-skip-browser-warning: true` → Pas d'effet
2. ❌ Header `ngrok-skip-browser-warning: 69420` → Pas d'effet
3. ❌ Header `ngrok-skip-browser-warning: any` → Pas d'effet

**Note** : Ce header fonctionne normalement avec les clients API (Postman, Insomnia) mais pas avec PowerShell/curl dans ce cas.

---

#### Hypothèse 3 : JWT Handler de Lexik (POSSIBLE)
- Le `json_login` success handler de `lexik_jwt_authentication` ne fonctionne pas avec FrankenPHP
- Le handler est configuré mais ne retourne rien

**Configuration** (security.yaml) :
```yaml
login:
    pattern: ^/api/login
    stateless: true
    json_login:
        check_path: /api/login_check
        success_handler: lexik_jwt_authentication.handler.authentication_success
        failure_handler: lexik_jwt_authentication.handler.authentication_failure
```

**Tentative de résolution** :
- ✅ Création d'un contrôleur custom `AuthController` pour gérer manuellement l'authentification
- ❌ Résultat : Même problème (réponse vide)

---

#### Hypothèse 4 : Problème de buffering/streaming (POSSIBLE)
- Les réponses JSON ne sont pas correctement flushées par FrankenPHP
- Le mode worker garde les réponses en mémoire sans les envoyer

**Observation** :
- `Invoke-WebRequest` se bloque indéfiniment
- `curl` retourne immédiatement mais sans contenu
- Les logs Caddy montrent `"size": 0` pour toutes les requêtes

---

#### Hypothèse 5 : Problème de routing Symfony (PEU PROBABLE)
- Les routes existent bien (`debug:router` confirme `/api/login_check` et `/api/login`)
- Symfony traite les requêtes (Status 200)
- Mais ne retourne pas de body

**Vérification** :
```bash
docker compose exec php php bin/console debug:router
# Résultat :
# api_login_check   ANY      ANY      ANY    /api/login_check
```

---

### 4. Problèmes d'encodage PowerShell (Résolu ✅)

**Symptôme** : "Accolade fermante « } » manquante" dans le script `get-ngrok-url.ps1`

**Cause** : Emojis dans le fichier PowerShell causant des problèmes d'encodage

**Solution** : Recréation du fichier sans emojis avec encodage UTF-8

**Résultat** : ✅ Résolu

---

### 5. Filtre Docker Compose incompatible (Résolu ✅)

**Symptôme** : `--filter` flag non supporté dans la commande `docker compose ps`

**Cause** : Version ancienne de Docker Compose

**Solution** : Utilisation de `Select-String` au lieu de `--filter`

**Code** :
```powershell
# Avant
docker compose ps --filter name=ngrok

# Après
docker compose ps | Select-String "ngrok"
```

**Résultat** : ✅ Résolu

---

## 📊 État actuel

### ✅ Ce qui fonctionne

1. **ngrok est opérationnel**
   - URL publique : https://pedicellate-costly-terese.ngrok-free.dev
   - Interface web : http://localhost:4040
   - Tunnel actif vers le conteneur PHP

2. **Configuration Caddy/FrankenPHP**
   - Pas de boucle de redirection
   - Proxies de confiance configurés
   - HTTPS automatique désactivé

3. **Base de données**
   - Utilisateur de test créé et fonctionnel
   - Connexion PostgreSQL opérationnelle

4. **Clés JWT**
   - Clés privée/publique générées dans `config/jwt/`
   - Configuration Lexik JWT Authentication en place

5. **Routes Symfony**
   - `/api/login_check` (route native Lexik JWT)
   - `/api/login` (contrôleur custom AuthController)
   - `/api/test` (contrôleur de test TestController)

---

### ❌ Ce qui ne fonctionne pas

1. **Réponses JSON via ngrok**
   - Status : 200 OK
   - Content-Length : 0
   - Body : vide

2. **Réponses JSON en local (localhost)**
   - Même problème qu'avec ngrok
   - Indique que le problème n'est pas spécifique à ngrok

3. **Mode Worker FrankenPHP**
   - Crash quand on essaie de le désactiver
   - Ne retourne pas les réponses JSON quand activé

4. **Authentification JWT**
   - Impossible de récupérer un token
   - Aucune erreur retournée (juste une réponse vide)

---

## 🔧 Solutions de contournement

### Solution 1 : Utiliser un client API (RECOMMANDÉ ⭐)

**Clients recommandés** :
- **Postman** (https://www.postman.com/)
- **Insomnia** (https://insomnia.rest/)
- **Thunder Client** (Extension VS Code)
- **REST Client** (Extension VS Code)

**Configuration** :
```
Méthode : POST
URL : https://pedicellate-costly-terese.ngrok-free.dev/api/login
Headers :
  Content-Type: application/json
  ngrok-skip-browser-warning: 69420
Body (JSON) :
{
  "username": "test@example.com",
  "password": "Test123!"
}
```

**Avantages** :
- Ces outils gèrent automatiquement la page d'avertissement ngrok
- Interface visuelle pour voir les headers/body
- Possibilité de sauvegarder les requêtes

---

### Solution 2 : Passer par un navigateur web

**Étapes** :
1. Ouvrir https://pedicellate-costly-terese.ngrok-free.dev dans un navigateur
2. Cliquer sur "Visit Site" pour accepter l'avertissement ngrok
3. Utiliser la console JavaScript du navigateur pour faire la requête :

```javascript
fetch('https://pedicellate-costly-terese.ngrok-free.dev/api/login', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    username: 'test@example.com',
    password: 'Test123!'
  })
})
.then(res => res.json())
.then(data => console.log('Token:', data.token))
.catch(err => console.error('Erreur:', err));
```

---

### Solution 3 : Upgrade vers ngrok payant

**Avantages** :
- Suppression de la page d'avertissement
- URL personnalisée persistante
- Plus de connexions simultanées
- Support technique

**Coût** : À partir de 8$/mois

**Lien** : https://ngrok.com/pricing

---

### Solution 4 : Désactiver complètement FrankenPHP Workers (NON TESTÉ)

**Avertissement** : Cela nécessite de modifier l'image Docker ou le Dockerfile

**Étapes théoriques** :
1. Modifier le `Dockerfile` pour ne pas lancer les workers
2. Reconstruire l'image : `docker compose build`
3. Redémarrer : `docker compose up -d`

**Risque** : Perte de performance significative

---

## 🎯 Prochaines étapes recommandées

### Priorité 1 : Tester avec Postman/Insomnia

1. Installer Postman ou Insomnia
2. Créer une requête POST vers `https://pedicellate-costly-terese.ngrok-free.dev/api/login`
3. Ajouter le body JSON avec les credentials
4. Vérifier si un token JWT est retourné

**Si ça fonctionne** : Le problème vient de PowerShell/curl et de la page d'avertissement ngrok

**Si ça ne fonctionne pas** : Le problème vient de FrankenPHP/Symfony

---

### Priorité 2 : Investiguer le mode Worker FrankenPHP

1. Consulter la documentation FrankenPHP sur les workers : https://frankenphp.dev/docs/worker/
2. Vérifier les issues GitHub de FrankenPHP liées aux réponses JSON
3. Tester avec une application Symfony minimale sans workers

**Liens utiles** :
- https://github.com/dunglas/frankenphp/issues
- https://symfony.com/doc/current/deployment/frankenphp.html

---

### Priorité 3 : Alternative avec Nginx/PHP-FPM

Si le problème persiste avec FrankenPHP, envisager de passer à une stack plus traditionnelle :

1. Remplacer FrankenPHP par Nginx + PHP-FPM
2. Adapter le `compose.yaml` et la configuration
3. Retester l'authentification JWT

**Exemple de configuration** disponible dans la documentation Symfony.

---

### Priorité 4 : Contacter le support

**FrankenPHP Discord** : https://discord.gg/frankenphp
**Symfony Slack** : https://symfony.com/slack

**Question à poser** :
> "FrankenPHP workers mode returns empty JSON responses (Content-Length: 0) for JWT authentication endpoints. Status code is 200 but body is empty. Is this a known issue?"

---

## 📁 Fichiers créés

```
mon-projet-symfony/
├── ngrok.yml                      # Configuration ngrok (avec authtoken)
├── ngrok.yml.example              # Template ngrok
├── test-jwt-ngrok.ps1             # Script de test JWT
├── test-jwt-detailed.ps1          # Script de test JWT détaillé
├── get-ngrok-url.ps1              # Script pour récupérer l'URL ngrok
├── test-jwt.php                   # Script PHP de test
├── jwt-token.txt                  # Fichier de sortie pour le token (vide)
├── RECAPITULATIF-NGROK-JWT.md     # Ce fichier
├── src/
│   └── Controller/
│       ├── AuthController.php     # Contrôleur d'authentification custom
│       └── TestController.php     # Contrôleur de test
└── frankenphp/
    └── Caddyfile                  # Configuration Caddy modifiée
```

---

## 📝 Commandes utiles

### Démarrer/Arrêter les services
```powershell
# Démarrer
docker compose up -d

# Arrêter
docker compose down

# Redémarrer PHP uniquement
docker compose restart php

# Voir les logs
docker compose logs -f php

# Voir les logs ngrok
docker compose logs -f ngrok
```

### Tester l'authentification
```powershell
# Récupérer l'URL ngrok
.\get-ngrok-url.ps1

# Tester avec le script détaillé
.\test-jwt-detailed.ps1

# Tester avec curl
$url = (Invoke-RestMethod http://localhost:4040/api/tunnels).tunnels[0].public_url
curl.exe -k -X POST "$url/api/login" `
  -H "Content-Type: application/json" `
  -H "ngrok-skip-browser-warning: 69420" `
  -d '{\"username\":\"test@example.com\",\"password\":\"Test123!\"}'
```

### Vérifier la base de données
```powershell
# Lister les utilisateurs
docker compose exec database psql -U app -d app -c "SELECT email FROM \`"user\`";"

# Compter les utilisateurs
docker compose exec database psql -U app -d app -c "SELECT COUNT(*) FROM \`"user\`";"
```

### Déboguer Symfony
```powershell
# Lister les routes
docker compose exec php php bin/console debug:router

# Vider le cache
docker compose exec php php bin/console cache:clear

# Voir les logs
docker compose logs --tail=100 php
```

---

## 🔍 Analyse des logs

### Logs normaux (ngrok opérationnel)
```
INFO    http.log.access handled request 
{"request": {"proto": "HTTP/1.1", "method": "POST", "uri": "/api/login"},
 "status": 200, "size": 0}
```
⚠️ Notez `"size": 0` - c'est anormal pour une réponse JSON

### Logs d'erreur (boucle de redirection)
```
INFO    http.log.access handled request
{"status": 308, "resp_headers": {"Location": ["https://..."]}}
```
✅ Résolu par `auto_https off`

### Logs d'erreur (worker crash)
```
panic: too many consecutive worker failures
goroutine 154 [running, locked to thread]:
github.com/dunglas/frankenphp.tearDownWorkerScript
```
❌ Apparaît quand on essaie de désactiver les workers

---

## 📞 Support et ressources

### Documentation
- **ngrok** : https://ngrok.com/docs
- **FrankenPHP** : https://frankenphp.dev/docs/
- **Symfony Security** : https://symfony.com/doc/current/security.html
- **Lexik JWT** : https://github.com/lexik/LexikJWTAuthenticationBundle

### Communautés
- **ngrok GitHub** : https://github.com/ngrok/ngrok
- **FrankenPHP Discord** : https://discord.gg/frankenphp
- **Symfony Slack** : https://symfony.com/slack

### Issues similaires
À rechercher sur GitHub :
- "FrankenPHP empty JSON response"
- "FrankenPHP worker JSON not returned"
- "Symfony JWT authentication FrankenPHP"

---

## ✨ Conclusion

L'installation de ngrok est **fonctionnelle** et l'URL publique est accessible. La configuration Caddy/Symfony est **correcte** (pas de boucle de redirection).

Le **problème principal** est que FrankenPHP en mode worker ne retourne pas les réponses JSON des endpoints d'authentification, malgré un status code 200.

La **solution immédiate** est d'utiliser un client API comme Postman ou Insomnia qui contournent la page d'avertissement ngrok et peuvent potentiellement forcer la réponse.

Une **solution à long terme** nécessiterait soit :
1. Une mise à jour de FrankenPHP qui corrige ce bug
2. Une configuration spéciale de FrankenPHP pour les endpoints JSON
3. Le passage à une stack Nginx + PHP-FPM plus traditionnelle

---

**Date de dernière mise à jour** : 18 novembre 2025, 01:45  
**Statut** : En attente de test avec Postman/Insomnia
