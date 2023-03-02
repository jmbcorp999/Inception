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

Le fichier Dockerfile
Un Dockerfile est un fichier texte qui contient les instructions nécessaires pour construire une image Docker. C'est en quelque sorte le Makefile d'un container.
Le Dockerfile est utilisé pour automatiser le processus de construction d'une image Docker. Il spécifie les dépendances de l'application, les étapes d'installation, la configuration de l'environnement et les commandes pour exécuter l'application.
Un Dockerfile ne sert qu'a deployer, normalement, qu'un seul et unique processus, application ou service.

- Les commandes disponibles
- FROM
- RUN
- ENTRYPOINT
- COPY
- WORKDIR

Le docker-compose

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
