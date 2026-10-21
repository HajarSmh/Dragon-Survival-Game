# Analyse du projet — « Serpents, dragons et circuit » (p5.js)

## 1. Présentation

Le projet est un **jeu-simulation en temps réel qui tourne dans le navigateur**, écrit en JavaScript avec la
bibliothèque **p5.js**. Il met en scène plusieurs familles d'agents autonomes (serpents, dragons, oiseaux,
fruits, voitures) dont **tous les déplacements sont produits par des forces** — les *steering behaviors* de
Craig Reynolds, étudiés en cours : `seek`, `flee`, `arrive`, `wander`, `pursue`, `separation`, `alignement`,
`cohésion`, suivi de chemin et évitement d'obstacles.

Le joueur contrôle les serpents **indirectement** avec la souris. Son but : **faire survivre ses serpents**
face à des dragons qui les chassent, jusqu'à ce que tous les dragons soient morts.

| | |
|---|---|
| **Victoire** | tous les dragons sont éliminés (écran *VICTORY*, avec le temps écoulé) |
| **Défaite** | tous les serpents sont morts (écran *GAME OVER*, avec le temps de survie) |
| **Rejouer** | bouton *Restart* ou touche **R** |

## 2. Déroulement d'une partie

1. **Écran de départ** : le joueur règle le nombre de **dragons** (1 à 6) et de **serpents** (1 à 6), puis
   clique sur *START GAME* (ou touche **Entrée**). Tant que la partie n'a pas démarré, les dragons ne font
   que se promener.
2. **Les serpents** se placent en orbite autour du curseur de la souris. Sans souris dans la fenêtre, ils se
   promènent sur une courbe de Lissajous.
3. **Les fruits** (pomme, orange, citron, raisin) restent immobiles. Quand un serpent passe à moins de
   110 px, le fruit se met à trembler, puis **fonce vers lui** et se fait manger.
4. **Manger fait grossir** le serpent (+20 % par fruit, jusqu'à ×3). Il commence **très rapide** et
   **ralentit à mesure qu'il grossit** : manger a donc un coût tactique.
5. Après une courte digestion (0,9 s), le serpent **éjecte une boule de feu** par la queue. Les boules de feu
   glissent, ralentissent puis disparaissent. Elles **attirent les dragons** (repérées de plus loin, jusqu'à
   3 fois plus attractives qu'une autre proie), et chaque boule mangée fait **rétrécir** le dragon de 15 % ;
   sous 45 % de sa taille, il **meurt**. C'est l'arme des serpents.
6. **Les dragons** chassent la proie la plus attractive dans un rayon de 450 px : boules de feu, oiseaux ou
   serpents. Une morsure retire à la queue d'un serpent environ 20 % de ses points de base ; s'il ne reste
   que la tête, **le serpent meurt**. Chaque serpent mordu fait **grossir** le dragon (+10 %, jusqu'à ×2).
   Les oiseaux mangés réapparaissent 6 s plus tard.
7. **Le circuit** est protégé par une **couverture de verre 3D transparente** : les dragons ne peuvent pas y
   entrer et ignorent tout ce qui s'y trouve. Voitures et serpent du circuit y roulent donc en sécurité.
   Quand un dragon heurte le verre, une onde cyan s'affiche à l'impact.

## 3. Les acteurs et leurs comportements

| Acteur | Fichier | Comportement (forces utilisées) |
|---|---|---|
| **Serpent de la souris** | `snake.js` | La tête est un véhicule qui fait `arrive` vers un point en orbite autour de la souris, avec une **force d'ondulation** sinusoïdale perpendiculaire à la vitesse. Le corps est une **chaîne de points à distance fixe** (chaque point suit le précédent). Largeur et longueur différentes selon le modèle (*Fil*, *Moyen*, *Gros*). Yeux « googly » (pupilles à ressort), clignement, langue fourchue. |
| **Dragon** | `dragon.js` | `seek` avec **anticipation** (il vise la position prédite de l'oiseau : position + vitesse × 8), `wander` sans proie, temps de digestion après un repas, **évitement du verre** (contrainte stricte). Vol au-dessus de tout, réapparition de l'autre côté de l'écran. |
| **Oiseau** | `bird.js`, `boids.js` | **Flocking** de Shiffman : alignement, cohésion, séparation, plus un peu de `wander`. 25 oiseaux, battement d'ailes, ombre portée. Ils ne font pas d'évitement d'obstacles. |
| **Fruit** | `fruit.js` | Immobile, puis `seek` vers le serpent proche. |
| **Boule de feu** | `fireball.js` | Éjectée par un serpent, ralentit (frottement), expire, rétrécit les dragons qui la mangent. |
| **Voiture** | `car.js` | **Suivi de chemin** : vise un point devant elle sur la piste, ralentit avant les virages, garde sa voie préférée, `separation` avec les autres voitures, évite pierres et serpents (évitement anticipé), reste toujours dans la piste. |
| **Serpent du circuit** | `trackSnake.js` | Serpent cyan qui parcourt la piste en contournant voitures et pierres. Un **détecteur de blocage** change de voie, puis passe en **mode fantôme** (semi-transparent, il traverse les obstacles 1,5 s) s'il reste coincé. |
| **Pierres** | `stone.js` | Obstacles ronds posés sur la piste ; leur position est stockée *relativement* à la piste (avancement + décalage latéral), donc ils suivent la piste quand la fenêtre est redimensionnée. |
| **Anneaux d'effet** | `splitRing.js` | Cercle qui grandit et s'efface : morsure, repas, mort, impact sur le verre. |

### Évitement d'obstacles anticipé (`avoid.js`)
Fonction partagée par les voitures, le serpent du circuit et les têtes de serpent. Elle reprend l'algorithme
`ahead / ahead2` du cours (points situés à vitesse × 30 et vitesse × 15 devant l'agent), avec deux
améliorations : tous les obstacles sont testés et **le plus menaçant** est retenu, et une **composante
latérale** évite qu'un véhicule arrivant pile en face reste bloqué.

### Contraintes strictes après les forces
Les obstacles et les bordures sont aussi appliqués comme des **corrections de position** (replacer l'agent sur
le bord, annuler la vitesse qui pousse contre l'obstacle). Elles sont répétées plusieurs fois pour qu'aucune
ne défasse l'autre. C'est ce qui garantit qu'une voiture ne sort jamais de la piste et ne roule jamais sur
une pierre.

## 4. Décor, rendu et interface

- **Fond psychédélique** (`background.js`) : anneaux concentriques ondulants dans une palette de 8 couleurs,
  vortex central, étoiles. Il réagit à la souris. Il est dessiné dans un canvas hors-écran à 65 % de la
  résolution, rafraîchi à 30 Hz, puis étiré : la charge de calcul reste faible.
- **Couverture de verre 3D** (`track.js`) : dessinée **une seule fois** dans un canvas hors-écran (refaite
  seulement si la piste ou la fenêtre change). Elle combine bord de verre, ombre portée, intérieur évidé,
  teinte, croissants d'ombre et de lumière (volume), arceaux transversaux et reflets. Seul un éclat lumineux
  qui glisse le long de la piste est recalculé à chaque image.
- **Piste** : courbe fermée de type **spline Catmull-Rom** passant par 12 points de contrôle en coordonnées
  normalisées (0 à 1), donc elle s'adapte à toute taille de fenêtre. Méthodes clés : `closest(x, y)` (point
  de la piste le plus proche, avec tangente et normale), `pointAt(s)` (point à l'abscisse curviligne *s*).
- **Musique** (`music.js`) : générée par la Web Audio API (aucun fichier audio), influencée par la position de
  la souris, avec des effets sonores.
- **Interface** (`index.html`, `style.css`, `gameSliders.css`) : barre de contrôle au style rétro (police
  *Press Start 2P*) avec curseurs segmentés, afficheurs LCD et boutons pixel art. Contrôles : luminosité du
  **fond**, **écart** entre les points des serpents, **volume** de la musique, ajout/retrait de **voitures**
  (jusqu'à 100), bouton **musique**, bouton **debug**.
- **Mode debug** (touche **D** ou bouton) : pour *toute instance de `Vehicle`*, affiche le rayon de collision,
  les vecteurs `ahead`/`ahead2`, la trajectoire, une étiquette « classe + vitesse » et une zone propre à la
  classe (distance de réveil du fruit, rayon de perception du dragon, zone de ralentissement du serpent…).

## 5. Architecture du code

### 5.1 Principe
JavaScript « classique », sans module ni outil de build. `index.html` charge les scripts **dans un ordre
précis** (classes et utilitaires d'abord, `sketch.js` en dernier) et tout vit dans l'espace global. Pour lancer
le jeu, il suffit d'ouvrir `index.html` dans un navigateur ou avec l'extension *Live Server* de VS Code (port
5502 configuré dans `.vscode/settings.json`).

### 5.2 Hiérarchie de classes

```
Vehicle  (vehicle.js : forces, seek, flee, arrive, pursue, separate, update, edges, path)
├── Car                    voiture suivant la piste
├── Snake                  corps en chaîne de points ; sa position = celle de la tête
│   └── TrackSnake         serpent du circuit
├── SnakeHead              tête du serpent (arrive + ondulation + yeux)
├── Fruit                  fruit qui fonce sur un serpent proche
├── Dragon                 chasseur
├── Fireball               boule de feu                         (fireball.js)
├── SplitRing              anneau d'effet visuel
├── Decoy, RandomTarget    ancien mode « avions »
└── BackgroundCircle       cercle décoratif (non utilisé)

Boid  (boids.js : flocking de Shiffman)
└── Bird                   oiseau

Sans héritage : Track, Stone, Bush, RetroBackground, DebugMode, helpers (aheadAvoid, SeekPopulation,
TargetSwarm, MouseInfluence, PsychedelicMusic)
```

L'héritage est au cœur de la conception : tout ce qui bouge **est un `Vehicle`**, donc reçoit `applyForce()`,
`update()`, `edges()`… et peut être traité de façon uniforme (par exemple par le mode debug). Chaque sous-classe
ne décrit que *ce qui la rend différente* : ses forces et son dessin (`show()`).

### 5.3 La boucle principale (`sketch.js`)
À chaque image, `draw()` enchaîne :

1. lecture de la souris (position + valeurs lissées pour le fond et la musique) ;
2. **fond**, voile d'atténuation, **piste** ;
3. **mise à jour** des serpents, des voitures et du serpent du circuit, des fruits (poursuite, repas), de la
   digestion et des boules de feu ;
4. **dessin dans l'ordre de profondeur** : pierres, voitures, serpent du circuit, fruits, boules de feu,
   serpents, puis **verre** (par-dessus), **oiseaux**, **dragons** ;
5. **événements de jeu** : `Dragon.step()` renvoie un événement (oiseau mangé, serpent mordu, boule de feu
   mangée) traité par `handleDragonEvent()` ; mort d'un serpent ou d'un dragon ; déclenchement de *GAME OVER* ou
   de *VICTORY* ;
6. retour des oiseaux mangés, mode debug, animations d'effets, mise à jour des compteurs.

Le temps est géré avec un `dt` en secondes plafonné à 0,1 s (pour qu'un onglet mis en pause ne provoque pas de
saut) ; les minuteurs (digestion, repas, respawn) en dépendent. `windowResized()` reconstruit la piste et
recale tous les objets sur la nouvelle taille.

## 6. Fichiers du dépôt

**Page et configuration** — `index.html` (HUD, écrans de départ / victoire / défaite, ordre des scripts),
`style.css`, `gameSliders.css`, `jsconfig.json`, `.vscode/settings.json`.

**Cœur** — `sketch.js`, `vehicle.js`, `avoid.js`, `debug.js`, `mouseInfluence.js` (normalisation et lissage de
la souris).

**Entités** — `snake.js`, `trackSnake.js`, `dragon.js`, `bird.js`, `boids.js`, `fruit.js`, `fireball.js`,
`splitRing.js`, `stone.js`, `bush.js` (buissons : présents mais désactivés, `BUSH_COUNT = 0`).

**Circuit** — `track.js`, `car.js`.

**Décor et son** — `background.js`, `music.js`, `libraries/p5.min.js`.

**Ancien mode « avions / cibles »** (conservé mais masqué par `SHOW_PLANES_AND_SHIPS = false`) —
`randomTarget.js` (avion qui fuit les véhicules, se divise avec le temps, lâche des leurres), `decoy.js`,
`population.js`, `targetSwarm.js` ; `target.js` et `backgroundCircle.js` ne sont pas utilisés.

## 7. Choix techniques à retenir

- **Tout par les forces** : `seek`, `arrive`, `wander`… produisent des mouvements fluides et organiques.
  Les agents sont *composés* par somme pondérée de forces (ex. voiture = suivi de piste + séparation + évitement ×3).
- **Corps de serpent en chaîne de points** à distance fixe : méthode simple, stable et très lisible visuellement.
- **Croissance progressive** : taille visée (`sizeGoal`) et taille actuelle (`sizeScale`) séparées, rattrapage
  doux, vitesse = base × boost / taille^1,2. Le jeu est équilibré en changeant quelques constantes en tête
  de fichier.
- **Dessins coûteux mis en cache** : fond (basse résolution, 30 Hz) et verre du circuit (rendu unique).
- **Modules « purs »** sans dépendance à p5 (`population.js`, `targetSwarm.js`, `avoid.js`) exportés pour Node,
  donc testables hors navigateur.

## 8. Limites connues et améliorations possibles

- **`jsconfig.json`** référence un fichier de types situé dans un chemin d'une autre machine
  (`/Users/...`) : sans conséquence à l'exécution, mais gêne les diagnostics de l'éditeur.
- **Noms hérités** : dans le code des serpents, les boules de feu sont encore stockées dans un tableau nommé
  `poops` et déclenchées par `eat()` / `updateDigestion()` ; un renommage améliorerait la lisibilité.
- **Code hérité** (ancien mode avions, `target.js`, `backgroundCircle.js`) : à déplacer dans un dossier
  `legacy/` ou à supprimer pour alléger le projet.
- **Performance** : `Track.closest()` parcourt tous les points de la piste (~170) et est appelée souvent
  (voitures, serpent du circuit, dragons). Mémoriser le dernier segment trouvé ou utiliser une grille spatiale
  permettrait de supporter beaucoup plus d'agents.
- **Organisation** : variables globales et dépendance à l'ordre des balises `<script>` ; pas de suite de tests
  automatisés ni de fichier de configuration central pour les constantes d'équilibrage.
- **Dépendance réseau** : la police *Press Start 2P* vient de Google Fonts (repli automatique sur Courier New
  hors ligne).