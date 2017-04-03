# work-tests

## Test swarm 

Tests sur le déploiement de ELK dans un SWARM.

Premier problème sérieu rencontré avec le ulimit pour Elastic Search non pris en compte dans le compose v3.
Obligé de positionner le ulimit comme defaut dans le démon docker.

## Autre possibilité non full swarm

Utiliser juste docker run, avec les bon ports mappés pour le elasticsearch.
Pas de network docker.
Utiliser ansible pour lancer le docker sur chaque machine.
Possibilité de faire un Swarm à côté pour les autres services ?
Avantage de ça, meilleure maitrise du déploiement précis des composants.