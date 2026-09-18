# VisualChord Display

Application web (HTML/CSS/JavaScript, sans dépendance externe) qui affiche le doigté d'un accord de guitare (manche), d'ukulélé (manche) ou de piano (clavier, main gauche / main droite) à partir de son nom, avec une fonction d'impression.

Elle fonctionne dans n'importe quel navigateur moderne — Windows, macOS, iPhone, iPad, Android — sans installation.

**Assistant de saisie :** juste au-dessus des exemples cliquables, un petit bloc « Assistant de saisie » permet de construire un accord sans avoir à le taper. Touchez (ou cliquez) une note, puis un type — Majeur, Mineur, 7 ou Basse — puis une précision : pour Majeur/Mineur/7, une liste complète reprenant tout le vocabulaire reconnu par l'application (triade simple, 6, 6/9, add9, sus2, sus4, maj7, maj9, maj11, maj13, dim, dim7, m7b5, m9, m11, mmaj7, 9, 11, 13, 7sus4, etc.) ; pour Basse, la liste des notes pour construire un accord renversé (ex. C/E). Le bouton « ‹ Retour » à chaque étape permet de revenir en arrière, et le champ de saisie reste modifiable normalement une fois l'accord inséré. Fonctionne entièrement au tap/clic (aucun survol nécessaire), donc identique sur ordinateur comme sur mobile ou tablette.

**Suites d'accords :** saisissez plusieurs accords séparés par des espaces (ex. `C G Am F`) pour afficher toute une grille — pratique pour une feuille d'accords de morceau, imprimable directement. Utilisez `|` pour séparer les mesures (ex. `C G | Am F | C`) : une barre verticale s'affiche alors entre chaque groupe, comme sur une vraie grille d'accords. Cliquez sur un accord de la suite pour l'afficher en grand (avec les différentes positions possibles à la guitare). Le bouton « Partager le lien » copie une adresse qui rouvre directement cette suite précise, mesures comprises.

**Écouter les accords :** un bouton 🔊 permet d'entendre le son de l'accord affiché. Le timbre s'adapte automatiquement à l'instrument sélectionné : un son de guitare ou d'ukulélé (même modèle physique de corde pincée façon nylon, avec caisse de résonance et petite réverbération de pièce, mais accordé plus haut et à la caisse plus petite pour l'ukulélé) en mode manche, ou un son de piano (attaque franche, résonance) en mode clavier. Dans une suite d'accords, chaque mini-diagramme possède son propre bouton d'écoute, et le bouton « Écouter la suite » joue l'ensemble des accords les uns après les autres avec surbrillance de l'accord en cours — pratique pour vérifier une grille à l'oreille avant de la jouer.

**Portée musicale :** dans la zone de résultat, juste à côté de l'étiquette d'instrument, un menu déroulant — « Diagramme / Afficher la portée » pour un accord seul, « Carte / Grille jazz / Tablature / Afficher la portée » pour une suite d'accords — remplace le diagramme habituel (manche ou clavier, ou la grille de cartes) par une véritable portée façon partition avec les notes de l'accord — présentée comme sur une partition imprimée : chiffrage 4/4, notes en rondes noir sur blanc (sans encadré ni couleur), barre de mesure finale. Pour la guitare et l'ukulélé, une seule portée en clé de sol, avec les notes réellement produites par le doigté affiché sur le manche — cordes à vide comprises, cordes étouffées exclues — et non un simple empilement théorique. Pour la guitare spécifiquement, l'affichage suit la convention habituelle des partitions imprimées : les notes sont écrites une octave au-dessus du son réel (comme pour une contrebasse), afin d'éviter une portée surchargée de lignes supplémentaires en dessous — cette convention ne concerne que l'affichage de la portée, jamais le son joué par le bouton d'écoute, et ne s'applique pas à l'ukulélé (dont l'accordage plus aigu se place déjà naturellement sur la portée). Pour le piano, une double portée (clé de sol / clé de fa) reliée par une accolade. La correspondance main gauche / main droite du piano (même code couleur que le clavier) reste indiquée sous la portée, dans la légende. Pour une suite d'accords, la portée devient continue : plus de case par accord, tous les accords s'enchaînent sur une seule portée (ou plusieurs, à la ligne, si la suite est longue), chaque mesure étant séparée par une simple barre verticale — comme sur une vraie partition. Par défaut, chaque accord forme sa propre mesure (ronde) ; utilisez `|` dans la saisie pour regrouper plusieurs accords dans une même mesure, avec la valeur rythmique adaptée automatiquement : deux accords dans une mesure sont des blanches, trois accords sont deux noires suivies d'une blanche, et quatre accords sont quatre noires — toujours pour un total de quatre temps. Cliquez (ou naviguez au clavier) sur une mesure de la portée pour ouvrir cet accord en grand, exactement comme avec les mini-cartes. Pour une suite d'accords, le même menu déroulant propose aussi la « Grille jazz » (disposition en losange façon set-list de jazz manuscrite, un accord par côté) comme alternative à la grille de cartes classique. Les deux menus (accord seul et suite d'accords) restent synchronisés — passer en portée sur l'un active automatiquement la portée sur l'autre. Réglage par défaut : diagramme / cartes.

**Tablature :** toujours dans ce même menu déroulant d'une suite d'accords, l'option « Tablature » remplace la grille de cartes par une véritable tablature continue (guitare ou ukulélé uniquement) : les cordes sont représentées par des lignes horizontales — de la plus aiguë en haut à la plus grave en bas, comme sur une tablature imprimée — et chaque accord affiche le numéro de frette à jouer sur chaque corde (`x` pour une corde étouffée), directement au-dessus du doigté réellement utilisé sur le manche. Comme pour la portée, chaque mesure est séparée par une barre verticale, la tablature s'enchaîne sur plusieurs lignes si la suite est longue, et cliquer sur un accord de la tablature l'ouvre en grand. Cette option n'a pas de sens pour le piano : un message l'indique si le piano est sélectionné.

Interface disponible en 6 langues (sélecteur en haut à droite de l'application) : français, anglais, allemand, italien, espagnol, chinois. Le choix est mémorisé automatiquement dans le navigateur.

## Contenu du dossier

- `index.html` — l'application complète (interface + moteur de reconnaissance d'accords + génération des diagrammes en SVG). Le logo ShareVisual y est déjà intégré (encodé directement dans le fichier).
- `guitar-chords.json` — base de doigtés de guitare (env. 2000 positions, issue du projet open source `@tombatossals/chords-db`).
- `ukulele-chords.json` — base de doigtés d'ukulélé (même projet open source, accordage standard sol-do-mi-la).
- `ShareVisual Logo vertical.png` — fichier logo d'origine, conservé pour référence.
- `manifest.json` — descripteur d'application web (nom, couleurs, icônes) utilisé par iOS/Android pour l'ajout à l'écran d'accueil.
- `apple-touch-icon.png` (+ variantes 167/152) — icône utilisée par iPhone/iPad quand vous ajoutez l'application à l'écran d'accueil.
- `icon-192.png`, `icon-512.png` — icônes utilisées par Android/Chrome (via `manifest.json`).
- `favicon-16.png`, `favicon-32.png` — icône affichée dans l'onglet du navigateur.

## Utiliser l'application

### Option 1 — en ligne (recommandé)
Le lien fourni dans la conversation Claude — ou l'adresse GitHub Pages une fois déployée — fonctionne immédiatement sur tous vos appareils (Windows, macOS, iPhone, iPad, Android) : il suffit de l'ouvrir dans un navigateur.

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

Sans serveur local, l'application fonctionne quand même mais bascule sur des doigtés de guitare et d'ukulélé générés automatiquement (moins « traditionnels ») car elle ne peut pas charger `guitar-chords.json` ni `ukulele-chords.json`.

### Option 3 — héberger l'application
Déposez simplement `index.html`, `guitar-chords.json` et `ukulele-chords.json` sur n'importe quel hébergement web statique (Netlify, GitHub Pages, Vercel, votre propre serveur…) : aucune configuration serveur particulière n'est requise (le logo est déjà intégré dans le HTML, pas besoin de fichier séparé).

## Aller plus loin : une « vraie » application native (Windows / macOS / iPhone / iPad / Android)

Cette page fonctionne déjà telle quelle dans les navigateurs de tous ces systèmes. Si vous souhaitez un jour une icône installable indépendante (App Store, Google Play, .exe, .dmg), la voie la plus simple à partir de ce même code est **Capacitor** (par Ionic), qui empaquette une page web dans une coquille native sans réécrire l'application :

```bash
npm install -g @capacitor/cli
mkdir visualchord-app && cd visualchord-app
npm init -y
npm install @capacitor/core @capacitor/ios @capacitor/android @capacitor/electron
npx cap init "VisualChord Display" "com.sharevisual.visualchord"
# copiez index.html, guitar-chords.json et ukulele-chords.json dans le dossier "www"
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
