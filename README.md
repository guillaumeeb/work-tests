# work-tests

## Test swarm 

Tests sur le déploiement de ELK dans un SWARM.

Premier problème sérieu rencontré avec le ulimit pour Elastic Search non pris en compte dans le compose v3.
Voir https://github.com/docker/docker/issues/25209, malgré la description de ulimit ici : https://docs.docker.com/compose/compose-file/#ulimits
Obligé de positionner le ulimit comme defaut dans le démon docker. 
Mais compliqué aussi. Ca ne marche pas non plus en utilisant le /etc/docker/daemon.json https://github.com/docker/docker/issues/25560
Dernière chance avec en redéfinissant le service docker... https://github.com/docker/docker/issues/9889
Seule solution, forcer les bonnes options --default-ulimit dans ce fichier. ça marche avec docker run, à tester avec service create.
 --> ça marche sur une machine. A mettre dans ansible. OK ça marche.
(Peut-être tromp dans le daemon.json, mettre ulimit sans 's'... à voir.)

Second problème : déclaration du réseau dans Swarm. Swarm est fait pour load balancer des services "identiques". Pas l'air super pour les services distribués... Par exemple, là il déclare une ip commune à tout mes démons elasticsearch dans le network du service. Ne pas utiliser le network de Swarm ? Du coup tous les elasticsearch se bound sur la même ip, et donc il y a conflit. Compliqué de se faire découvrir les services entre eux... Peut être avec dnsrr ? https://docs.docker.com/engine/swarm/networking/#use-dns-round-robin-for-a-service. Mais pas dans compose v3.
C'est pas très sec le Swarm mode et compose v3...
Du coup utiliser plutôt le docker service create ? Ca risque de faire beaucoup d'options dans la ligne de commande...
 Actions : 1. Essayer avec service create et dnsrr.
           2. Essayer avec 3 (ou 4) services non répliqués dans le docker-compose, mais bon, pas trop philo swarm, est-ce que ça vaut le coup ?
           3. Passer à la possibilité non full swarm ci-dessous...
Vu la stabilité et les changements des 6 derniers mois. Si cas d'utilisation pas commun, mieux vaut surement en rester à du codker "classique" ? Donc possibilité suivante ?


## Autre possibilité non full swarm

Utiliser juste docker run, avec les bon ports mappés pour le elasticsearch.
Pas de network docker.
Utiliser ansible pour lancer le docker sur chaque machine.
Possibilité de faire un Swarm à côté pour les autres services ? On utilise Swarm pour les différents Kibana ou logstash, on veut pas savoir où ils sont déployés ?
Avantage de ça, meilleure maitrise du déploiement précis des composants.