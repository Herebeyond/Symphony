# 🚀 Installation de Symfony Docker - Guide Débutant

**Ce guide est conçu pour les débutants complets.** Suivez chaque étape exactement comme écrit et vous réussirez !

## ⚠️ AVANT DE COMMENCER

**LISEZ CECI EN ENTIER AVANT DE FAIRE QUOI QUE CE SOIT !**

**🔧 Pour développer :** 
- `ENTITES.md` → Créer nouveaux objets métier
- `DONNEES-BDD.md` → Manipuler les données via commandes
- `API-PLATFORM.md` → Installer et configurer des APIs REST modernes
- `DEPANNAGE.md` → Solutions aux problèmes courants

**📝 PREMIÈRE ÉTAPE après installation :** [AJOUT-ADMINER.md](AJOUT-ADMINER.md) - **Indispensable** pour avoir une interface graphique de base de données !

### Prérequis Obligatoires
- ✅ **Docker Desktop** installé ET en cours d'exécution ([Télécharger ici](https://www.docker.com/products/docker-desktop/))
- ✅ **Git** installé ([Télécharger ici](https://git-scm.com/downloads))
- ✅ **Windows PowerShell** (déjà présent sur Windows)

### ⏱️ Temps nécessaire (la durée peut changer en fonction du hardware et de la connexion internet)
- **Première installation complète : 8-12 minutes**
- **La construction Docker prend 4-6 minutes** (normal, ne pas interrompre !)
- **Démarrage containers : 30-60 secondes**

## 🎯 INSTALLATION ÉTAPE PAR ÉTAPE

### Étape 1 : Créer le Projet depuis Zéro (3-5 minutes)

```powershell
# 1. Aller dans votre dossier de travail (remplacer par votre chemin)
exemple :
cd C:\Users\VotreNom\Desktop

# 2. Cloner le template officiel Symfony-Docker
git clone https://github.com/dunglas/symfony-docker.git mon-projet-symfony

# 3. Entrer dans le dossier
cd mon-projet-symfony

# 4. Supprimer l'historique git du template (optionnel)
Remove-Item -Recurse -Force .git

# 5. Initialiser votre propre dépôt Git (optionnel)
git init
git add .
git commit -m "Initial commit from symfony-docker template"
```

**⚠️ IMPORTANT :** 
- Le template officiel `dunglas/symfony-docker` contient un Symfony vierge
- Vous devrez ensuite ajouter vos propres entités et fonctionnalités
- Ce guide suppose que vous partez du template de base

### Étape 2 : Nettoyer l'environnement (1 minute)

**⚠️ IMPORTANT :** Arrêtez d'abord tous les autres projets Docker :

```powershell
# Arrêter TOUS les containers Docker actuels
docker stop $(docker ps -aq)
```

**⚠️ ATTENTION :** la prochaine action supprimera tous vos volumes, si vous ne voulez pas faire cela, vous pouvez vérifier manuellement les ports si aucun n'entre en conflit


```powershell
# Nettoyer les volumes (pour éviter les conflits)
docker system prune -f --volumes
```

### Étape 3 : Construire le Projet (4-6 minutes)

**⚠️ ATTENTION :** Cette étape peut être LONGUE (4-6 minutes). Ne fermez PAS la fenêtre !

```powershell
# Construction du projet (étape potentiellement longue - PATIENTEZ !)
docker compose build --pull --no-cache

# Démarrer le projet (Peut également prendre du temps)
docker compose up -d --wait
```

## ✅ VÉRIFICATION DE L'INSTALLATION

### 1. **Vérifier que les Services Fonctionnent**
```powershell
# Voir l'état de tous les services (dans notre cas, un seul devrait être en cours d'exécution)
docker compose ps

# Tous les services doivent afficher "healthy" ou "running"
```

### 2. **Tester l'Application Web**
- 🌐 **HTTP :** [http://localhost](http://localhost)

**Vous devez voir :** Page d'accueil Symfony par défaut avec le message "Welcome to Symfony" et des informations sur la version

### 3. **Tester l'Interface Base de Données (Adminer)**

> **⚠️ IMPORTANT :** Le template officiel n'inclut **PAS** Adminer par défaut. Pour l'ajouter, suivez le guide [AJOUT-ADMINER.md](AJOUT-ADMINER.md) après cette installation.

- 📊 **URL :** [http://localhost:8080](http://localhost:8080) (seulement après ajout d'Adminer)
- **Système :** PostgreSQL
- **Serveur :** database
- **Utilisateur :** app  
- **Mot de passe :** !ChangeMe!
- **Base de données :** app


**Si Adminer n'est pas installé :** Cette étape ne fonctionnera pas - c'est normal !

### 4. **Vérifier Symfony en Ligne de Commande**
```powershell
# Vérifier la version de Symfony
docker compose exec php bin/console --version

# Lister les commandes disponibles
docker compose exec php bin/console list

# Vérifier l'état de l'environnement
docker compose exec php bin/console about
```

### 5. **Vérifier les Logs (en cas de problème)**
```powershell
# Voir les logs en temps réel
docker compose logs -f php

# Voir les logs de la base de données
docker compose logs database
```

**⏱️ TEMPS TOTAL D'INSTALLATION :** 4-6 minutes selon votre connexion internet

> **✨ FÉLICITATIONS !** Si vous voyez la page d'accueil Symfony, votre installation est réussie ! 
> 
> **📝 Note :** Adminer n'est pas inclus par défaut. Pour ajouter PostgreSQL + Adminer, consultez le guide [AJOUT-ADMINER.md](AJOUT-ADMINER.md).

## 📝 ÉTAPES SUIVANTES

### 1️⃣ **PREMIÈRE ÉTAPE OBLIGATOIRE :** Ajouter une Base de Données
👉 **[AJOUT-ADMINER.md](AJOUT-ADMINER.md)** - Installer PostgreSQL + Adminer (interface graphique)

### 2️⃣ **Pour Développer :**
- 🏗️ **[ENTITES.md](ENTITES.md)** - Créer des entités et tables de base de données
- 💾 **[DONNEES-BDD.md](DONNEES-BDD.md)** - Manipuler les données via commandes
- 🌐 **[API-PLATFORM.md](API-PLATFORM.md)** - Créer des APIs REST modernes
- 🎨 **[NODEJS.md](NODEJS.md)** - Utiliser Node.js et NPM pour le frontend
- 📊 **[DIAGRAMMES.md](DIAGRAMMES.md)** - Générer des schémas de base de données

### 3️⃣ **En Cas de Problème :**
- 🛠️ **[DEPANNAGE.md](DEPANNAGE.md)** - Solutions à tous les problèmes courants

## 📋 COMMANDES DE BASE

```powershell
# Démarrer le projet
docker compose up -d

# Arrêter le projet
docker compose down

# Voir les logs
docker compose logs -f php

# Accéder au container
docker compose exec php bash
```

## 🌐 URLs IMPORTANTES

- **Application Symfony :** http://localhost
- **Adminer (BDD) :** http://localhost:8080 (après installation - voir AJOUT-ADMINER.md)

---

**⚠️ Besoin d'aide ?** Consultez **[DEPANNAGE.md](DEPANNAGE.md)** pour toutes les solutions aux problèmes courants.