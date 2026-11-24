# Projet Ionic : Objets perdus et trouvés

Document de conception : consignes, choix techniques et plan de réalisation.

---

## 1. Consignes du TP

**TP : créer une application mobile.** Le thème est au choix, sauf celui du TP fait en cours. L'application contient :

1. Un login avec Firebase
2. Une liste et une fiche
3. Un formulaire avec upload d'image
4. Des notifications et un backend sur Firebase

**Livrable :** le fichier APK.

**Thème retenu : Objets perdus et trouvés (campus).** Les étudiants publient un objet perdu ou trouvé, avec une photo, un lieu et un contact.

---

## 2. Stack technique

### 2.1 Le socle

| Techno | Rôle |
|---|---|
| **Ionic 8 (`@ionic/vue`)** | Les composants d'interface mobile (listes, cartes, formulaires, onglets) |
| **Vue 3** (Composition API, `<script setup lang="ts">`) | Le framework |
| **TypeScript** | Le langage |
| **Vite** | Le bundler, configuré par `ionic start` |
| **`@ionic/vue-router`** (sur Vue Router 4) | La navigation, avec les transitions mobiles |
| **Pinia** | Le store global : utilisateur connecté, liste des annonces |
| **Capacitor** | Le pont vers l'Android natif : caméra, notifications, projet Android |

### 2.2 Firebase et services

| Consigne | Service | Librairie / import |
|---|---|---|
| 1. Login | Firebase Authentication (email/mot de passe) | `firebase/auth` |
| 2. Liste et fiche | Cloud Firestore | `firebase/firestore` |
| 3. Upload d'image | **Cloudinary**, dont l'URL est stockée dans Firestore, avec `@capacitor/camera` pour la photo | `fetch` vers l'API Cloudinary |
| 4. Notifications | Firebase Cloud Messaging (FCM) et notifications locales | `@capacitor/push-notifications`, `@capacitor/local-notifications` |
| 4. Backend | Firestore, Auth et règles de sécurité | — |

On utilise directement le SDK Firebase modulaire (`npm i firebase`). VueFire est optionnel et n'est pas utilisé ici.

### 2.3 Ce qu'il faut installer

**Sur la machine (une seule fois) :**
- **Node.js** (LTS)
- **Ionic CLI** : `npm install -g @ionic/cli`
- **Android Studio**, qui fournit le SDK Android et Gradle
- **JDK 17 ou 21**, généralement déjà embarqué dans Android Studio
- *(Optionnel)* **Firebase CLI** : `npm install -g firebase-tools`, utile seulement pour les Cloud Functions ou pour déployer les règles depuis des fichiers

**Dans le projet (avec npm, rien à télécharger à part) :**
- Capacitor est installé automatiquement par `ionic start ... --capacitor`. Sinon, la commande est `npm install @capacitor/core @capacitor/cli @capacitor/android`.
- Pour Firebase : `npm install firebase`
- Pour les plugins : `npm install @capacitor/camera @capacitor/push-notifications @capacitor/local-notifications`, puis `npx cap sync`
- Pour le store : `npm install pinia`

**Création du projet :**
```bash
ionic start objets-perdus tabs --type=vue --capacitor
```

### 2.4 Configuration sur la console Firebase

Sur https://console.firebase.google.com :
1. Créer le projet.
2. Ajouter une **application Web** et récupérer la config (`apiKey`, `projectId`…) pour le fichier `.env`.
3. Ajouter une **application Android** avec le même identifiant que `appId` dans `capacitor.config.ts`, par exemple `com.monnom.objetsperdus`.
4. Télécharger **`google-services.json`** et le placer dans `android/app/`. Il est indispensable pour FCM.
5. Activer **Authentication** (email/mot de passe), **Firestore** et **Cloud Messaging**.

---

## 3. Gratuité : rester sur le plan Spark

| Service | Gratuit (Spark) ? | Remarque |
|---|---|---|
| Authentication | ✅ | Largement suffisant |
| Firestore | ✅ | 1 Go stocké, 50 000 lectures et 20 000 écritures par jour |
| Cloud Messaging | ✅ | Toujours gratuit |
| Envoi de notifications depuis la console | ✅ | — |
| **Storage** | ❌ | Les nouveaux projets doivent être en Blaze (carte bancaire) |
| **Cloud Functions** | ❌ | Blaze obligatoire |

Le plan Blaze exige une carte bancaire et n'a **pas de plafond automatique** : les alertes de budget préviennent mais ne bloquent pas.

**Choix retenu (100 % gratuit) :**
- Les images sont stockées sur **Cloudinary** (offre gratuite, sans carte), avec un « upload preset » non signé. Seule l'URL est enregistrée dans Firestore.
- Les notifications sont envoyées en push depuis la **console Firebase**, en complément de notifications **locales** déclenchées par l'application.
- **Limite :** sans Cloud Functions, l'application ne peut pas envoyer automatiquement un push aux autres utilisateurs quand une annonce est ajoutée.
- ⚠️ **À vérifier avec le prof :** est-ce que Cloudinary est accepté pour les images, puisque la consigne dit « backend sur Firebase » ?

---

## 4. Plan de réalisation : « Mais Dis, Comment Il Tiendra Vraiment »

Interprétation des initiales retenue : **M**odèle → **D**onnées → **C**ontrôle → **I**nterface → **T**ests → **V**ersion finale.

### M : Modèle (les données)

**Collection `users/{uid}`**
```ts
interface AppUser {
  uid: string;
  displayName: string;
  email: string;
  phone?: string;        // contact affiché sur les annonces
  fcmToken?: string;     // pour les notifications push
  createdAt: Timestamp;
}
```

**Collection `items/{itemId}`**
```ts
interface Item {
  id: string;
  type: 'perdu' | 'trouve';
  title: string;            // « Clé USB rouge »
  description: string;
  category: 'electronique' | 'papiers' | 'vetements' | 'cles' | 'autre';
  location: string;         // « Bâtiment B, salle 12 »
  date: Timestamp;          // date de la perte ou de la découverte
  imageUrl: string;         // URL Cloudinary
  status: 'ouvert' | 'resolu';
  ownerId: string;          // uid de l'auteur
  ownerName: string;
  ownerContact: string;
  createdAt: Timestamp;
}
```

### D : Données (l'accès à Firebase)

Ce sont les seuls fichiers qui importent Firebase.

| Fichier | Fonctions |
|---|---|
| `firebase/config.ts` | `initializeApp`, export de `auth` et `db` |
| `services/auth.service.ts` | `register()`, `login()`, `logout()`, `onUserChange()` |
| `services/item.service.ts` | `listenItems(filtre)`, `getItem(id)`, `addItem()`, `updateItem()`, `markResolved()`, `deleteItem()` |
| `services/upload.service.ts` | `uploadImage(blob)`, qui renvoie l'URL Cloudinary |
| `services/notification.service.ts` | Permission, récupération du token FCM, notifications locales |

**Règles de sécurité Firestore :**
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /items/{id} {
      allow read: if request.auth != null;
      allow create: if request.auth != null
                    && request.resource.data.ownerId == request.auth.uid;
      allow update, delete: if request.auth != null
                    && resource.data.ownerId == request.auth.uid;
    }
    match /users/{uid} {
      allow read: if request.auth != null;
      allow write: if request.auth != null && request.auth.uid == uid;
    }
  }
}
```
Avec ces règles, seul l'auteur d'une annonce peut la modifier ou la supprimer.

### C : Contrôle (la logique)

- **`stores/auth.store.ts`** (Pinia) : l'utilisateur courant, mis à jour par `onAuthStateChanged`.
- **`stores/item.store.ts`** : la liste en temps réel (`onSnapshot`), les filtres (type, catégorie, recherche) et la valeur calculée `mesAnnonces`.
- **`composables/useCamera.ts`** : prendre une photo ou en choisir une dans la galerie, puis la compresser.
- **`router/index.ts`** : un `beforeEach` qui redirige vers `/login` si l'utilisateur n'est pas connecté.

**Règle d'architecture :** les pages ne parlent jamais directement à Firebase. Les pages s'occupent de l'affichage, les stores de l'état partagé, et les services de l'accès à Firebase.

### I : Interface (les écrans)

```
Login ─┬─ Register
       ▼
[Onglets]
├── 🔍 Annonces     → liste + segment Perdu/Trouvé + recherche + filtre catégorie
│        └── Fiche  → photo, infos, bouton « Contacter », « Marquer résolu » si auteur
├── ➕ Publier      → formulaire + photo (caméra/galerie)
├── 📋 Mes annonces → modifier / supprimer / résoudre
└── 👤 Profil       → infos, déconnexion
```

**Composants réutilisables :** `ItemCard.vue` (vignette, titre, badge Perdu/Trouvé, lieu) et `StatusBadge.vue`.

**Notifications :**
- Une notification locale « Annonce publiée ✅ » après l'ajout.
- Des notifications push envoyées depuis la console Firebase, par exemple « Pensez à vérifier les objets trouvés cette semaine ».

**Particularités de Vue à retenir :**
- Chaque vue doit être enveloppée dans `<ion-page>`, sinon la navigation casse.
- Les composants Ionic (`IonPage`, `IonContent`, `IonList`…) s'importent explicitement dans chaque fichier `.vue`.
- Pour une liste temps réel, combiner `onSnapshot()` avec un `ref([])`, et se désabonner dans `onUnmounted`.

### T : Tests (sur un vrai téléphone)

- [ ] Inscription, déconnexion puis reconnexion
- [ ] Accès à `/items` sans être connecté : redirection vers le login
- [ ] Publication avec une photo prise à la caméra, puis une autre avec une photo de la galerie
- [ ] L'annonce apparaît en temps réel sur un 2ᵉ appareil
- [ ] Les filtres Perdu/Trouvé et catégorie fonctionnent
- [ ] Un autre utilisateur ne peut pas modifier mon annonce (règles Firestore)
- [ ] La notification push envoyée depuis la console est bien reçue
- [ ] Le passage en « Résolu » se voit dans la liste

Les notifications push ne fonctionnent pas dans le navigateur : il faut tester sur un téléphone.

### V : Version finale (l'APK)

Permissions Android à vérifier dans `AndroidManifest.xml` : caméra et `POST_NOTIFICATIONS` (Android 13 et plus).

```bash
ionic build
npx cap sync android
cd android && ./gradlew assembleDebug
```
L'APK se trouve ensuite dans `android/app/build/outputs/apk/debug/app-debug.apk`.

Pour un APK signé, passer par Android Studio : *Build → Generate Signed APK*.

---

## 5. Arborescence du projet

```
src/
├── firebase/
│   └── config.ts
├── services/
│   ├── auth.service.ts
│   ├── item.service.ts
│   ├── upload.service.ts
│   └── notification.service.ts
├── stores/
│   ├── auth.store.ts
│   └── item.store.ts
├── composables/
│   └── useCamera.ts
├── types/
│   └── item.ts
├── views/
│   ├── LoginPage.vue
│   ├── RegisterPage.vue
│   ├── TabsPage.vue
│   ├── ItemListPage.vue
│   ├── ItemDetailPage.vue     (route /items/:id)
│   ├── ItemFormPage.vue       (ajout / édition + photo)
│   ├── MyItemsPage.vue
│   └── ProfilePage.vue
├── components/
│   ├── ItemCard.vue
│   └── StatusBadge.vue
├── router/
│   └── index.ts
└── main.ts
.env                           VITE_FIREBASE_API_KEY=…, VITE_CLOUDINARY_CLOUD_NAME=…
```

---

## 6. Ordre de travail

1. Initialiser le projet, configurer Firebase et le login (**M, D, C**)
2. Construire la liste et la fiche, avec des données ajoutées à la main dans la console
3. Ajouter le formulaire, la caméra et l'upload Cloudinary
4. Mettre en place les notifications (FCM et locales)
5. Faire les tests, puis générer l'APK
