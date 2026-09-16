# VisualChord Display

Application web (HTML/CSS/JavaScript, sans dépendance externe) qui affiche le doigté d'un accord de guitare (manche) ou de piano (clavier, main gauche / main droite) à partir de son nom, avec une fonction d'impression.

Elle fonctionne dans n'importe quel navigateur moderne — Windows, macOS, iPhone, iPad, Android — sans installation.

**Suites d'accords :** saisissez plusieurs accords séparés par des espaces (ex. `C G Am F`) pour afficher toute une grille — pratique pour une feuille d'accords de morceau, imprimable directement. Utilisez `|` pour séparer les mesures (ex. `C G | Am F | C`) : une barre verticale s'affiche alors entre chaque groupe, comme sur une vraie grille d'accords. Cliquez sur un accord de la suite pour l'afficher en grand (avec les différentes positions possibles à la guitare). Le bouton « Partager le lien » copie une adresse qui rouvre directement cette suite précise, mesures comprises.

Interface disponible en 6 langues (sélecteur en haut à droite de l'application) : français, anglais, allemand, italien, espagnol, chinois. Le choix est mémorisé automatiquement dans le navigateur.

## Contenu du dossier

- `index.html` — l'application complète (interface + moteur de reconnaissance d'accords + génération des diagrammes en SVG). Le logo ShareVisual y est déjà intégré (encodé directement dans le fichier).
- `guitar-chords.json` — base de doigtés de guitare (env. 2000 positions, issue du projet open source `@tombatossals/chords-db`).
- `ShareVisual Logo vertical.png` — fichier logo d'origine, conservé pour référence.
- `manifest.json` — descripteur d'application web (nom, couleurs, icônes) utilisé par iOS/Android pour l'ajout à l'écran d'accueil.
- `apple-touch-icon.png` (+ variantes 167/152) — icône utilisée par iPhone/iPad quand vous ajoutez l'application à l'écran d'accueil.
- `icon-192.png`, `icon-512.png` — icônes utilisées par Android/Chrome (via `manifest.json`).
- `favicon-16.png`, `favicon-32.png` — icône affichée dans l'onglet du navigateur.

## Utiliser l'application

### Option 1 — en ligne (recommandé)
L'adresse GitHub Pages une fois déployée — fonctionne immédiatement sur tous vos appareils (Windows, macOS, iPhone, iPad, Android) : il suffit de l'ouvrir dans un navigateur.

**Ajouter une icône sur l'écran d'accueil de l'iPhone/iPad :**
1. Ouvrez l'adresse du site dans **Safari** (l'ajout à l'écran d'accueil depuis Chrome sur iOS ne propose pas cette option).
2. Appuyez sur le bouton **Partager** (le carré avec une flèche vers le haut, dans la barre du bas).
3. Faites défiler et choisissez **« Sur l'écran d'accueil »**.
4. Confirmez le nom (« VisualChord ») et appuyez sur **Ajouter**.

Une icône avec le pictogramme ShareVisual apparaît alors sur l'écran d'accueil, et l'application s'ouvre en plein écran, sans la barre d'adresse de Safari, comme une vraie application installée.

### Option 2 — en local sur votre ordinateur
Les navigateurs bloquant par sécurité la lecture d'un fichier JSON local ouvert directement (double-clic), il faut lancer un petit serveur local dans ce dossier :

```bash
# avec Python (déjà installé sur macOS/Linux)
python3 -m http.server 8000
# puis ouvrir http://localhost:8000 dans le navigateur
```

```bash
# ou avec Node.js
npx serve .
```

Sous Windows, si Python est installé : `python -m http.server 8000`, ou utilisez l'extension « Live Server » de VS Code.

Sans serveur local, l'application fonctionne quand même mais bascule sur des doigtés de guitare générés automatiquement (moins « traditionnels ») car elle ne peut pas charger `guitar-chords.json`.

### Option 3 — héberger l'application
Déposez simplement `index.html` et `guitar-chords.json` sur n'importe quel hébergement web statique (Netlify, GitHub Pages, Vercel, votre propre serveur…) : aucune configuration serveur particulière n'est requise (le logo est déjà intégré dans le HTML, pas besoin de fichier séparé).

## Aller plus loin : une « vraie » application native (Windows / macOS / iPhone / iPad / Android)

Cette page fonctionne déjà telle quelle dans les navigateurs de tous ces systèmes. Si vous souhaitez un jour une icône installable indépendante (App Store, Google Play, .exe, .dmg), la voie la plus simple à partir de ce même code est **Capacitor** (par Ionic), qui empaquette une page web dans une coquille native sans réécrire l'application :

```bash
npm install -g @capacitor/cli
mkdir visualchord-app && cd visualchord-app
npm init -y
npm install @capacitor/core @capacitor/ios @capacitor/android @capacitor/electron
npx cap init "VisualChord Display" "com.sharevisual.visualchord"
# copiez index.html et guitar-chords.json dans le dossier "www"
npx cap add ios       # nécessite Xcode sur macOS
npx cap add android   # nécessite Android Studio
npx cap add electron  # pour Windows/macOS en application de bureau
npx cap sync
```

Vous obtenez alors des projets Xcode / Android Studio / Electron prêts à compiler et signer pour publication sur l'App Store, le Play Store, ou en exécutable Windows/macOS — cette étape de compilation et de signature nécessite votre propre matériel (un Mac pour iOS) et vos comptes développeur, elle n'est donc pas réalisable depuis cet environnement.

## Vocabulaire d'accords reconnu

Notes : `A B C D E F G`, avec `#` (dièse) ou `b` (bémol) — ex. `C`, `F#`, `Bb`.

Types : accords de base (`m`, `5`, `sus2`, `sus4`, `aug`, `dim`), septièmes (`7`, `maj7`, `m7`, `dim7`, `m7b5`), accords étendus (`9`, `11`, `13`, `maj9`, `m9`…), ajouts et sixtes (`6`, `6/9`, `add9`), et accords de basse alternative (`C/E`, `G/B`…).

Un panneau d'aide intégré à l'application liste des exemples complets.
