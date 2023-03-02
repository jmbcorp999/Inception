# Inception

## Notions generales

### C'est quoi Docker ?
Docker est une plate-forme logicielle qui vous permet de concevoir, tester et déployer des applications rapidement. Docker intègre les logiciels dans des unités normalisées appelées conteneurs, qui rassemblent tous les éléments nécessaires à leur fonctionnement, dont les bibliothèques, les outils système, le code et l'environnement d'exécution. Avec Docker, vous pouvez facilement déployer et dimensionner des applications dans n'importe quel environnement, avec l'assurance que votre code s'exécutera correctement.
Le fonctionnement de Docker repose sur le noyau Linux et les fonctions de ce noyau, comme les groupes de contrôle cgroups et les espaces de nom. Ce sont ces fonctions qui permettent de séparer les processus pour qu’ils puissent s’exécuter de façon indépendante.
En effet, le but des conteneurs est d’exécuter plusieurs processus et applications séparément. C’est ce qui permet d’optimiser l’utilisation de l’infrastructure sans pour autant atténuer le niveau de sécurité par rapport aux systèmes distincts.
La principale difference avec une machine virtuelle "traditionnelle" (VMWare, VirtualBox...) repose sur le fait que chaque container est dedie a l'execution d'un seul et unique processus, une seule application, a l'interieur d'un environnement virtuel base sur un noyau Linux. Une machine virtuelle "traditionnelle", elle, execute un systeme d'exploitation complet, et est en mesure de de faire tourner plusieurs services ou applications simultanement.
Avantages et inconvenients de Docker, par rapport a une machine virtuelle classique :
- Avantages : deploiement rapide, facile, et compatible avec la majorite des plateformes (Windows, OSX, Linux).
- Inconvenients : ne s'adapte pas a tous les types de projets.

### Comment structurer notre projet
Pour respecter les consignes du projet, mais egalement pour disposer d'un projet clair, il est souhaitable de structurer le projet de la sorte.
```
.
├── Makefile
├── .gitignore
└── srcs
    ├── docker-compose.yml
    ├── .env
    └── requirements
        ├── mariadb
        │   ├── Dockerfile
        │   ├── .dockerignore
        │   ├── conf
        │   │   └── configuration.conf
        │   └── tools
        │       └── configure.sh
        ├── nginx
        │   ├── Dockerfile
        │   ├── .dockerignore
        │   ├── conf
        │   │   └── configuration.conf
        │   └── tools
        │       └── configure.sh
        └── wordpress
            ├── Dockerfile
            ├── .dockerignore
            ├── conf
            │   └── configuration.conf
            └── tools
                └── configure.sh
```
Bien entendu, libre a vous de nommer les fichiers de configuration et les tools comme bon vous semble.

### Description du contenu du projet

#### Makefile
Il est souhaitable de creer pas mal de commande au sein de votre Makefile, dans l'objectif de pouvoir interagir de maniere assez concrete avec votre projet. Exemple de commandes :

1. up 
2. build
3. start
4. down
5. reload
6. status
7. logs
8. clean
9. cleanv
10. prune
11. fclean
12. volume

up: intro volumes ## Launch Inception in background
	@ docker-compose -f ./srcs/docker-compose.yml up -d

build: intro volumes ## Build Inception (use make start after, to launch)
	@ docker-compose -f ./srcs/docker-compose.yml build

start: ## Begin Inception
	@ docker-compose -f srcs/docker-compose.yml start

down: ## Stop Inception
	@ docker-compose -f srcs/docker-compose.yml down

reload: volumes ## Restart Inception
	@ docker-compose -f srcs/docker-compose.yml up --build

status: ## Display Inception status
	@ docker ps

logs: ## See Inception logs
	@ docker-compose -f ./srcs/docker-compose.yml logs

clean: down ## Stop Inception & Clean inception docker (prune -f)
	@ docker system prune -f

cleanv: ## Remove persistant datas
	@ sudo rm -rf ~/data

prune: down ## Remove all dockers on this system (prune -a)
	@ docker system prune -a

fclean: down prune cleanv ## Remove all dockers on this system & Remove persistant datas
	@ echo

volumes: ## Prepare folders needed
	@ mkdir -p ~/data/website/wordpress
	@ mkdir -p ~/data/database
	@ mkdir -p ~/data/portainer



#### Le fichier Dockerfile
Un Dockerfile est un fichier texte qui contient les instructions nécessaires pour construire une image Docker. C'est en quelque sorte le Makefile d'un container.
Le Dockerfile est utilisé pour automatiser le processus de construction d'une image Docker. Il spécifie les dépendances de l'application, les étapes d'installation, la configuration de l'environnement et les commandes pour exécuter l'application.
Un Dockerfile ne sert qu'a deployer, normalement, qu'un seul et unique processus, application ou service.

#### Le docker-compose
Comme vu precedement, un container est destine a faire tourner une seule application ou un seul service, hors il est souvent necessaire pour le bon deroulement d'un projet d'en disposer de plusieurs. C'est la que rentre en jeu le docker compose.
Docker Compose est un outil destiné à définir et exécuter des applications Docker à plusieurs conteneurs. Dans Compose, vous utilisez un fichier YAML pour configurer les services de votre application. Ensuite, vous créez et vous démarrez tous les services à partir de votre configuration en utilisant une seule commande. C'est un peu comme un Makefile qui appelerait plusieurs autres Makefile pour compiler les librairies necessaires au fonctionnement du programme.
Pour info, YAML est un language de programmation regulierement utilise en scripting, il est tres proche du language humaim et son utilisation pour les Dockers est tres facile.


- Les commandes disponibles
- FROM
- RUN
- ENTRYPOINT
- COPY
- WORKDIR



Le makefile et ses options

Le fichier .dockerignore

L'environnement

Les volumes

Les networks

Nginx

MariaDB

Wordpress

Les bonus :
- Redis
- Serveur FTP
- Site statique
- Portainer
- Autres idees de bonus
