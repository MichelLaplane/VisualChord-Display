# VisualChord Display — Principes de conception

Ce document résume les principes et décisions de conception établis au fil du
développement de l'application, en particulier le moteur de détection
d'accords à partir d'un fichier audio. Il est destiné à servir de mémoire
technique pour les évolutions futures, indépendamment de l'historique de
conversation (qui n'est pas exportable depuis cette session).

Portée : ce document couvre ce qui a été explicitement discuté et décidé dans
les échanges disponibles à la rédaction de ce fichier (version 1.7.1), pas
nécessairement la toute première genèse de l'application.

## 1. Principes généraux de l'application

- **Fichier unique auto-suffisant.** `index.html` est un seul fichier HTML/CSS/
  JS vanilla (pas de framework, pas de dépendance externe au runtime autre que
  les polices Google Fonts). Tout le code JS est dans une IIFE. Ce choix
  permet de publier l'app comme Artifact Claude et de la distribuer comme un
  simple fichier à ouvrir dans un navigateur.
- **Multi-instrument et multilingue.** L'app gère guitare/ukulélé/piano et
  plusieurs langues (FR/EN/DE/IT/ES/ZH) via un objet `LANG_DATA` et une
  fonction `applyStaticTranslations()`. Toute nouvelle chaîne visible doit
  être ajoutée dans les 6 blocs de langue, jamais câblée en dur dans le HTML.
- **Pas de bibliothèque audio externe.** La FFT (`fftInPlace`), le fenêtrage
  (`hannWindow`), le downsampling (`decimateAverage`) et toute l'analyse
  spectrale sont écrits à la main en JS pur — cohérent avec le choix de
  fichier unique sans dépendance.
- **Versionning explicite.** `APP_VERSION` est bumpé à chaque changement
  significatif et affiché dans l'UI (`version-badge`), ce qui a servi de point
  de vérification systématique dans les tests automatisés (Playwright) pour
  confirmer qu'on teste bien la bonne version du fichier.

## 2. Principes de génération des grilles d'accords (diagrammes de manche)

Le terme « grille » désigne ici le diagramme visuel d'un accord sur le
manche (guitare/ukulélé) ou le clavier — ce que `renderFretboardSVG` /
`renderPianoSVG` dessinent.

**Base de données d'abord, génération algorithmique en secours.**
Pour guitare et ukulélé, `findFrettedPositions` cherche d'abord le doigté
dans une base de données JSON chargée au démarrage (`guitar-chords.json` /
`ukulele-chords.json`, via `GUITAR_DB`/`UKULELE_DB`) — des doigtés réels,
choisis à la main, pour les accords usuels. Si l'accord demandé n'y figure
pas, `fallbackFrettedPositions` **génère** un doigté par recherche
exhaustive : pour chaque corde, toutes les frettes 0 à 4 (plus « étouffée »)
qui tombent sur une note de l'accord sont candidates, puis toutes les
combinaisons possibles sur l'ensemble des cordes sont notées selon :
- le nombre de notes de l'accord manquantes (le plus pénalisant, de loin) ;
- le nombre de cordes étouffées ;
- si la note la plus grave jouée est bien la fondamentale de l'accord ;
- l'étalement des frettes utilisées (compacité du doigté) ;
- la somme des numéros de frette (préférence pour les positions basses du
  manche).

**Principe retenu :** ce classement garantit qu'un nom d'accord valide
produit TOUJOURS un diagramme jouable, même absent de la base, sans jamais
avoir besoin d'étoffer la base à la main pour chaque nouvel accord possible.
Le résultat généré (`{frets, fingers, baseFret, barres, generated:true}`)
a exactement la même forme qu'un doigté venu de la base — le moteur de
rendu SVG ne sait pas, et n'a pas besoin de savoir, lequel des deux il
dessine.

**Rendu du diagramme.** `renderFretboardSVG` est un dessin SVG purement
géométrique (cordes = lignes verticales, frettes = lignes horizontales,
cases proportionnées à l'espacement des cordes) : cercle plein pour une
note tenue (avec le numéro de doigt si connu), cercle vide au-dessus du
manche pour une corde à vide, croix pour une corde étouffée, barre épaisse
pour un barré (bout carré s'il couvre tout le manche, arrondi sinon pour
éviter la collision avec l'étiquette « Nfr » du numéro de frette de départ).

## 3. Principes des gammes et des arpèges

**Unification par une forme de données commune.** La vue « Gammes » et la
vue « Arpèges » partagent exactement la même structure de données en sortie
— `{scaleKey, intervals, pitchClasses}` — produite soit par
`chordScaleInfo()` (une gamme de référence), soit par `chordArpeggioInfo()`
(les notes propres de l'accord). `scaleOrArpeggioInfo(p, mode)` est le seul
point d'aiguillage entre les deux. Grâce à cette forme commune, absolument
tout le reste du pipeline — placement sur le manche
(`scaleNeckPositions`), rendu SVG (`renderNeckScaleSVG`,
`renderPianoScaleSVG`), lecture audio (`buildChordScaleSchedule`) — marche
à l'identique pour une gamme ou un arpège, sans un seul `if (mode===...)`
dispersé dans le reste du code.

**Choix de la gamme associée à un accord.** `QUALITY_TO_SCALE` associe
chacune des 41 qualités d'accord à l'une de 12 gammes de référence
(`SCALE_DEFS`), choisies pour que l'ensemble des notes de la gamme soit
toujours un **sur-ensemble strict** des notes de l'accord — chaque note de
l'accord doit obligatoirement se retrouver dans la gamme dessinée à côté.
Une option « pentatonique » (`SCALE_TO_PENTATONIC`) peut ensuite substituer
une gamme pentatonique majeure/mineure proche. Une seule approximation
documentée dans le code : `mmaj7b5` n'a pas d'équivalent 7 notes exact
(racine + tierce mineure + quinte diminuée + septième majeure en même
temps n'existe dans aucune gamme standard), elle emprunte donc la gamme
mineure mélodique déjà utilisée pour le reste de la famille `mmaj7`.

**L'arpège commence toujours sur la fondamentale, pas la gamme par
défaut.** Sur le manche, une gamme « montante, dans l'ordre » n'est pas
garantie par construction : une corde grave peut atteindre, dans la même
fenêtre de frettes, une note de la gamme plus grave que la fondamentale
elle-même (p. ex. en do majeur, la 1ère frette de la corde de Mi grave
donne un Fa qui est dans la gamme mais sous le do le plus grave accessible
ailleurs). **Principe retenu :** `scaleNeckPositions` repère la frette où
la fondamentale apparaît le plus bas, puis élimine toute position en
dessous — la gamme dessinée et jouée commence donc toujours sur sa
fondamentale et ne fait que monter, comme la vue piano le fait déjà
naturellement.

**Une note = un seul point joué, même si plusieurs cases la proposent.**
Sur le manche, une même hauteur réelle (un unisson) peut être accessible à
plusieurs endroits à la fois. `scaleNeckPlayedKeys` ne retient que la
première occurrence de chaque hauteur et c'est cette même fonction qui
sert à la fois à estomper visuellement les cases « non jouées »
(`renderNeckScaleSVG`) et à construire la liste des notes réellement
envoyées au moteur audio (`buildChordScaleSchedule`) — les deux ne peuvent
donc jamais se contredire sur quel point correspond à quel son.

## 4. Principes de génération des sons (synthèse audio)

Tout le son est produit en JavaScript pur, sans fichier audio ni
bibliothèque externe, via l'API Web Audio.

**Un bus maître unique.** Toutes les voix (notes de piano, accords de
guitare/ukulélé) passent par un même compresseur dynamique puis un même
nœud de gain (`getMasterBus`) avant les haut-parleurs — comme une vraie
console de mixage, ce qui évite l'écrêtage/la distorsion numérique quand
plusieurs notes sonnent ensemble. Ce point de passage unique donne aussi un
seul endroit où tout couper d'un coup : `stopAllAudioNow()` fait descendre
le gain du bus à zéro en une rampe de 30 ms, quelle que soit la durée
restante programmée de chaque note individuellement.

**Voix de piano : synthèse additive.** `playPianoTone` combine la
fondamentale et 3 harmoniques (gains décroissants 1 / 0,32 / 0,14 / 0,06)
via des oscillateurs Web Audio classiques, chacun avec sa propre enveloppe
(attaque rapide, puis décroissance exponentielle en deux temps).

**Voix de guitare/ukulélé : modélisation physique (Karplus-Strong), et
pourquoi elle n'est PAS construite comme un graphe Web Audio en direct.**
`karplusStrongNylon` simule une corde nylon pincée par la technique
classique Karplus-Strong (une boucle à retard dont la sortie est
réinjectée, moyennée et légèrement amortie à chaque tour) — portée note
pour note d'une référence Python existante. **Principe retenu,
délibérément à contre-courant de l'approche « graphe Web Audio » plus
habituelle :** tout ce calcul est fait en tableaux JS classiques (comme le
ferait `numpy`), et Web Audio n'intervient qu'à la toute fin pour lire le
résultat déjà calculé. Raison documentée dans le code : Web Audio traite
son graphe par blocs fixes de 128 échantillons, donc une boucle à retard
construite avec de vrais nœuds (un `DelayNode` qui se réinjecte) sonne
toujours un peu plus longtemps que le retard réellement demandé — un vrai
bug qui désaccordait silencieusement toute note au-dessus de ~344 Hz dans
une version précédente de l'application, corrigé en abandonnant le graphe
en direct pour ce calcul plutôt qu'en compensant le symptôme.

**Caisse de résonance et réverbération, par instrument.**
`applyResonanceBox` ajoute 4 résonances de corps (filtres passe-bande
étroits) mélangées 60 % signal sec / 40 % résonance — les fréquences
diffèrent entre guitare (`GUITAR_BODY_MODES`, graves) et ukulélé
(`UKULELE_BODY_MODES`, plus aiguës, reflétant sa caisse bien plus petite).
`applyRoomReverb` (3 filtres en peigne + 1 passe-tout diffusant + un
passe-bas doux) est appliqué une seule fois, après que toutes les cordes
d'un accord ont été sommées — « comme une vraie pièce autour de l'accord
entier », pas une pièce par corde.

**Le son joué est toujours celui du diagramme affiché, jamais une voix
théorique générique.** `chordAudioNotes` fait sonner les notes RÉELLES du
doigté affiché à l'écran (cordes à vide à leur vraie hauteur, cordes
étouffées exclues, cordes jouées à leur frette réelle) plutôt qu'un voicing
théorique abstrait — le même principe de cohérence diagramme/son que celui
déjà décrit plus haut pour les gammes (section 3) : l'affichage visuel et
ce qui sort des haut-parleurs ne peuvent jamais se contredire, parce qu'ils
sont calculés à partir de la même source.

## 5. Principes de la détection d'accords à partir d'un fichier audio

Le pipeline (`detectChordsFromAudioBuffer`) a évolué par couches successives,
chacune corrigeant un défaut concret observé soit sur des cas synthétiques,
soit sur un morceau réel fourni par l'utilisateur. L'ordre ci-dessous reflète
cette évolution et les raisons de chaque choix.

### 5.1 Chroma pondérée par la basse (bass-weighted chroma)

Le problème : l'accord réel est défini par la note jouée à la **basse**, pas
forcément par la note la plus énergique dans l'ensemble du spectre (un
voicing de piano/guitare peut faire ressortir la tierce ou la quinte plus fort
que la fondamentale). Une chroma « plate » (toutes les fréquences comptent à
poids égal) laisse cette énergie du haut du spectre l'emporter sur la basse,
ce qui est la cause la plus fréquente d'un mauvais choix de fondamentale sur
un enregistrement réel.

**Principe retenu :** les bins de fréquence en dessous de `BASS_BOOST_MAX_FREQ`
(250 Hz, environ jusqu'à B3) comptent `BASS_BOOST_WEIGHT` fois (3x) avant la
normalisation du vecteur de chroma. Validé par un cas synthétique construit
exprès (basse G2 forte mais discrète harmoniquement, contre un contenu aigu
plus fort mais trompeur) : sans pondération l'accord détecté était faux
(racine D au lieu de G), avec pondération il devient correct.

### 5.2 Filtre de confirmation par durée minimale (MIN_CONFIRM_SEGS)

Le problème : un segment isolé, différent de ses deux voisins, est souvent un
« blip » (bruit de médiator, note de passage) et pas un vrai accord.

**Principe retenu, après une itération ratée :** ne supprimer un run d'un seul
segment que si ses DEUX voisins (prev et next) sont **identiques entre eux**
— c'est la signature d'un vrai blip (accord stable, blip, même accord stable).
La première version (suppression si le run ne correspond à AUCUN des deux
voisins) semblait plus logique mais détruisait de vraies progressions rapides
où chaque accord diffère naturellement de ses voisins (prev ≠ next est la
norme dans un enchaînement rapide, pas une anomalie). Leçon générale : un
filtre anti-bruit doit cibler la signature spécifique du bruit qu'il veut
éliminer, pas une condition générique qui recoupe aussi des cas légitimes.

### 5.3 Granularité d'analyse ajustable

Le problème : la durée d'un « instant d'analyse » (segment regroupant
plusieurs trames FFT) est un compromis — plus court réagit plus vite aux
changements d'accords mais est plus sensible au bruit ponctuel ; plus long est
plus stable mais peut fondre deux accords rapides en un seul.

**Principe retenu :** exposer ce compromis à l'utilisateur plutôt que de
choisir une seule valeur figée. `AUDIO_DETECT_SEG_CHOICES = [1,2,4,6]` (en
unités de trame de ~0,37s), sélecteur dans l'UI, persistance dans
`localStorage`, et re-détection sur le buffer audio déjà décodé sans besoin de
re-uploader le fichier (`audioImportLastBuffer` gardé en mémoire).

### 5.4 Rejet de la percussion — trois approches, en ordre croissant de robustesse

C'est la partie la plus substantielle du pipeline, parce qu'un seul filtre ne
suffit pas : différents types de percussion ont des signatures spectrales
différentes, et chaque filtre ajouté cible un angle mort du précédent.

**a) Planéité spectrale seule (approche abandonnée en pratique, mais gardée en
complément).**
Le bruit large bande (caisse claire, charley, claquements de mains) a une
énergie étalée uniformément sur toutes les fréquences — mesurable par la
« planéité de Wiener » (moyenne géométrique / moyenne arithmétique du spectre
d'amplitude). `PERCUSSIVE_FLATNESS_MIN = 0.5` : au-dessus de ce seuil, la
trame entière est exclue du calcul de chroma.
**Limite connue et documentée dans le code :** une percussion **accordée et
résonnante** (congas, bongos, surdo) concentre son énergie sur UNE fréquence,
exactement comme une vraie note — elle n'est donc pas spectralement plate et
passe inaperçue. C'est précisément ce qui s'est produit sur l'intro de
batterie du morceau « Terre » fourni par l'utilisateur pour calibrer le
correctif suivant.

**b) Séparation harmonique/percussive par filtrage médian (HPSS).**
Principe standard (la même idée que `librosa.effects.hpss()`) : pour chaque
cellule (trame, bin de fréquence), comparer
- une estimation « harmonique » = médiane de l'énergie de ce bin sur une
  fenêtre de trames voisines dans le TEMPS (`HPSS_TIME_WIN`, 7 trames soit
  ~2,6s de contexte) — un vrai accord tenu reste stable dans le temps ;
- une estimation « percussive » = médiane de l'énergie des bins voisins en
  FRÉQUENCE à cet instant précis (`HPSS_FREQ_WIN`, 17 bins) — un coup de
  percussion illumine un large pan de fréquences en même temps.

Le bin est pondéré par un masque **continu** (ratio
`harmEst/(harmEst+percEst)`, pas un seuil binaire 0/1). Le masque binaire dur
a été essayé en premier et a **régressé** un test synthétique existant (perte
d'un accord entier dans une progression rapide à 1 changement/seconde),
parce qu'une fenêtre temporelle de ~3s est plus large qu'un accord réel très
court — il se fait donc classer « instable dans le temps » et masquer
entièrement à tort. Le masque continu, combiné à une fenêtre plus petite,
corrige cette régression sans réintroduire le problème de la percussion
accordée.

**c) Porte de rejet au niveau du segment (le correctif décisif).**
Même avec le masque HPSS continu appliqué bin par bin, un groove de
percussion dense et continu (plusieurs coups qui se superposent) laisse
encore assez d'énergie résiduelle « ayant l'air tonale » pour produire un
accord faux mais confiant après pondération. **Principe retenu :** calculer,
par segment, le ratio d'énergie totale effectivement « harmonique »
(`segMaskedEnergy / segTotalEnergy`) ; si ce ratio tombe sous
`PERCUSSIVE_GATE_MIN` (0,40), le segment entier est traité comme silence/pas
d'accord clair, plutôt que de simplement repondérer sa chroma. C'est ce
troisième filtre, combiné au filtre de planéité (a) resté actif en
complément — les deux ciblent des signatures de bruit différentes et ne sont
pas redondants — qui a permis de rendre silencieuse l'intro de batterie du
morceau réel testé, alors que (a) et (b) seuls ne suffisaient pas.

**Limite assumée :** la séparation reste imparfaite sur une percussion très
dense et continue — quelques segments isolés très brefs peuvent encore
glisser à travers. C'est un compromis honnête plutôt qu'une solution à 100 %,
documenté comme tel dans le code et auprès de l'utilisateur.

### 5.5 Piège découvert : fusion erronée à travers un vrai silence

En ajoutant le filtre (c) ci-dessus, un bug plus ancien et plus profond est
apparu : le code qui assemble la timeline finale fusionnait deux runs
identiques consécutifs (même accord) en un seul bloc continu **sans vérifier
qu'aucun vrai silence (gap) ne les séparait**. Cette fusion avait été écrite à
l'origine pour un cas légitime (un run intermédiaire supprimé parce que trop
faible/trop court, entre deux runs du même accord, doit bien se refondre en
un seul bloc continu) — mais elle s'appliquait aussi, à tort, quand
l'intermédiaire était un vrai silence (un segment rejeté par la porte 2.4c),
redonnant artificiellement l'illusion d'un accord tenu en continu pendant
toute l'intro de percussion.

**Principe retenu :** chaque run conservé garde l'index de sa position dans
le tableau complet des runs (`runsIndex`, gaps inclus). Au moment d'assembler
la timeline, on ne fusionne deux runs du même accord que s'il n'existe
**aucun** gap réel entre eux dans ce tableau complet — uniquement des runs
supprimés (même accord ou non) sont « transparents » à la fusion, jamais un
vrai silence. Leçon générale : un silence détecté ne doit jamais être
« absorbé » silencieusement par une logique de fusion pensée pour un tout
autre cas (les blips courts) — toujours vérifier l'intention originale d'une
règle avant de la laisser s'appliquer à un nouveau cas.

## 6. Méthodologie de test retenue pour le pipeline de détection

- **Suite synthétique de régression** (accords purs générés par addition de
  sinusoïdes, avec enveloppe de fondu) couvrant plusieurs cas : triades de
  base, accords de septième, changements rapides (1 accord/seconde),
  fréquence d'échantillonnage réduite, accords diminués/suspendus, accord
  unique tenu. Toute modification du pipeline doit être rejouée contre cette
  suite avant d'être expédiée.
- **Calibration sur un fichier réel fourni par l'utilisateur** dès qu'un
  défaut est rapporté sur un morceau précis — les heuristiques qui marchent
  en synthétique ne garantissent pas un comportement correct sur un
  enregistrement réel (c'est exactement ce qui s'est passé avec le filtre de
  planéité seul). `ffmpeg` pour décoder le MP3 en PCM brut, et au besoin
  `librosa` (Python) comme référence « vérité terrain » pour visualiser un
  spectrogramme et identifier précisément les frontières réelles
  percussion/accord avant de régler un seuil.
- **Extraction du cœur DSP hors du fichier HTML** pour le tester en Node pur
  (`eval()` du bloc de code extrait), en repérant systématiquement la ligne de
  fermeture exacte de la fonction via une recherche du `return` qui la
  précède plutôt qu'un calcul de décalage de lignes — un décalage d'une ligne
  après une édition est l'erreur la plus fréquente rencontrée dans ce
  processus.
