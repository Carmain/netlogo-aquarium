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

#### Déplacements

Le déplacement des poissons se défini de la sorte :
 * A chaque repas, le poisson gagne de l'énergie
 * Manger le fait aussi grandir jusqu'à obtenir sa taille maximale
 * Si le poisson est suffisament grand, il obtient la capacité de se reproduire
 * Si le poisson a suffisament d'énergie pour se reproduire, il le fait (la reproduction cependant lui en fait perdre).
 * Si le poisson n'a plus d'énergie, il meurt.

Le poisson grandi et gagne de l'énergie à chaque repas :
```
to eat-alga
  ask vegans [
    if pcolor = green [
      display-gravels
      set energy (energy + energy-from-alga)
      grow
    ]
  ]
end
```

Si le poisson est suffisament grand, il peux se reproduire. De plus, la reproduction coûte de l'énergie :
```
to reproduce
  ask vegans [
    if energy > birth-energy and size >= max-fish-size  [
      set energy energy - birth-energy
      hatch 1 [
        set energy birth-energy
        set size 1
      ]
    ]
  ]
end
```

Si le poisson n'a plus d'énergie, il meurt :
```
to check-death
  ask vegans [
    if energy <= 0 [ die ]
  ]
end
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
