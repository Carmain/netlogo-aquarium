# netlogo-aquarium

## Qu'est ce que netologo-aquarium ?

netlogo-aquarium est un projet scolaire dans le cadre d'une option de robotique dont l'intitulé est le suivant :
> Construire un modèle d’écosystème d’un aquarium (15 espèces minimum, ne pas oublier la nourriture végétale).
Il vous est demandé de créer toutes les interactions entre les ennemis et les amis.
Cet écosystème doit survivre entre végétaux, animaux et la qualité de l’eau.
Tous les agents sont autonomes après leur position de naissance et l’évolution est réaliste selon des critères donnés.
Il est important d’inclure les prises de décisions, les multi agents et la gestion des collisions.
Tous les agents devront gérer des émissions de champ positif pour attirance comme les nourritures et des champs répulsif pour éviter d’être mangé.
Les plantes peuvent aussi adopter cette demande.

Pour ce faire, nous allons comme le nom du projet l'indique travailler sur netlogo.
En guise de rendu, nous explciterons çi dessous les différentes étapes pour réaliser un tel projet.
Ce dernier sera aussi accompagné de petits films permettant de démontrer les différents résultats & avancements.

## La construction du projet

### Setup de l'aquarium et de l'environnement

L'aquarium sera le terrain dans lequel les poissons évolueront.
Ce terrain est tapissé de gravier symbolisé par des patches de couleur grise.

Le code ci-dessous permet la définition de ce qu'est un patch et de lui attribuer trois variables permettant de stocker une valeur numérique.
L'avantage est que chaque patche garde en mémoire la trace de passage des agents poissons ainsi que la proximité des agents algues.

A noter ici qu'il a été décidé de désactiver les paramètres `World wraps horizontally` & `World wraps vertically` afin
```
patches-own [
  ;; amount of tracks on this patch
  carnivorous-track
  vegans-track
  algae-track
]
```

Vient ensuite la définition des différents agents. Chaque agent est nommé afin de les manipuler plus facilement et explicitement dans le code.
Les poissons végétariens et carnivores possèdent aussi un attribut `energy` utilisé par la suite pour moduler leurs cycles de vie.

```
breed [ vegans vegan ] ;; vegans fishes
breed [ carnivorous a-carnivorous ] ;; carnivorous fishes
breed [ algae alga ]

vegans-own [ energy age ]
carnivorous-own [ energy age ]
```

La fonction `setup` quand à elle s'occupe d'appeler un ensemble de procédures afin d'initaliser l'environnement.
Elle est appelé dans l'interface via le bouton `Setup`. Quelques précisions cependant :
 * `clear-all` permet de remettre à zéro l'environnement pour une nouvelle instatiation
 * `reset-ticks`permet grossièrement de remettre le compteur de cyle à zéro.

```
to setup
  clear-all
  setup-background
  setup-algae
  setup-fishes
  reset-ticks
end
```

#### Initialisation du décors et du terrain

L'initialisation du terrain et des patchs de fait via la méthode `setup-background`.
Cette méthode recouvre le monde de patches de variate de gris. Ces patches pourront par la suite interragir avec les agents les parcourants.

```
to setup-background
  ask patches [
    set pcolor grey + (random-float 0.8) - 0.4
  ]
end
```

#### Initialisation des agents
Trois types d'agents existent, répartis selon une chaîne alimentaire en pyramide :
 * Sur le plus bas niveau et en plus grande quantité à l'initalisation, nous allons retrouver la nourriture végétale représenté par des agents en forme d'algue de couleur verte.
 * Au second niveau, ce sont les poissons végétariens se nourrissant d'algues. Ces derniers sont en nuance de bleu.
 * Enfin au sommet de la pyramide nous retrouvons les carnivores, représenté par des poissons en nuance de rouge dont la forme diffère des végétariens.

initialisation des algues sur le terrain :
Le nombre d'algues peut être défini par le slider `number-of-algae` dans l'interface du programme. Au setup, ce nombre est réprti de manière aléatoire sur le terrain.
```
to setup-algae
  set-default-shape algae "plant"
  create-algae number-of-algae [
    setxy random-xcor random-ycor
    set color 63
    set size 2.3
  ]
end
```

Initialisation des agents poissons :
Ici, le programme s'occupe de créer les deux sortes de poissons disponibles. Ces derniers se distinguent facilement par la couleur,
la forme et la taille de départ légèrement plus grande pour les carnivores. Ces derniers sont répartis eux aussi aléatoirement sur la carte.
Enfin, l'énergie attribué à la naissance est fixé arbitrairement par l'attribut dont la valeur est fixé par le slider `birth-energy`.
```
to setup-fishes
  set-default-shape vegans "fish 2"
  create-vegans number-of-vegans [
    basic-fishes-setup
    set size 2
    random-in-range-color 102
  ]

  set-default-shape carnivorous "fish"
  create-carnivorous number-of-carnivorous [
    basic-fishes-setup
    set size 2.2
    random-in-range-color 14
  ]
end

to basic-fishes-setup
  setxy random-xcor random-ycor
  set energy birth-energy
  set age 0
end
```

### Gestion des déplacement des agents & recherche de nourriture

La mise en mouvement du projet est effectué via un ensemble de procédures dans `go`.
Cette méthode peut être appelée via l'interface avec le bouton `Go`.

Avant de passer aux différentes procéure régissant le comportement des agents,
il convient d'expliquer cette ligne : `if not any? vegans or not any? carnivorous [ stop ]`.
Cette dernière permet d'arrêter le programme si toute la famille des carnivores ou des herbivores venait à disparaître.

```
to go
  move-fishes
  set-tracks
  check-death
  reproduce

  if not any? vegans or not any? carnivorous [ stop ]
  tick
end
```

#### Déplacements & recherche de nourriture

Le déplacement des agens végétatiens & carnivores se fait via la procédure `move-fishes`:
```
to move-fishes
  ;; Move the vegans fishes
  ask vegans [
    ifelse energy < 25
      [
        hunt-algae
        eat-alga
      ]
      [ walk-around ]

    forward 0.3
    set vegans-track 60
    if not can-move? 1 [ rt 180 ]
    set energy energy - 0.25
  ]

  ;; Move the carnivorous fishes
  ask carnivorous [
    ifelse energy < 25
      [
        hunt-vegans
        forward 0.5
        eat-fish
      ]
      [
        walk-around
        forward 0.3
      ]

    set carnivorous-track 60
    if not can-move? 1 [ rt 180 ]
    set energy energy - 0.25
  ]
end
```

Le comportement des deux familles de poissons est quasiment indentique à un détail près que nous expliciterons par la suite.

Si le poisson a suffisament d'énergie (au dessus de 25 points), ce dernier ne fera de circuler dans l'aquarium, simulé par la méthode `walk-around`.
Cette dernière ne sert juste qu'à donner une direction aléatoire au poisson.
```
to walk-around
 right random 50
 left random 50
end
```

Si le poisson au contraire commence à s'affamer, il va partir en chasse à la recherche de nourriture via les méthodes `hunt-algae` & `hunt-vegans`
respectivement pour les hebivores & les carnivores.

La chasse va se constituer à la recherche des patches à droite, à gauche et en face du poissons contenant la valeur la plus haute de la trace recherché,
à savoir `algae-tracks` pour les végétariens et `vegans-track` pour les carnivores.
Ici n'est présenté que la fonction pour la recherche d'algues, cependant les carnivores ont un comportement similaire décrit dans le code d'une manière quasiment indentique.

Le résultat de cette recherche oriente dont l'agent vers la trace la plus forte. C'est ensuite dans la fonction `move-fishes` qu'un `forward` est fait pour le faire avancer.
Une différence est à souligner entre les deux familles, si les carnivores repèrent une trace, ces derniers se déplacent plus rapidement pour rattraper leur proie.

```
to hunt-algae
  let track-ahead algae-track-at-angle 0
  let track-right algae-track-at-angle 45
  let track-left algae-track-at-angle -45
  fix-position-with-tracks track-left track-ahead track-right
end

;; Adjust the position of the fish when it found some tracks
to fix-position-with-tracks [track-left track-ahead track-right]
  if (track-right > track-ahead) or (track-left > track-ahead)
  [
    ifelse track-right > track-left
      [ right 45 ]
      [ left 45 ]
  ]
end

to-report vegans-track-at-angle [angle]
  let p patch-right-and-ahead angle 1
  if p = nobody [ report 0 ]
  report [ vegans-track ] of p
end
```

#### Diffusion des traces

#### Reproduction des poissons

Si le poisson est suffisament grand, il peux se reproduire. De plus, la reproduction coûte de l'énergie :
```
```

#### Décès des poissons

Si le poisson n'a plus d'énergie, il meurt :
```
```

### Valeurs par défaut pour les paramètres :

| Titre                          | paramètre          | valeur |
|--------------------------------|--------------------|--------|
| Nombre d'algues                | `number-of-algae`  | 50     |
| Nombre de poissons végératiens | `number-of-vegans` | 10     |
| Taille maximale des poissons   | `max-fish-size`    | 2      |
| Croissance des poissons        | `fish-grow`        | 0.1    |
| Energie venant des algues      | `enery-from-alga`  | 40     |
| Energie à la naissance         | `birth-enery`      | 50     |

## Définition des modèles des agents

## A propos du carnet de bord en vidéo :

Chaque étape conséquence a été pris en vidéo afin de garder des traces de l'avancement du projet.

La capture de vidéo peux se faire après le **setup** en cliquant sur le bouton **Record**.
Ce dernier appellera ensuite la fonction `make-movie` dont le code est le suivant :

```
to make-movie
  ;; prompt user for movie location
  user-message "First, save your new movie file (choose a name ending with .mov)"
  let path user-new-file
  if not is-string? path [ stop ]  ;; stop if user canceled

  ;; run the model
  setup
  movie-start path
  while [ticks < 500] [
    movie-grab-view
    go
  ]

  ;; Stop and export the movie
  movie-close
  user-message (word "Exported movie to " path)
end
```
