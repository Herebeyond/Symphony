# 📦 Node.js et NPM dans Symfony Docker

Ce guide explique comment utiliser Node.js et NPM dans votre environnement Symfony Docker sans avoir besoin d'installer Node.js sur votre machine hôte.

## 🎯 Avantages de l'Intégration

- ✅ **Pas d'installation locale** - Node.js et NPM sont intégrés dans le conteneur
- ✅ **Version cohérente** - Tous les développeurs utilisent la même version de Node.js
- ✅ **Performance optimisée** - Cache NPM persistant pour des installations plus rapides
- ✅ **Isolation complète** - Les node_modules restent dans le conteneur
- ✅ **Environnement propre** - Pas de conflits avec d'autres projets

## 🚀 Utilisation Quotidienne

### Versions Installées

```bash
# Vérifier la version de Node.js (LTS)
docker compose exec php node --version
# Sortie: v22.19.0

# Vérifier la version de NPM
docker compose exec php npm --version
# Sortie: v10.9.3
```

### Gestion des Packages

```bash
# Initialiser ou mettre à jour package.json
docker compose exec php npm init
docker compose exec php npm install

# Installer des dépendances de développement
docker compose exec php npm install --save-dev webpack webpack-cli
docker compose exec php npm install --save-dev sass postcss autoprefixer

# Installer des dépendances de production
docker compose exec php npm install --save axios lodash
docker compose exec php npm install --save vue@next

# Désinstaller des packages
docker compose exec php npm uninstall package-name

# Mettre à jour des packages
docker compose exec php npm update
```

### Scripts NPM

```bash
# Exécuter des scripts définis dans package.json
docker compose exec php npm run dev
docker compose exec php npm run build
docker compose exec php npm run test
docker compose exec php npm run lint

# Lancer un serveur de développement (si configuré)
docker compose exec php npm run serve
```

### Outils en Ligne de Commande (npx)

```bash
# Utiliser des outils sans les installer globalement
docker compose exec php npx webpack --version
docker compose exec php npx create-react-app my-frontend
docker compose exec php npx vue create my-vue-app
docker compose exec php npx @angular/cli new my-angular-app

# Exécuter des linters ou formateurs
docker compose exec php npx eslint src/
docker compose exec php npx prettier --write "src/**/*.js"
```

## 📁 Structure de Fichiers Recommandée

```
symfony-docker/
├── package.json              # Configuration NPM
├── package-lock.json          # Versions exactes des packages
├── node_modules/             # Packages installés (géré par Docker)
├── assets/                   # Sources JavaScript/CSS
│   ├── js/
│   │   ├── app.js
│   │   └── components/
│   ├── scss/
│   │   ├── app.scss
│   │   └── components/
│   └── images/
├── public/                   # Assets compilés
│   ├── js/
│   └── css/
└── webpack.config.js         # Configuration Webpack
```

## 🛠️ Configuration Typique

### Package.json Exemple

```json
{
  "name": "symfony-docker-app",
  "version": "1.0.0",
  "scripts": {
    "dev": "webpack --mode development --watch",
    "build": "webpack --mode production",
    "lint": "eslint assets/js/",
    "format": "prettier --write 'assets/**/*.{js,css,scss}'"
  },
  "devDependencies": {
    "webpack": "^5.0.0",
    "webpack-cli": "^5.0.0",
    "sass-loader": "^13.0.0",
    "css-loader": "^6.0.0",
    "style-loader": "^3.0.0",
    "babel-loader": "^9.0.0",
    "@babel/core": "^7.0.0",
    "@babel/preset-env": "^7.0.0",
    "eslint": "^8.0.0",
    "prettier": "^3.0.0"
  },
  "dependencies": {
    "axios": "^1.0.0",
    "lodash": "^4.0.0"
  }
}
```

### Webpack.config.js Exemple

```javascript
const path = require('path');

module.exports = {
    entry: './assets/js/app.js',
    output: {
        filename: 'app.js',
        path: path.resolve(__dirname, 'public/js'),
    },
    module: {
        rules: [
            {
                test: /\.scss$/,
                use: ['style-loader', 'css-loader', 'sass-loader'],
            },
            {
                test: /\.js$/,
                exclude: /node_modules/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: ['@babel/preset-env']
                    }
                }
            }
        ],
    },
};
```

## 🔄 Workflow de Développement

### 1. Développement avec Watch Mode

```bash
# Lancer Webpack en mode watch
docker compose exec php npm run dev

# Dans un autre terminal, démarrer le serveur Symfony
docker compose up -d
```

### 2. Construction pour la Production

```bash
# Construire les assets pour la production
docker compose exec php npm run build

# Les fichiers optimisés sont dans public/js/ et public/css/
```

### 3. Intégration avec Twig

Dans vos templates Twig :

```twig
{# templates/base.html.twig #}
<link href="{{ asset('css/app.css') }}" rel="stylesheet">
<script src="{{ asset('js/app.js') }}"></script>
```

## ⚡ Optimisations de Performance

### Cache NPM Persistant

Le projet est configuré avec un volume `npm_cache` pour des installations plus rapides :

```yaml
# compose.yaml
volumes:
  npm_cache:

# compose.override.yaml
services:
  php:
    volumes:
      - npm_cache:/root/.npm
```

### Node_modules Isolés

Les `node_modules` restent dans le conteneur pour de meilleures performances :

```yaml
# compose.override.yaml
services:
  php:
    volumes:
      - /app/node_modules  # Volume anonyme
```

## 🆘 Dépannage

### Problème : "npm command not found"

```bash
# Vérifiez que le conteneur est démarré
docker compose ps

# Reconstruisez l'image si nécessaire
docker compose build --no-cache php
```

### Problème : Installations NPM Lentes

```bash
# Vérifiez que le cache NPM est utilisé
docker compose exec php npm config get cache

# Nettoyez le cache si nécessaire
docker compose exec php npm cache clean --force
```

### Problème : Module non trouvé

```bash
# Réinstallez les dépendances
docker compose exec php rm -rf node_modules package-lock.json
docker compose exec php npm install
```

## 🔗 Intégrations Populaires

### Avec Symfony Webpack Encore

```bash
# Installer Symfony Webpack Encore
docker compose exec php composer require symfony/webpack-encore-bundle

# Configurer Encore
docker compose exec php npm install @symfony/webpack-encore --save-dev
```

### Avec Vue.js

```bash
# Installer Vue.js
docker compose exec php npm install vue@next

# Installer le loader Vue pour Webpack
docker compose exec php npm install --save-dev vue-loader @vue/compiler-sfc
```

### Avec React

```bash
# Installer React
docker compose exec php npm install react react-dom

# Installer Babel preset pour React
docker compose exec php npm install --save-dev @babel/preset-react
```

## 📚 Ressources Complémentaires

- [Documentation Node.js Officielle](https://nodejs.org/docs/)
- [Guide NPM](https://docs.npmjs.com/)
- [Webpack Documentation](https://webpack.js.org/concepts/)
- [Symfony Webpack Encore](https://symfony.com/doc/current/frontend.html)

---

**💡 Astuce :** Utilisez toujours `docker compose exec php` devant vos commandes Node.js pour garantir un environnement cohérent !