# work-tests

## Test swarm 

Tests sur le déploiement de ELK dans un SWARM.

Premier problème sérieu rencontré avec le ulimit pour Elastic Search non pris en compte dans le compose v3.
Voire https://github.com/docker/docker/issues/25209.
Obligé de positionner le ulimit comme defaut dans le démon docker. 
Mais je n'ai pas réussi. Ca ne marche pas non plus en utilisant le /etc/docker/daemon.json https://github.com/docker/docker/issues/25560
Dernière chance avec en redéfinissant le service docker... https://github.com/docker/docker/issues/9889
Seule solution, forcer les bonnes options --default-ulimit dans ce fichier. ça marche avec docker run, à tester avec service create.
 --> ça marche sur une machine. A mettre dans ansible.
Peut-être tromp dans le daemon.json, mettre ulimit sans 's'... à voir.


## Autre possibilité non full swarm

Utiliser juste docker run, avec les bon ports mappés pour le elasticsearch.
Pas de network docker.
Utiliser ansible pour lancer le docker sur chaque machine.
Possibilité de faire un Swarm à côté pour les autres services ?
Avantage de ça, meilleure maitrise du déploiement précis des composants.