# To-do : Objets perdus et trouvés

On coche au fur et à mesure. Chaque étape doit être comprise avant de passer à la suivante.

---

## Étape 0 : Créer le projet vierge

- [ ] Installer Ionic CLI : `sudo npm install -g @ionic/cli`
- [ ] Vérifier l'installation : `ionic -v` doit afficher un numéro de version
- [ ] Se placer dans le dossier du projet : `cd ~/S5/MrRojo/Projet-ionic`
- [ ] Créer l'application :
  ```bash
  ionic start objets-perdus tabs --type=vue --capacitor --package-id=com.mrrojo.objetsperdus --no-git
  ```
  - `tabs` : modèle de départ avec des onglets en bas de l'écran
  - `--type=vue` : on utilise Vue, pas Angular
  - `--capacitor` : prépare le passage vers Android
  - `--package-id` : l'identifiant unique de l'application Android (à adapter si besoin)
  - `--no-git` : Git existe déjà à la racine, on n'en crée pas un deuxième
  - Si Ionic demande de créer un compte Ionic, répondre **non**
- [ ] Entrer dans l'application : `cd objets-perdus`
- [ ] Lancer l'application : `ionic serve`, qui l'ouvre dans le navigateur
- [ ] Ouvrir les outils développeur (F12) et activer la vue mobile
- [ ] ✅ **Le projet vierge est prêt**

---

## Étape 1 : Comprendre le projet vierge

- [ ] Parcourir les dossiers : `src/`, `public/`, `node_modules/`, `package.json`, `capacitor.config.ts`
- [ ] Lire `src/main.ts` : comment l'application démarre
- [ ] Lire `src/router/index.ts` : comment on passe d'une page à l'autre
- [ ] Lire `src/views/Tab1Page.vue` : les trois parties d'un fichier `.vue` (`<template>`, `<script>`, `<style>`)
- [ ] Exercice : modifier le titre de Tab1 et voir l'application se recharger toute seule
- [ ] Installer Pinia : `npm install pinia`, puis le brancher dans `main.ts`

---

## Étape 2 : Les modèles (TypeScript)

- [ ] Créer `src/types/user.ts` avec l'interface `AppUser`
- [ ] Créer `src/types/item.ts` avec l'interface `Item` et les types `ItemType`, `ItemCategory`, `ItemStatus`
- [ ] Comprendre `interface`, les unions (`'perdu' | 'trouve'`) et les champs optionnels (`?`)

---

## Étape 3 : Configurer Firebase

**Sur la console** (https://console.firebase.google.com) :
- [ ] Créer le projet Firebase (Google Analytics : facultatif, on peut le désactiver)
- [ ] Ajouter une **application Web** et copier la config (`apiKey`, `authDomain`, `projectId`…)
- [ ] Activer **Authentication**, avec le fournisseur Email/Mot de passe
- [ ] Créer la base **Firestore** en mode production, région `europe-west`
- [ ] Coller les **règles de sécurité** (voir `CONCEPTION.md`)

**Dans le code :**
- [ ] `npm install firebase`
- [ ] Créer `.env` avec les variables `VITE_FIREBASE_*`
- [ ] Ajouter `.env` dans `.gitignore`
- [ ] Créer `src/firebase/config.ts` (`initializeApp`, export de `auth` et `db`)

---

## Étape 4 : Le login

- [ ] `src/services/auth.service.ts` : `register()`, `login()`, `logout()`
- [ ] `src/stores/auth.store.ts` : l'utilisateur courant avec `onAuthStateChanged`
- [ ] À l'inscription, créer le document `users/{uid}`
- [ ] Page `LoginPage.vue` : email, mot de passe, bouton, message d'erreur
- [ ] Page `RegisterPage.vue` : nom, email, téléphone, mot de passe
- [ ] Ajouter les routes `/login` et `/register`
- [ ] Ajouter le **guard** dans le routeur : redirection vers `/login` si l'utilisateur n'est pas connecté
- [ ] Ajouter le bouton de déconnexion dans `ProfilePage.vue`
- [ ] Test : inscription, déconnexion, reconnexion, puis tentative d'accès sans être connecté

---

## Étape 5 : La liste et la fiche

- [ ] Ajouter 3 ou 4 annonces **à la main** dans la console Firestore (avec des URL d'images d'exemple)
- [ ] `src/services/item.service.ts` : `listenItems()` (temps réel) et `getItem(id)`
- [ ] `src/stores/item.store.ts` : la liste, les filtres et `mesAnnonces`
- [ ] Composants `ItemCard.vue` et `StatusBadge.vue`
- [ ] Page `ItemListPage.vue` : liste, segment Perdu/Trouvé, recherche, filtre par catégorie
- [ ] Page `ItemDetailPage.vue` (route `/items/:id`) : photo, infos, bouton « Contacter »
- [ ] Renommer et organiser les onglets : Annonces, Publier, Mes annonces, Profil
- [ ] Test : la liste se met à jour toute seule quand on modifie Firestore dans la console

---

## Étape 6 : Le formulaire et la photo

**Cloudinary :**
- [ ] Créer un compte gratuit sur https://cloudinary.com
- [ ] Créer un **upload preset non signé**
- [ ] Ajouter `VITE_CLOUDINARY_CLOUD_NAME` et `VITE_CLOUDINARY_PRESET` dans `.env`

**Dans le code :**
- [ ] `npm install @capacitor/camera`, puis `npx cap sync`
- [ ] `src/composables/useCamera.ts` : photo depuis la caméra ou la galerie
- [ ] `src/services/upload.service.ts` : envoi à Cloudinary, qui renvoie l'URL
- [ ] Compléter `item.service.ts` avec `addItem()`, `updateItem()`, `markResolved()` et `deleteItem()`
- [ ] Page `ItemFormPage.vue` : type, titre, description, catégorie, lieu, date, photo, plus la validation
- [ ] Page `MyItemsPage.vue` : modifier, supprimer, marquer comme résolu
- [ ] Sur la fiche, afficher « Marquer résolu » seulement si l'utilisateur est l'auteur
- [ ] Test : publier une annonce avec photo et la voir apparaître dans la liste

---

## Étape 7 : Android et notifications

**Préparer la machine :**
- [ ] Installer **Android Studio** (il fournit le SDK Android et Gradle)
- [ ] Installer un **JDK 17 ou 21** (la version actuelle est Java 8, trop ancienne)
- [ ] Activer le **mode développeur** et le **débogage USB** sur le téléphone

**Android :**
- [ ] `ionic build`
- [ ] `npm install @capacitor/android`, puis `npx cap add android`
- [ ] Console Firebase : ajouter une **application Android** avec le même `package-id`
- [ ] Télécharger `google-services.json` et le placer dans `android/app/`
- [ ] Lancer l'application sur le téléphone : `npx cap run android`

**Notifications :**
- [ ] `npm install @capacitor/push-notifications @capacitor/local-notifications`, puis `npx cap sync`
- [ ] Vérifier la permission `POST_NOTIFICATIONS` dans `AndroidManifest.xml`
- [ ] `src/services/notification.service.ts` : demander la permission et récupérer le token
- [ ] Enregistrer le `fcmToken` dans `users/{uid}` à chaque connexion
- [ ] Notification locale « Annonce publiée ✅ » après une publication
- [ ] Test : envoyer un push depuis la console Firebase (Messaging, puis message de test avec le token)

---

## Étape 8 : Tests sur un vrai téléphone

- [ ] Inscription, déconnexion puis reconnexion
- [ ] Redirection vers le login si l'utilisateur n'est pas connecté
- [ ] Publication avec une photo prise à la caméra, puis une autre avec une photo de la galerie
- [ ] Mise à jour en temps réel sur un 2ᵉ appareil
- [ ] Filtres Perdu/Trouvé et catégorie
- [ ] Un autre utilisateur ne peut pas modifier mon annonce
- [ ] Le push envoyé depuis la console est bien reçu
- [ ] Le passage en « Résolu » est visible dans la liste

---

## Étape 9 : Livrable APK

- [ ] Choisir l'icône et le nom de l'application (optionnel : `@capacitor/assets`)
- [ ] `ionic build && npx cap sync android`
- [ ] `cd android && ./gradlew assembleDebug`
- [ ] Récupérer `android/app/build/outputs/apk/debug/app-debug.apk`
- [ ] Installer cet APK sur un téléphone « propre » pour vérifier qu'il fonctionne
- [ ] (Optionnel) Générer un APK signé avec Android Studio : *Build → Generate Signed APK*
- [ ] 🎉 Rendre l'APK
