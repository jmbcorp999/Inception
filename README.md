# Inception

Ce document a pour but de vous aider a comprendre le principe des Dockers, mieux apprehender le projet et peut etre vous assurer un depart plus simple en vous donnant quelques conseils et astuces qui m'ont aider a le realiser. N'y cherchez pas de solution toute faite, ni de code tout pret a copier-coller, ce n'est absolument pas l'objectif de ce document.

[I - Notions generales](#notions)

[&emsp;C'est quoi Docker ?](#definition)

[&emsp;Comment structurer notre projet ?](#struct)

[II - Description du contenu du projet](#description)

[&emsp;Makefile](#makefile)

[&emsp;Le fichier Dockerfile](#dockerfile)

[&emsp;Le docker-compose](#dockercompose)

[&emsp;Le fichier .dockerignore](#dockerignore)

[&emsp;Le fichier .env](#env)

[&emsp;Les fichiers de configuration](#conf)

[&emsp;Les scripts](#script)

[&emsp;III - Strategie et conseil d'approche](#strategie)

<br><br>  
---
<br><br>

## <a name="notions"></a>I - Notions generales

### <a name="definition"></a>C'est quoi Docker ?
Docker est une plate-forme logicielle qui vous permet de concevoir, tester et déployer des applications rapidement. Docker intègre les logiciels dans des unités normalisées appelées conteneurs, qui rassemblent tous les éléments nécessaires à leur fonctionnement, dont les bibliothèques, les outils système, le code et l'environnement d'exécution. Avec Docker, vous pouvez facilement déployer et dimensionner des applications dans n'importe quel environnement, avec l'assurance que votre code s'exécutera correctement.
Le fonctionnement de Docker repose sur le noyau Linux et les fonctions de ce noyau, comme les groupes de contrôle cgroups et les espaces de nom. Ce sont ces fonctions qui permettent de séparer les processus pour qu’ils puissent s’exécuter de façon indépendante.
En effet, le but des conteneurs est d’exécuter plusieurs processus et applications **séparément**. C’est ce qui permet d’optimiser l’utilisation de l’infrastructure sans pour autant atténuer le niveau de sécurité par rapport aux systèmes distincts.
**La principale difference avec une machine virtuelle "traditionnelle" (VMWare, VirtualBox...) repose sur le fait que chaque container est dedie a l'execution d'un seul et unique processus**, une seule application, a l'interieur d'un environnement virtuel base sur un noyau Linux. Une machine virtuelle "traditionnelle", elle, execute un systeme d'exploitation complet, et est en mesure de de faire tourner plusieurs services ou applications simultanement.

Avantages et inconvenients de Docker, par rapport a une machine virtuelle classique :
- Avantages : deploiement rapide, facile, et compatible avec la majorite des plateformes (Windows, OSX, Linux).
- Inconvenients : ne s'adapte pas a tous les types de projets, impossible de creer un environnement base sur un noyau autre que Linux.

### <a name="struct"></a>Comment structurer notre projet
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

<br><br>
---
<br><br>

## <a name="description"></a>II - Description du contenu du projet

### <a name="makefile"></a>Makefile
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

### <a name="dockerfile"></a>Le fichier Dockerfile
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


### <a name="dockercompose"></a>Le docker-compose
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

### <a name="dockerignore"></a>Le fichier .dockerignore
Le fichier .dockerignore est utilisé pour exclure des fichiers et des répertoires spécifiques de l'image Docker qui est construite à partir d'un répertoire source. Cela permet de réduire la taille de l'image et d'éviter d'inclure des fichiers inutiles ou sensibles. Il fonctionne exactement comme un fichier .gitignore.

Exemple de fichier .dockerignore :
```
logs/
README.md
secret.txt
.DS_Store
```

### <a name="env"></a>Le fichier .env (gestion de l'environnement)
Le fichier .env est utilisé pour stocker des variables d'environnement qui seront utilisées lors de la création et du lancement de conteneurs Docker. Il permet de définir des valeurs de variables d'environnement pour des images Docker spécifiques sans avoir à les définir explicitement dans le fichier docker-compose.yml ou lors de la commande docker run. Cela presente un grand interet lorsque l'on veux par exemple transmettre les futurs mots de passe de nos services.
Il faudra pour cela s'assurer de bien faire figurer la ligne ``env_file: .env`` dans votre docker-compose, pour chaque service ayant besoin de recuperer ces variables.

Exemple de fichier .env :
```
DOMAIN_NAME=XXXXX.42.fr

MYSQL_HOST=mariadb
MYSQL_ROOT_PASSWORD=123456

WP_DB_NAME=wordpress
WP_DB_USER=XXXXXX
WP_DB_PASSWORD=123456
```

### <a name="config"></a>Les fichiers de configuration / Les scripts

Ces deux approchent, liees a la configuration des Dockers, sont complementaires. Dans certains cas, il est possible de se passer de fichiers de configuration et de script, pour d'autres cas, l'un, l'autre voir les deux seront necessaires, a vous de juger.

**<a name="conf"></a>1. Les fichiers de configuration**

Certains services necessitent de modifier les fichiers de configuration, deux possibilites s'offrent alors a nous.

a. Modifier le fichier par le biais d'un script

On peut pour cela utiliser tous les outils classiques d'un shell, tels que `AWK`, `SED`,... Et toutes les astuces apprisent lors des series Shell de la piscine, ou mieux, celles acquises lors de la conception du Minishell. Cette solution peut etre choisie s'il y a tres peu de lignes a modifier.

b. Recuperer le fichier en amont, le modifier, et le copier au bon endroit au deploiement.

Un choix plus raisonnable, lorsqu'il s'agit de supprimer, ajouter ou modifier de nombreuses lignes de configuration pour notre service. Cela va etre le cas pour le service Nginx, et pour la partie PHP du service Wordpress. Il suffira d'utiliser l'instruction `COPY` pour integrer ce fichier au Docker.

**<a name="script"></a>2. Les scripts**

Tres utiles, voir indispensables pour le deploiement de certains services. Ils permettent d'eviter la creation de trop nombreuses images Docker intermediaires, a chaque usage notamment des instructions RUN. Leur utilisation est egalement recommande pour debuguer nos Dockers. Un peu a la maniere d'un `printf` en C, l'utilisation de commande `echo` a l'interieur de ces scripts permets de mieux localiser les problemes, de connaitre le stade de deploiement de notre Docker, et de verifier le passage dans certaines boucles conditionnelles.

Il faut faire attention cependant au systeme sur lequel ils seront executes. En effet, certaines commandes different entre `debian:buster` et `alpine` c'est pourquoi le portage de ces scripts d'un systeme a l'autre demandera des adaptions. 

Dans tous les cas, il s'agit simplement de scripts shell qui devront etre copies grace a l'instruction `COPY`, puis executes avec l'instruction `RUN`. Il existe une autre solution, qui consiste a creer un volume dans le docker-compose, voici un exemple (totalement fictif) :    
```
volumes:
  - "~/data/mariadb/configuration.conf:/etc/mariadb/defaut.conf"
```

<br><br>  
---
<br><br>

## <a name="strategie"></a>III - Strategie et conseil d'approche

La strategie de travail que je vais enoncer est la mienne, ce ne sont que des conseils et il existe probablement des moyens plus efficaces, a vous de les trouver. A savoir qu'il m'a fallu 8 jours pour mettre en place le projet, full bonus.

### Mise en place de votre machine virtuelle hote.

Le projet ne s'appelle pas Inception pour rien, nous allons donc devoir commencer par deployer une machine virtuelle, a l'aide de VirtualBox, comme pour le projet Born2BeRoot. Rassurez vous, rien n'oblige a configurer votre machine en ligne de commande, vous pouvez tout a fait installer une interface graphique comme Gnome par exemple. De toute facon, vu qu'il faudra tester votre Wordpress depuis un navigateur web, il faudra bien l'installer a un moment ou un autre. Autant le faire directement et profiter de la simplicite d'un environnement graphique pour poser les bases de votre projet.

Pas facile de trouver une distribution qui marche correctement sur les ordinateurs de l'ecole, **il faudra donc privilegier une distribution legere, peu gourmande en RAM**. Exemple de distributions qui rentrent dans ces criteres : Debian, Mint, Lubuntu... Faites vos recherches et vos tests par vous meme. 

Quoi qu'il en soit, **je vous conseille de stocker votre VM sur une cle USB3 ou un disque dur externe**, les sessions etant tres limitees en place, et les goinfres ayant tendance a digerer plutot rapidement vos donnes. Cela ne concerne que la VM de virtualbox, le stockage de vos fichiers docker pouvant rester sur votre bureau (je le conseille meme fortement), je vous expliquerai plus tard comment partager vos donnees entre votre Mac de l'ecole, et votre VM.

Je vous passe les etapes d'installation de docker et docker-compose, il y a suffisamment de tuto disponibles sur internet pour le faire.

----

### Le choix de la distribution des Dockers

Par habitude, j'ai choisi de sacrifier la vitesse de deploiement, pour privilegier le confort d'un environnement mieux maitrise, pour tous les modules dockers necessitant une configuration complexe. J'ai donc pour ces modules decide d'opter pour debian:buster. Pour certain docker plus simple a deployer j'ai tout de meme opte pour alpine.
Rien n'interdit dans le sujet de mixer ces distributions pour notre docker-compose. Voici un petit recapitulatif des pour et contre, pour faciliter vos choix :

**1. debian:buster**

&emsp;&emsp;a. Avantages :

&emsp;&emsp;&emsp;- Stable

&emsp;&emsp;&emsp;- Integre bash, avec des commandes familieres

&emsp;&emsp;&emsp;- Pas d'evolution de version possible entre le demarrage du sujet et son rendu

&emsp;&emsp;b. Inconvenients :

&emsp;&emsp;&emsp;- Plus lent a deployer

&emsp;&emsp;&emsp;- Pas de gestion native de PHP au dela du 7.3

&emsp;&emsp;&emsp;- Plus lourd (50 mo)

**2. alpine:3.17**

&emsp;&emsp;a. Avantages :

&emsp;&emsp;&emsp;- Deploiement plus rapide (em moyenne 2 fois plus rapide)

&emsp;&emsp;&emsp;- Plus leger (5 mo)

&emsp;&emsp;b. Inconvenients :

&emsp;&emsp;&emsp;- Certaines version, lors de mes tests, se sont averees instables sur le gestion de certains process

&emsp;&emsp;&emsp;- Commande de base legerement differentes de bash

----

### 1. Creer notre serveur Nginx

C'est la premiere etape a mettre en place, et aussi parmis les plus simples du projet.
Je vous conseille, aussi bien pour ce module que pour les autres, de commencer par une de ces commandes, a taper directement dans votre terminal habituel (en fonction de la distribution choisie) :

```docker run -it debian:buster bash```

ou

```docker run -it alpine:3.17 sh```

Cette commande va verifier si l'image docker est disponible, la telecharger au besoin, lancer le docker et ouvrir un prompt `bash` ou `sh`.
Vous allez pouvoir commencer a travailler, verifier les packages a installer, taper vos commandes et verifier les fichiers de configuration.

Prenons l'exemple, pour le Docker Nginx, sous debian:buster :

**Attention : les commandes suivantes ne sont valables que sur Debian ! Sur Alpine les commandes different, par exemple `apt install` est remplace par `apk add`!**

Nous allons commencer, pour l'entrainement, a lancer la commande precedente : `docker run -it debian:buster bash`. Nous nous retrouvons face un terminal bash.

Il faut d'abord, et je vous conseille d'en faire de meme pour tous vos dockers, commencer par mettre a jour la liste des paquets disponibles avec la commande `apt update` ou `apt-get update`. La difference entre `apt` et `apt-get` reside dans le fait que `apt-get` est plus bas niveau que `apt` et donc moins visuel. Pour les scripts il est generalement conseille d'utiliser `apt-get`, mais pour notre cas a nous, cela n'a pas une importance capitale. Cette etape consiste bien a mettre a jour la **liste des paquets disponibles** et non pas les paquets eux meme ! 

Pour mettre a jour les paquets, il faut utiliser la commande `apt upgrade -y`. Il faut se poser la question : est ce utile de les mettre a jour pour nos Docker, sachant que cela va ralentir leur deploiement ? Je vous laisse seuls juges. En tout cas, si vous choisissez de le faire n'oubliez pas le flag `-y` qui va permettre de valider automatiquement votre choix !

Maintenant, attaquons le coeur du sujet, installer Nginx. Vu que c'est le premier Docker, je vais vous aider de facon plus complete que je ne le ferais sur les prochains. Pour rappel, le sujet indique **Un container Docker contenant NGINX avec TLSv1.2 ou TLSv1.3 uniquement.**. Il va donc falloir installer installer Nginx ET un moyen de generer des certificats SSL. Nous pouvons installer les deux en meme temps et sans que le prompt nous demande confirmation, Il faut donc taper la commande `apt install -y nginx openssl`, et oui, encore une fois, n'oubliez pas le flag `-y`.

Pour l'instant, si nous avons suivi les etapes notre fichier Docker devrait ressembler a ca :

```
FROM	debian:buster
RUN		apt update; apt install -y nginx openssl
```
Notez que je n'utilise qu'une commande `RUN` pour limiter les surcouches crees, et donc eviter de ralentir inutilement le deploiement.

Ensuite nous allons generer un certificat SSL, ainsi que sa cle, en commencant par creer le dossier (non obligatoire, vous pouvez choisir de les placer dans un dossier deja existant, j'ai choisi de le faire pour etre plus propre) :
```
RUN		mkdir -p /etc/nginx/ssl; openssl req -newkey rsa:4096 -x509 -sha256 -days 365 -nodes -out /etc/nginx/ssl/ssl_certificat.pem -keyout /etc/nginx/ssl/ssl_key.key -subj "/C=FR/ST=Nice/L=Nice/O=42/OU=VOTRE_LOGIN/CN=VOTRE_LOGIN/"
```

Particularite de Nginx, il ne peux demarrer que si le dossier `/run/nginx` existe. Nous allons donc nous en assurer, avec une commande simple `mkdir -p /run/nginx`. L'argument `-p` va etre notre meilleur ami, a chaque fois nous aurons besoin de creer un dossier ! Il permet a `mkdir` de creer tous les dossiers parents si necessaire (avec les droits 777), et aucune erreur ne sera reportee si le dossier existe deja, donc aucun risque de plantage de creation de notre Docker en effectuant cette commande !

Maintenant, avant de demarrer notre serveur Nginx, il va falloir s'occuper de sa configuration, le plus simple etant de modifier le fichier en local, et d'utiliser la commande `COPY` pour copier notre configuration au bon endroit, a l'interieur de notre Docker. Voici une partie de notre fichier de configuration :
```
server {

	listen 443 ssl;
	listen [::]:443 ssl;

	server_name inception;

	ssl_certificate /etc/nginx/ssl/ssl_certificat.pem;
	ssl_certificate_key /etc/nginx/ssl/ssl_key.key;
	ssl_protocols TLSv1.2 TLSv1.3;

	root /var/www/html;
	index index.php index.html index.htm;

	location / {
		autoindex on;
		try_files $uri $uri/ =404;
	}
    /* Il faudra rajouter la partie concernant PHP ici  */
}
```

MariaDB

Wordpress

Les bonus :
- Redis
- Serveur FTP
- Site statique
- Portainer
- Autres idees de bonus
