# Inception

## I - Notions generales

### C'est quoi Docker ?
Docker est une plate-forme logicielle qui vous permet de concevoir, tester et déployer des applications rapidement. Docker intègre les logiciels dans des unités normalisées appelées conteneurs, qui rassemblent tous les éléments nécessaires à leur fonctionnement, dont les bibliothèques, les outils système, le code et l'environnement d'exécution. Avec Docker, vous pouvez facilement déployer et dimensionner des applications dans n'importe quel environnement, avec l'assurance que votre code s'exécutera correctement.
Le fonctionnement de Docker repose sur le noyau Linux et les fonctions de ce noyau, comme les groupes de contrôle cgroups et les espaces de nom. Ce sont ces fonctions qui permettent de séparer les processus pour qu’ils puissent s’exécuter de façon indépendante.
En effet, le but des conteneurs est d’exécuter plusieurs processus et applications **séparément**. C’est ce qui permet d’optimiser l’utilisation de l’infrastructure sans pour autant atténuer le niveau de sécurité par rapport aux systèmes distincts.
**La principale difference avec une machine virtuelle "traditionnelle" (VMWare, VirtualBox...) repose sur le fait que chaque container est dedie a l'execution d'un seul et unique processus**, une seule application, a l'interieur d'un environnement virtuel base sur un noyau Linux. Une machine virtuelle "traditionnelle", elle, execute un systeme d'exploitation complet, et est en mesure de de faire tourner plusieurs services ou applications simultanement.
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

----------------------------
## II - Description du contenu du projet

### Makefile
Il est souhaitable de creer pas mal de commandes au sein de votre Makefile, dans l'objectif de pouvoir interagir de maniere assez complete avec votre projet. Il n'est pas obligatoire de recreer toutes ces commandes, mais c'est ce que j'ai fait. Exemple de commandes :

1. build -> construit les images Docker pour les conteneurs.
2. up -> lance le projet (prevoir ou non l'argument -d pour lancer en arriere plan).
3. start -> demarre le projet deja "build".
5. down -> arrete et supprime les dockers.
6. reload -> reconstruit les images dockers et lance le projet.
7. status -> affiche les informations sur les conteneurs en cours d'exécution.
8. logs -> affiche les journaux des conteneurs.
9. clean -> arrête et supprime les conteneurs Docker et supprime tous les fichiers Docker sans étiquette.
10. cleanvol -> supprime les données persistantes dans les dossiers spécifiés (volumes).
11. prune -> supprime tous les conteneurs et images Docker non utilises presents sur le systeme.
12. fclean -> combine les commandes down, prune et cleanv.
13. volume -> crée les dossiers nécessaires pour stocker les données persistantes des conteneurs.

**Astuce :**
Creez une commande "help" contenant cette commande :
`@ awk 'BEGIN {FS = ":.*##";} /^[a-zA-Z_0-9-]+:.*?##/ { printf "  \033[36m%-15s\033[0m %s\n", $$1, $$2 } /^##@/ { printf "\n\033[1m%s\033[0m\n", substr($$0, 5) } ' $(MAKEFILE_LIST)`
Rajoutez des commentaires sur chacune de vos commandes comme cela :
``up: ## Launch Inception in background``
Et la, comme par magie, lorsque vous faites ``make help``, cela donne :
```
  up               Launch Inception in background
  build            Build Inception (use make start after, to launch)
  start            Begin Inception
  down             Stop Inception
  reload           Restart Inception
  status           Display Inception status
  logs             See Inception logs
  clean            Stop Inception & Clean inception docker (prune -f)
  cleanv           Remove persistant datas
  prune            Remove all dockers on this system (prune -a)
  fclean           Remove all dockers on this system & Remove persistant datas
  volumes          Prepare folders needed
 ```

### Le fichier Dockerfile
Un Dockerfile est un fichier texte qui contient les instructions nécessaires pour construire une image Docker. C'est en quelque sorte le Makefile d'un container.
Le Dockerfile est utilisé pour automatiser le processus de construction d'une image Docker. Il spécifie les dépendances de l'application, les étapes d'installation, la configuration de l'environnement et les commandes pour exécuter l'application.
**Un Dockerfile ne sert a deployer, normalement, qu'un seul et unique processus, application ou service.**

Voici un exemple, assez complet mais volontairement simplifie, de ce a quoi devrait ressembler votre fichier `Dockerfile` pour le service Nginx :
```
FROM	NOM_DISTRIBUTION

RUN	apt update; apt install -y NOM_DES_PAQUETS_A_INSTALLER

RUN	mkdir -p NOM_DOSSIER_A_CREER; 

RUN	COMMANDE_POUR_LA_CREATION_DES_CERTIFICATS_SSL

COPY	conf/nginx.conf DOSSIER_DE_DESTINATION/default.conf

ENTRYPOINT	["COMMANDE_A_EXECUTER"]
```

Les principales instructions :

**1. FROM**

L'instruction FROM est utilisée pour spécifier l'image de base à partir de laquelle le conteneur sera construit. Elle doit être la première instruction dans le fichier Dockerfile, et spécifie l'image à utiliser. Pour le projet ca sera donc debian:buster ou alpine:3.17 (au 02/03/2023, car il faut utiliser l'avant derniere version stable pour Alpine, et la derniere est actuellement la version 3.17.2)

**2. RUN**

L'instruction RUN est utilisée pour exécuter des commandes dans le conteneur lors de sa construction. Les commandes sont exécutées dans un shell à l'intérieur du conteneur. Par exemple, ``RUN apt-get update && apt-get install -y nginx`` installe le serveur web nginx dans le conteneur. Il est possible d'executer plusieurs commandes d'affile, en utilisant `&&` ou `;`.
Astuce: pensez a utiliser les arguments de validation automatique des choix, tel que `-y` pour apt ou apt-get sur une base Debian.

**3. COPY**

L'instruction COPY est utilisée pour copier des fichiers depuis l'hôte vers le conteneur. Vous en aurez besoin pour copier les fichiers de configuration et les scripts shell a l'interieur de votre Docker, chaque fois que cela est necessaire.

**4. ENTRYPOINT**

L'instruction `ENTRYPOINT` est utilisée pour spécifier la commande à exécuter lorsque le conteneur est lancé. Elle définit la commande qui sera exécutée par défaut lors de l'exécution du conteneur. Utile, voir essentiel, pour lancer le service contenu dans le Docker. **En effet, si aucun service ou aucune application n'est executee au sein de votre Docker, ce dernier se fermera automatiquement, une fois la liste des instructions terminees !**. La formulation est un peu speciale, si vous avez besoin d'executer votre commande avec des arguments. Pour etre certain que la commande soit transmises correctement, il faudra placer l'ensemble entre `[]`, placer chaque partie entre `"` et mettre des virgules en guise de separateur, par exemple, pour Nginx : `ENTRYPOINT	["nginx", "-g", "daemon off;"]`. L'instruction `ENTRYPOINT` est normalement place a la toute fin du Dockerfile.

**5. CMD**

Instruction assez similaire a `ENTRYPOINT`. En l'absence de `ENTRYPOINT`, joue le meme role. Par contre, si l'instruction `CMD` est passee apres `ENTRYPOINT`, elle permet de passer des arguments a `ENTRYPOINT`. Il est a noter que si le Docker est lance aves des arguments, ces derniers ecraseront les arguments fournis par `CMD`.

**6. EXPOSE**

Permet d'ouvrir les ports specifies du containeur. Cette instruction est facultative si les ports ont ete specifies dans le docker-compose.


### Le docker-compose
Comme vu precedement, un container est destine a faire tourner une seule application ou un seul service, hors il est souvent necessaire pour le bon deroulement d'un projet d'en disposer de plusieurs. C'est la que rentre en jeu le docker compose.
Docker Compose est un outil destiné à définir et exécuter des applications Docker à plusieurs conteneurs. Dans Compose, vous utilisez un fichier YAML pour configurer les services de votre application. Ensuite, vous créez et vous démarrez tous les services à partir de votre configuration en utilisant une seule commande. C'est un peu comme un Makefile qui appelerait plusieurs autres Makefile pour compiler les librairies necessaires au fonctionnement du programme.
Pour info, YAML est un language de programmation regulierement utilise en scripting, il est tres proche du language humaim et son utilisation pour les Dockers est tres facile.

Ce que vous allez integrer dans votre docker-compose :
1. Le(s) reseau(x) virtuel(s) sur le(s)quel(s) va evoluer votre projet

2. Les volumes, pour la persistance des donnees (prise en compte des modifications sur les fichiers du site, et des donnees presentes dans la database, entre autres).

3. Les services, Nginx, MariaDB, Wordpress (incluant le service PHP) pour la partie obligatoire. Le serveur FTP, Redis et un (ou plusieurs) service de votre choix pour les bonus.

Voici un exemple, volontairement simplifie, de ce a quoi devrait ressembler votre fichier `docker-compose.yml` :
```
networks:
  inception:

volumes:
  database:
    driver_opts:
      type: none
      o: bind
      device: REPERTOIRE_SOURCE_DU_VOLUME
  website:
    driver_opts:
      type: none
      o: bind
      device: REPERTOIRE_SOURCE_DU_VOLUME

services:
  nginx:
    build: requirements/nginx/
    container_name: nginx
    ports:
      - PORTS_A_OUVRIR
    volumes:
      - "website:/var/www/html"
    depends_on:
      - wordpress
    networks:
      - inception
    restart: always

  wordpress:
    build: requirements/wordpress/
    container_name: wordpress
    ports:
      - PORTS_A_OUVRIR
    volumes:
      - "website:/var/www/html"
    depends_on:
      - mariadb
      - redis
    networks:
      - inception
    restart: always
    env_file: .env
```
**Attention, l'utilisation du language YAML requiert une identation parfaite pour fonctionner !**

### Le fichier .dockerignore
Le fichier .dockerignore est utilisé pour exclure des fichiers et des répertoires spécifiques de l'image Docker qui est construite à partir d'un répertoire source. Cela permet de réduire la taille de l'image et d'éviter d'inclure des fichiers inutiles ou sensibles. Il fonctionne exactement comme un fichier .gitignore.

Exemple de fichier .dockerignore :
```
logs/
README.md
secret.txt
.DS_Store
```


### Le fichier .env (gestion de l'environnement)
Le fichier .env est utilisé pour stocker des variables d'environnement qui seront utilisées lors de la création et du lancement de conteneurs Docker. Il permet de définir des valeurs de variables d'environnement pour des images Docker spécifiques sans avoir à les définir explicitement dans le fichier docker-compose.yml ou lors de la commande docker run. Cela presente un grand interet lorsque l'on veux par exemple transmettre les futurs mots de passe de nos services.

Exemple de fichier .env :
```
DOMAIN_NAME=XXXXX.42.fr

MYSQL_HOST=mariadb
MYSQL_ROOT_PASSWORD=123456

WP_DB_NAME=wordpress
WP_DB_USER=XXXXXX
WP_DB_PASSWORD=123456
```

----------------------------

## Les fichiers de configuration / Les scripts

Ces deux approchent, liees a la configuration des Dockers, sont complementaires. Dans certains cas, il est possible de se passer de fichiers de configuration et de script, pour d'autres cas, l'un, l'autre voir les deux seront necessaires, a vous de juger.

**1. Les fichiers de configuration**

Certains services necessitent de modifier les fichiers de configuration, deux possibilites s'offrent alors a nous.

a. Modifier le fichier par le biais d'un script

On peut pour cela utiliser tous les outils classiques d'un shell, tels que `AWK`, `SED`,... Et toutes les astuces apprisent lors des series Shell de la piscine, ou mieux, celles acquises lors de la conception du Minishell. Cette solution peut etre choisie s'il y a tres peu de lignes a modifier.

b. Recuperer le fichier en amont, le modifier, et le copier au bon endroit au deploiement.

Un choix plus raisonnable, lorsqu'il s'agit de supprimer, ajouter ou modifier de nombreuses lignes de configuration pour notre service. Cela va etre le cas pour le service Nginx, et pour la partie PHP du service Wordpress.

**2. Les scripts**

Tres utiles, voir indispensables pour le deploiement de certains services. Ils permettent d'eviter la creation de trop nombreuses images Docker intermediaires, a chaque usage notamment des instructions RUN. Leur utilisation est egalement recommande pour debuguer nos Dockers. Un peu a la maniere d'un `printf` en C, l'utilisation de commande `echo` a l'interieur de ces scripts permets de mieux localiser les problemes, de connaitre le stade de deploiement de notre Docker, et de verifier le passage dans certaines boucles conditionnelles.

Il faut faire attention cependant au systeme sur lequel ils seront executes. En effet, certaines commandes different entre `debian:buster` et `alpine` c'est pourquoi le portage de ces scripts d'un systeme a l'autre demandera des adaptions. Dans tous les cas, il s'agit simplement de scripts shell.


Nginx

MariaDB

Wordpress

Les bonus :
- Redis
- Serveur FTP
- Site statique
- Portainer
- Autres idees de bonus
