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

; Setup the color of the background (gravels)
to setup-background
  ask patches [
    set pcolor grey + (random-float 0.8) - 0.4
  ]
end



to go
  regrow-seaweed
  tick
end

; Grows a number of algae defined by a slider
to regrow-seaweed
  ask patches [
    if count(patches with [pcolor = green]) < number-of-seaweed [
      if random 100 > 3 [
        set pcolor green
      ]
    ]
  ]
end
```
