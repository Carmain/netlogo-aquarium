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

### Etape 1 : Créer un environnement viable

#### Premiers pas dans l'aquarium, les végétariens et les algues

#### Définition de l'aquarium

L'aquarium sera le terrain dans lequel les poissons évolueront.
Ce terrain est tapissé de gravier non comestible pour les animaux y vivant mais où y pousse des algues.
Ici, les algues seront symbolisés par des carrés verts et les graviers par des carrés gris.

Le code suivant permet la définition du sol ainsi que l'apparition d'un nombre d'algues définis par un slider.

```
to setup
  clear-all
  setup-background
  reset-ticks
end

;; Setup the color of the background (gravels)
to setup-background
  ask patches [
    set pcolor grey + (random-float 0.8) - 0.4
  ]
end



to go
  grow-alga
  tick
end

;; Grows a number of algae defined by a slider
to grow-alga
  ask patches [
    if count(patches with [pcolor = green]) < number-of-alga [
      if random 100 > 3 [
        set pcolor green
      ]
    ]
  ]
end
```

#### Apparition des végératiens

##### Déplacements

Afin de les manipuler le plus facilement possible, nous allons nommer nos tortues végératiens à l'aide de `breed` :
```
breed [vegans vegan] ;; vegans fishes
```

Une procédure appelée dans `setup` permet ensuite leur initialisation avec un nombre défini selon un slider :

```
to setup-vegan-fishes
  create-vegans number-of-vegan
  set-default-shape vegans "fish"
  ask vegans [
    setxy random-xcor random-ycor
    set color blue
  ]
end
```

Leurs déplacements sont regulés via une autre procédure appelée dans `go` :
```
to move-vegan-fish
  ask vegans [
    move
  ]
end


to move
  right random 50
  left random 50
  forward 1
end
```

##### Recherche de nourriture & reproduction

Le comportement des poissons se défini de la sorte :
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

## Valeurs par défaut pour les paramètres :

| Titre                          | paramètre          | valeur |
|--------------------------------|--------------------|--------|
| Nombre d'algues                | `number-of-algae`  | 50     |
| Nombre de poissons végératiens | `number-of-vegans` | 10     |
| Taille maximale des poissons   | `max-fish-size`    | 2      |
| Croissance des poissons        | `fish-grow`        | 0.1    |
| Energie venant des algues      | `enery-from-alga`  | 40     |
| Energie à la naissance         | `birth-enery`      | 50     |
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
