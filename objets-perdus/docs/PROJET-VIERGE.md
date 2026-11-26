# Comprendre le projet vierge

Ce document explique ce que `ionic start` a créé : à quoi sert chaque fichier, et lesquels on va modifier.

---

## 1. Vue d'ensemble

```
objets-perdus/
├── src/                    ⭐ NOTRE CODE : c'est ici qu'on travaille 95 % du temps
│   ├── main.ts             Point de démarrage de l'application
│   ├── App.vue             Composant racine (la « coquille »)
│   ├── router/index.ts     Les routes : quelle URL affiche quelle page
│   ├── views/              Les pages (écrans)
│   ├── components/         Les morceaux réutilisables
│   ├── theme/variables.css Les couleurs et le thème
│   └── vite-env.d.ts       Déclaration technique pour TypeScript (ne pas toucher)
│
├── public/                 Fichiers copiés tels quels (favicon, images fixes)
├── tests/                  Tests automatiques (on ne s'en sert pas pour le TP)
├── docs/                   Notre documentation
├── node_modules/           Les librairies téléchargées par npm (ne jamais toucher)
│
├── index.html              La page HTML unique de l'application
├── package.json            La « carte d'identité » du projet et ses dépendances
├── package-lock.json       Versions exactes installées (géré par npm)
├── capacitor.config.ts     Config Android (identifiant et nom de l'application)
├── ionic.config.json       Config de l'Ionic CLI
├── vite.config.ts          Config de Vite (l'outil qui compile)
├── tsconfig*.json          Config de TypeScript
├── eslint.config.js        Règles de qualité du code
├── .gitignore              Fichiers que Git doit ignorer
└── .vscode/                Extensions VS Code recommandées
```

**À retenir :** on travaille dans `src/`. Les fichiers de configuration à la racine ne changent que rarement.

---

## 2. Comment l'application démarre

Quand on lance `ionic serve`, voici ce qui se passe, dans l'ordre :

```
index.html            1. Le navigateur charge la page, qui contient <div id="app">
   │                     et appelle main.ts
   ▼
src/main.ts           2. Crée l'application Vue, branche Ionic et le routeur,
   │                     puis l'installe dans <div id="app">
   ▼
src/App.vue           3. La coquille : <ion-app> + <ion-router-outlet>
   │                     (l'emplacement où le routeur affichera les pages)
   ▼
src/router/index.ts   4. Regarde l'URL : « / » redirige vers « /tabs/tab1 »
   │
   ▼
views/TabsPage.vue    5. Affiche la barre d'onglets en bas,
   │                     avec un emplacement pour la page de l'onglet actif
   ▼
views/Tab1Page.vue    6. La page de l'onglet 1 s'affiche dans cet emplacement
```

---

## 3. Les fichiers de `src/`, un par un

### `main.ts` : le démarrage
```ts
const app = createApp(App).use(IonicVue).use(router);

router.isReady().then(() => {
  app.mount('#app');
});
```
- `createApp(App)` crée l'application à partir du composant racine `App.vue`.
- `.use(IonicVue)` active Ionic : les composants mobiles et les animations.
- `.use(router)` active la navigation entre les pages.
- `app.mount('#app')` affiche l'application dans le `<div id="app">` de `index.html`. On attend d'abord que le routeur soit prêt.
- Les nombreux `import '...css'` chargent les styles d'Ionic. La ligne `dark.system.css` active le **mode sombre automatique**, selon le réglage du téléphone.

👉 **On le modifiera** pour ajouter Pinia : `.use(createPinia())`.

### `App.vue` : la coquille
```vue
<ion-app>
  <ion-router-outlet />
</ion-app>
```
- `<ion-app>` est le conteneur obligatoire de toute application Ionic.
- `<ion-router-outlet>` est un **emplacement vide** que le routeur remplit avec la page correspondant à l'URL.

👉 On n'y touchera presque jamais.

### `router/index.ts` : la navigation
```ts
const routes = [
  { path: '/', redirect: '/tabs/tab1' },
  {
    path: '/tabs/',
    component: TabsPage,
    children: [
      { path: 'tab1', component: () => import('@/views/Tab1Page.vue') },
      ...
    ],
  },
];
```
- Chaque **route** associe une **URL** à une **page**.
- `redirect` : si on arrive sur `/`, on est envoyé vers `/tabs/tab1`.
- `children` : des pages **à l'intérieur** de `TabsPage`. La barre d'onglets reste visible et seul le contenu change.
- `() => import(...)` : la page est chargée **seulement quand on y va** (*lazy loading*), ce qui accélère le démarrage.
- `@/` est un raccourci pour `src/` (défini dans `vite.config.ts` et `tsconfig.json`).

👉 **On le modifiera beaucoup** : routes `/login`, `/register`, `/items/:id`, plus le guard qui bloque l'accès si l'utilisateur n'est pas connecté.

### `views/` : les pages
| Fichier | Rôle | Ce qu'il deviendra |
|---|---|---|
| `TabsPage.vue` | Barre d'onglets en bas | Annonces / Publier / Mes annonces / Profil |
| `Tab1Page.vue` | Page de l'onglet 1 | `ItemListPage.vue` |
| `Tab2Page.vue` | Page de l'onglet 2 | `ItemFormPage.vue` |
| `Tab3Page.vue` | Page de l'onglet 3 | `MyItemsPage.vue` |

### `components/ExploreContainer.vue`
Un composant d'exemple qui affiche « Tab 1 page, Explore UI Components ». Il montre comment **passer une donnée** d'une page à un composant :
```vue
<!-- Dans Tab1Page.vue : on envoie la donnée -->
<ExploreContainer name="Tab 1 page" />

<!-- Dans ExploreContainer.vue : on la reçoit -->
defineProps({ name: String });
<strong>{{ name }}</strong>
```
C'est le principe des **props**. On l'utilisera pour `ItemCard`, qui recevra une annonce à afficher.

👉 **À supprimer** plus tard, quand nos vraies pages seront en place.

### `theme/variables.css`
C'est là qu'on personnalise les **couleurs** de l'application (`--ion-color-primary`…).

---

## 4. Anatomie d'un fichier `.vue`

Un fichier `.vue` regroupe **trois parties** :

```vue
<template>
  <!-- 1. LE HTML : ce qu'on voit à l'écran -->
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Tab 1</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      ...
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
  // 2. LE CODE (TypeScript) : imports, données, fonctions
  import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent } from '@ionic/vue';
</script>

<style scoped>
  /* 3. LE STYLE : CSS qui ne s'applique qu'à ce fichier (scoped) */
</style>
```

**Structure d'une page Ionic :**
| Balise | Rôle |
|---|---|
| `<ion-page>` | **Obligatoire** pour chaque page, sinon la navigation casse |
| `<ion-header>` + `<ion-toolbar>` + `<ion-title>` | La barre du haut avec le titre |
| `<ion-content>` | Le contenu qui défile |
| `<ion-header collapse="condense">` | Grand titre style iOS qui rétrécit au scroll |

**Règles à retenir :**
- Chaque composant Ionic utilisé dans `<template>` doit être **importé** dans `<script>`. Si on écrit `<ion-button>`, il faut `import { IonButton } from '@ionic/vue'`.
- `:fullscreen="true"` : le `:` devant un attribut veut dire « la valeur est du code », ici le booléen `true`, et non le texte `"true"`.
- `{{ name }}` affiche la valeur d'une variable dans le HTML.
- `<script setup lang="ts">` : `setup` est la syntaxe moderne de Vue 3, et `lang="ts"` indique qu'on écrit en TypeScript.

---

## 5. Les fichiers de configuration (racine)

### `package.json` : la carte d'identité
- `name` : le nom du projet.
- `scripts` : des raccourcis lancés avec `npm run <nom>`. Par exemple `npm run build`, et `npm run dev` qui fait la même chose que `ionic serve`.
- `dependencies` : les librairies **utilisées par l'application** et incluses dans l'APK (Vue, Ionic, Capacitor…).
- `devDependencies` : les outils de **développement seulement** (Vite, TypeScript, ESLint, tests…), absents de l'APK.

Quand on fait `npm install firebase`, npm ajoute automatiquement une ligne dans `dependencies`.

### `node_modules/` et `package-lock.json`
- `node_modules/` contient **le code de toutes les librairies** (plusieurs centaines de dossiers). Il est **lourd** et **ignoré par Git** : n'importe qui peut le recréer avec `npm install`.
- `package-lock.json` note les **versions exactes** installées, pour que toute l'équipe ait les mêmes. Il faut le committer, mais ne jamais le modifier à la main.

### `capacitor.config.ts` : la config Android
```ts
appId: 'com.mrrojo.objetsperdus',  // identifiant unique de l'application sur Android
appName: 'Objets Perdus',          // nom affiché sous l'icône
webDir: 'dist'                     // dossier produit par « ionic build », que Capacitor met dans l'APK
```
⚠️ `appId` doit être **identique** à celui déclaré dans la console Firebase (application Android).

### `vite.config.ts`
La config de **Vite**, l'outil qui compile et sert l'application. On y trouve notamment l'alias `@` → `src/`. On n'y touchera pas.

### `tsconfig.json`, `tsconfig.node.json`, `tsconfig.e2e.json`
La config de **TypeScript**. `"strict": true` signifie que TypeScript est sévère et signale beaucoup d'erreurs potentielles, ce qui est une bonne chose. On n'y touchera pas.

### `index.html`
La **seule** page HTML de l'application : Vue remplit le `<div id="app">`. On pourra y changer le `<title>Ionic App</title>`.

### Les autres
| Fichier | Rôle |
|---|---|
| `ionic.config.json` | Indique au CLI que c'est un projet Vue + Vite avec Capacitor |
| `eslint.config.js` | Règles de qualité du code (`npm run lint`) |
| `.prettierignore` | Fichiers que Prettier (formatage automatique) doit ignorer |
| `.editorconfig` | Indentation et encodage communs à tous les éditeurs |
| `.browserslistrc` | Versions de navigateurs supportées |
| `.gitignore` | Ce que Git ignore : `node_modules/`, `dist/`… |
| `.vscode/extensions.json` | Extensions VS Code recommandées (Vue - Official, ESLint, Prettier) |
| `playwright.config.ts`, `tests/` | Tests automatiques, hors du périmètre du TP |

---

## 6. Récapitulatif : ce qu'on va toucher

| Fichier / dossier | Fréquence |
|---|---|
| `src/views/`, `src/components/` | ⭐⭐⭐ Tout le temps |
| `src/router/index.ts` | ⭐⭐ Souvent |
| `src/main.ts` | ⭐ Une ou deux fois (Pinia) |
| `src/theme/variables.css` | ⭐ Pour les couleurs |
| `capacitor.config.ts`, `index.html` | Rarement |
| `package.json` | Automatiquement, via `npm install` |
| `node_modules/`, `package-lock.json`, `vite.config.ts`, `tsconfig*.json` | ❌ Jamais à la main |

**Nouveaux dossiers qu'on créera dans `src/` :** `types/`, `firebase/`, `services/`, `stores/`, `composables/`.

---

## 7. Exercice pour vérifier sa compréhension

1. Lancer `ionic serve`.
2. Dans `src/views/Tab1Page.vue`, remplacer les deux `Tab 1` par `Annonces`. La page se recharge toute seule.
3. Dans `src/views/TabsPage.vue`, changer le libellé `<ion-label>Tab 1</ion-label>` en `Annonces`.
4. Dans `src/router/index.ts`, trouver quelle ligne fait qu'on arrive sur l'onglet 1 au démarrage.
