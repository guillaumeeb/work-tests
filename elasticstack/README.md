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
cf. https://sematext.com/blog/2016/12/12/docker-elasticsearch-swarm/ ça a l'air possible. Mais est-ce souhaitable ? Définitivement pas sec.

Actions : 1. Essayer avec service create et dnsrr. Ou alors créer le réseau swarm en dehors du compose, et rattacher le compose au réseau créé en external !
           2. Essayer avec 3 (ou 4) services non répliqués dans le docker-compose, mais bon, pas trop philo swarm, est-ce que ça vaut le coup ?
           3. Passer à la possibilité non full swarm ci-dessous...
Vu la stabilité et les changements des 6 derniers mois. Si cas d'utilisation pas commun, mieux vaut surement en rester à du coder "classique" ? Donc possibilité suivante ?

1. Ca marche !! cf. le fichier docker_create_cmd.txt. Par contre, elasticsearch n'a pas de port ouvert sur l'extérieur via le dnsrr. A priori pas grave dans notre cas, il est accédé par Kibana ou logstash ou Kafka à l'intérieur du Swarm directement sur le réseau (qui font proxy). Sinon il faudrait créer un proxy (haproxy, ou nginx) par dessus pour y accéder.
Le routing mesh sur kibana fonctionne très bien aussi ! On peut facilement accéder à l'ihm depuis n'importe quel noeud.
Est-ce qu'on peut déployer les services avec Ansible ? Sur la machine principale ? Ca doit se faire. Tester si démarrés ou non ?
2. Franchement, ça semble pas valoir le coup, et puis pour le placement des services c'est quasi impossible. Si on fait ça, plutôt passer au point 3 où on maitrise tout.

Autre réflexion : Swarm marche très bien pour logstash ou kibana. On déploie le service, on y accède depuis n'importe quel neoud, on a une sorte de proxy automatique. C'est cool. On pourrait imaginer avoir 2 kibana, qui sont stateless pour haute dispo, et 2 logstashs aussi ?.

## Questions à se poser 
 * Utiliser Kafka en tampon ou pas ? Ca complexifie l'architecture et la maintenance, est-ce vraiment utile ? Dans un premier temps non. Beats et logstash ont des possibilités de tampons sur disque si ES ou Logstash surchargés.
 * Regarder le déploiement d'un, ou plusieurs logstash ? Un logstash par fonctionnalité semble bien : syslog, metrics, gpfs files, product logs, pbs logs... Plus versatile, et simplifié avec Swarm, on prend juste un port différent... A voir, il semble qu'on peut mettre plusieurs fichiers de conf dans un logstash et donc bien séparer les fonctionnalités déjà.
 * Filebeat/Metricbeats docker ou pas ? A priori pas d'image officielle encore, mais y a  pas de raison avec un bon montage de volume. A tester, pas évident car liés au système (file système, ou stats ressources...)

## Tâches en cours
 * Déploiement d'une logstash configuré en syslog. Logs bien transmis mais non formatés actuellement --> OK
 Attention, logstash par défaut a xpack déployé dans le docker, qui cherche à se connecter à un elasticsearch configuré dans le logstash.yaml. Il faut modifier la conf ou le désactiver.
 * Déploiement de filebeat --> via Ansible. Metricbeat aussi déployé via ansible. Ca marche bien sur elastic.
 Par contre, compliqué de charger les template.json et autre dashboard, mais ça se fait, cf fichiers playbook et cmd.txt.
 * Documenter le déploiement en partant de zero. Il y a qqs commande manuelles (creation des services ES/Kibana/Logstash, lancement filebeat et metricbeat)


## Autre possibilité non full swarm, pour elasticsearch essentiellement
Et éventuellement Kafka si utilisé.

Utiliser juste docker run, avec les bon ports mappés pour le elasticsearch.
Pas de network docker pour ES.
Utiliser ansible pour lancer le docker sur chaque machine.
Possibilité de faire un Swarm à côté pour les autres services ? On utilise Swarm pour les différents Kibana ou logstash, on veut pas savoir où ils sont déployés ? Ca serait bien quand même ! --> je confirme ça marche bien.
Avantage de ça, meilleure maitrise du déploiement précis de certains composants distribués.