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

## Comment lancer netlogo-aquarium ?

 * [Télécharger Netlogo](https://ccl.northwestern.edu/netlogo/download.shtml)
 * Ouvrir le fichier `aquarium.nlogo` via ce logiciel
 * Cliquer sur `Setup`
 * Cliquer sur `Go`

## Actions possible sur l'environnement & paramètres

### Les actions

 * **Setup** : Permet d'initialiser le monde suivant les apramètres donné ci-dessous
 * **Go** : Lance l'évolution du monde après l'avoir initialisé via l'action `Setup`
 * **Record** : Permet de lancer une capture vidéo de l'évolution du monde

### Les paramètres

 * **number-of-algae** : Permet de faire varier le nombre d’agents algue sur le terrain.
 * **number-of-vegans** : Fait varier le nombre d’agents poissons végétariens dans l’aquarium.
 * **number-of-carnivorous** : Fait varier le nombre d’agents poissons carnivores dans l’aquarium.
 * **max-fish-size** : Permet de définir la taille maximale que les poissons peuvent prendre. Au fur et à mesure, les poissons vieillissent et grandissent en taille. Cette variable est utile car ce n’est qu’à partir du moment où le poisson a atteint sa taille maximale qu’il peut commencer à se reproduire.
 * **diffusion-rate** : Définit la puissance de diffusion des traces des agents herbivores, carnivores & des algues.
 * **evaporation-rate** : Définit la vitesse d’évaporation des traces laissé par les agents.
 * **energy-from-alga** : Énergie gagnée par un herbivore lorsqu’il consomme une algue.
 * **energy-from-fish** : Énergie gagnée par un herbivore lorsqu’il mange un végétarien.
 * **birth-energy** : Énergie de base que les poissons ont à la naissance.
 * **starving-death** : Si la variable est à “Off”, les poissons ne peuvent pas mourrir de faim.

#### Valeurs par défaut pour les paramètres :

| paramètre               | valeur |
|-------------------------|--------|
| `number-of-algae`       | 50     |
| `number-of-vegans`      | 10     |
| `number-of-carnivorous` | 10     |
| `max-fish-size`         | 2      |
| `diffusion-rate`        | 2      |
| `evaporation-rate`      | 2      |
| `enery-from-alga`       | 40     |
| `enery-from-fish`       | 40     |
| `birth-enery`           | 50     |
| `starving-death`        | Off    |
