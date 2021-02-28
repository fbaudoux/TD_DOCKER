Docker nous permet de démarrer des conteneurs qui exécutent des applications sur une machine sans avoir besoin de les installer.


# Les bases

Commençons, comme tout bon informaticien, par lancer le hello-world.
Avec Docker cela revient à écrire quelque chose comme:

```docker run hello-world```

Cette commande demande au programme docker ( qui doit donc être installé sur la machine ) de démarrer un conteneur basé sur l'image hello-world

Pour lancer ce conteneur, docker a donc besoin d'une image, ici "hello-world".  
Si il ne la trouve pas, il va la télécharger dans un repository.
Le repository par défaut est le docker hub. 

![image](uploads/cbaaf588c60f85a85036219a2b26a3a9/image.png)

Le docker hub dispose d'un site web qui permet de rechercher toutes les images mises à disposition.  
https://hub.docker.com/

Mais revenons à l'exécution de notre Hello world.  
On voit sur les 5 premières lignes, le téléchargement de l'image depuis le repository.  
![image](uploads/05ff35c9431efcc7e79351b86991b098/image.png)

Si je lance une seconde fois, la même commande, que se passe t il ?

![image](uploads/a8611931d06c20fc785a89853b4f3042/image.png)


L'image n'est pas téléchargée à nouveau, cela signifie que Docker possède une liste d'images en local.

Nous pouvons savoir combien d'images différentes notre docker local a dans son cache :    
```docker info``` 

![image](uploads/d73a05276f71d1f95d9737fefc48e24e/image.png)

On comprend en regardant ces infos que les images et les conteneurs sont 2 notions différentes pour Docker, et que ces notions n'ont pas le même cycle de vie puisque les conteneurs semblent pouvoir avoir plusieurs états ( running,paused,stopped )

Nous allons donc réaliser quelques manipulations pour bien comprendre ces notions. 

## Les images 

Si l'on veut voir la liste de toutes les images dans le cache de notre Docker nous avons la commande:  
```docker images```

![image](uploads/caa7d48375fe63ab7a0115235b73d9dd/image.png)

Dans l'exemple ci dessus, on voit qu'il a 4 images.
Donc nous pouvons lancer chacune de ces applications depuis notre machine sans avoir à les installer.
Une image est un modèle d'application et la commande ```docker run``` démarre une instance d'application à partir d'un modèle. Ces instances sont appelées des conteneurs.

Pour faire l'analogie avec la programmation orientée objet, on pourrait voir ça comme :
* l'image docker est une classe
* le conteneur docker est une instance de cette classe


### Commandes utiles à la gestion des images

#### Lister les images 
Pour obtenir la liste des images, on a déja vu ```docker images``` . La commande peut prendre en troisième argument un filtre sur le nom des images à rechercher.  
Par exemple :  
![image](uploads/df005ed42541824216b854eefaeb72d4/image.png)

Si on analyse le contenu des différentes informations qui nous sont données

* REPOSITORY  : le nom de l'image Docker, éventuellement préfixée par le nom du repository d'ou elle provient. Dans le cas du Docker hub , il n'y a pas de prefix
* TAG : généralement la version de l'image, ou pourra également trouvé le mot clé LATEST qui signifie que c'est la dernière version de l'image mise à disposition sur le repository
* IMAGE ID : Un sha 256 pour cette image.
* CREATED : La date de création de l'image dans son repository. Ce n'est pas la date à laquelle l'image a été téléchargée sur votre Docker.
* SIZE : La taille de l'image dans le cache Docker local

Pourquoi, il y a un IMAGE ID alors qu'on a déjà un REPOSITORY et un TAG ?  
Les images peuvent être construites localement ( on verra cela par la suite ), et si l'on construit plusieurs fois la même image, on peut se retrouver avec des situations comme la suivante :  
![image](uploads/fcc16239a24eff697ec7bccd29cf06fb/image.png)

Avant de construire une image satisfaisante du logiciel(celle qui a le tag develop-snapshot), j'ai construit 4 images du même logiciel qui ont des sha256 différent.


#### Supprimer les images 

La syntaxe pour la suppression d'une image :

```docker rmi IMAGE ID```

D'ailleurs l'image de Hello World ne m'est plus utile, je veux la supprimer  

```docker images Hello*```   , pour récuperer l' ID  
![image](uploads/68a2146e9495563f0faefee418c9ed62/image.png)


```docker rmi XXXXXXXXX``` pour la suppression    

![image](uploads/cb51d1dd8a5f338bdf1a8bfef9c21cc1/image.png)


Non seulement, Docker ne veut pas me supprimer l'image mais en plus il me dit que cette image est utilisée par conteneur qui est stoppé !!  
Nous allons avoir besoin de creuser un peu plus cette notion de conteneur avant de pouvoir supprimer cette image.

## Les conteneurs


### Lister les conteneurs
Je peux lister les images docker, je dois pouvoir lister les conteneurs qui sont issus de ces images.
La commande qui permet de lister les conteneurs en cours d'exécution est :

```docker ps```

![image](uploads/b9a7d968481832a5e9f8bb80f5bb438c/image.png)

* CONTAINER ID : Un sha 256 pour ce conteneur
* IMAGE        : Le nom de l'image , plus le tag de l'image , séparés un _:_
* COMMAND      : Le nom de la commande qui est exécuté par le conteur à son démarrage ( le nom de l'application en fait )
* CREATED      : La date de création du conteneur sur votre machine
* STATUS       : le cycle de vie ( en cours d'exécution )
* PORT         : le/les ports exposés par le conteneur 
* NAMES        : le nom du conteneur


Pourquoi je n'ai rien dans la liste ? Pourtant j'ai lancé un conteneur Hello World !

En réalité, le conteneur a démarré, il a lancé la commande lui permettant d'exécuter l'application et ... 
l'application s'est terminée. 
L'application Hello World affiche juste un texte puis se termine donc le conteneur avait terminé son travail, il s'est stoppé.

Pourquoi je ne peux pas supprimer l'image Hello World si il n'y a plus de conteneur actif pour cette image ?

C'est parce que le conteneur existe toujours, d'ailleurs on peut le voir en exécutant la commande :  

```docker ps -a```

![image](uploads/49e01c8575933b1df79301fcdb95f02b/image.png)

A quoi ça sert de garder un conteneur stoppé ?

En premier lieu, cela sert pour l'analyse des logs.
Un conteneur arrêté peut l'être à cause d'un crash de l'application et il faut être en mesure d'analyser ce qui a causé le crash.

La commande ```docker logs ID_CONTENEUR``` nous permet d'accéder aux logs d'un conteneur

![image](uploads/03c168868cfd38976eb71838bca43361/image.png)


Si l'analyse de logs ne suffit pas , on peut même redémarrer un conteneur stoppé grâce à :  
```docker start ID_CONTENEUR```

Il est à noter que contrairement à ```docker run``` , ```docker start``` démarre un conteneur en mode daemon et on ne voit donc pas le résultat sur la sortie standard

Pour voir le résultat sur la sortie standard, il faudra utiliser ```docker start -ai``` comme dans l'exemple ci-dessous.   

![image](uploads/2c53ddb9f6fdc0447672f6a5728c8edd/image.png)

### Supprimer les conteneurs

On comprend bien l'intérêt de garder le conteneur pour faire des analyses après l'arrêt, mais il nous faut toutefois un moyen de faire le ménage.

```docker rm ID_CONTENEUR``` va nous permettre de supprimer définitivement le conteneur

On peut également décider au moment du lancement que le conteneur devra être supprimé dès qu'il s'arrête, en utilisant l'option --rm 

```docker run --rm hello-world```

On ne retrouve pas de trace de ce conteneur lorsque son exécution est terminée.
Comme dans l'exemple ci-dessous.  

![image](uploads/681d3c4af577d68e66d2b850061d322c/image.png)

Je peux maintenant supprimer l'image Hello world

![image](uploads/75c8b3001e149355abd69b049fe667da/image.png)


### Manipuler les conteneurs

#### Lancer un conteneur en mode terminal interactif

Nous allons utiliser une image appelée busybox.
Si on fait un ```docker run busybox```, on verra la même chose qu'avec le Hello World, un conteneur qui s'exécute puis se termine. 
Mais en exécutant ensuite un ```docker ps -a```, on peut voir que la commande exectué par busybox est un shell ```sh```

![image](uploads/e603d85759554ec33cbdb6269e7baa2b/image.png)

Donc ce conteneur lance un ![image](uploads/63c2c199f0a5471b2a0d9e0a7e3b8ca7/image.png), nous pouvons interagir avec ce shell en utilisant la commande :
```docker run -ti nom_de_l_image```  

  ![image](uploads/07d429ebe1b10cc39ba7e327f37c9046/image.png)

On se retrouve dans un shell qui est en cours d'exécution par notre conteneur busybox
On peut donc lancer des commandes shell depuis ce terminal et on s'aperçoit que notre conteneur dispose de son propre système de fichiers !

![image](uploads/302cc8c0400931d6ba8c246beb39e39c/image.png)

Tant que nous sommes connectés à ce terminal, le conteneur reste en execution.
Si j'ouvre une autre fenêtre de commande et que je demande ```docker ps``` , je vais voir l'execution de mon conteneur.

![image](uploads/1b6e6e3eec1fec1eb0c791abf698c086/image.png)

Si je quitte le shell via la commande ```exit```, le conteneur busybox va s'arrêter 

je quitte 
![image](uploads/40dba7d80530f093383f8850a947259c/image.png)

le conteneur n'est plus en exécution
![image](uploads/1c22c8bce64b0779b5de21384a66f1ab/image.png)

#### Un conteneur fait partie du bétail (Pet vs Cattle)

Nous venons de voir que l'on pouvait prendre la main dans un shell et que le conteneur avait son propre système de fichier. Donc on peut faire des bêtises dans ce système de fichier.
Par exemple : 

```docker run -ti busybox```
```rm -rf bin ```
```ls``` 

![image](uploads/3d3bf992ae8158a7f1a574d3d65edc85/image.png)

-> J'ai rendu notre conteneur complétement inutilisable, je n'ai plus qu'a quitter

Le fait d'avoir un dégradé un conteneur, ne m'empêche absolument d'en relancer un parfaitement opérationnel 
 
```docker run -ti busybox``` 
```ls```

![image](uploads/97c1ae5bedadb9602ad03d16dd9c4a73/image.png)

Les images sont importantes, les conteneurs, eux , sont sacrifiables

D'accord mais si j'avais des données présentes dans le système de fichier de mon conteneur ? je ne vais pas les perdre si mon conteneur crashe?
Pas si l'on utilise correctement les volumes docker.

#### Lancer un conteneur en mappant un volume

La syntaxe permettant de donner accès à un répertoire de votre machine depuis l'intérieur d'un conteneur se fait au lancement du conteneur.  
La syntaxe est :

```docker run -v "repertoire local":"repertoire dans le conteneur" nom_de_l_image```

Dans le cas de notre busybox, cela donne quelque chose comme :

```docker run -ti  -v "C:\Users\fred\busyboxdata":"/data" busybox```

Si je lance mon conteneur avec cette ligne de commande, je constate qu'il y a un répertoire data accessible dans le conteneur.
Je peux créer un fichier dans ce répertoire et quitter le conteneur.
![image](uploads/95552be57f0eba4f02fc43e02ca8129e/image.png)

Le fichier, lui, persiste sur ma machine
![image](uploads/6e1c3c233f2b657249fc0263fdb632e9/image.png)
 
Si je relance un conteneur, je vais retrouver mon fichier

![image](uploads/c47339f051d1f3b933d7c5294ed056c3/image.png)


#### Lancer un conteneur de la vraie vie

Jusqu'à présent, nous avons utilisés des conteneurs qui se terminaient d'eux même et qui ne faisaient pas grand chose. Il est temps de passer à des à quelque chose de plus utile, nous allons démarrer un serveur Web sous docker.
Nous utiliserons le serveur Web Apache httpd.
Rendons nous sur le site du docker hub et trouvons le nom de l'image

![image](uploads/b00abba13f300b0a86b05324f15b9ec2/image.png)


```docker run httpd``` 

![image](uploads/e73f15c3d8858ef54476d88294ed44b2/image.png)

Première constatation, mon terminal de commande est bloqué. Je dois en ouvrir un autre pour trouver le port exposé par mon conteneur

![image](uploads/dfd4496c705b1f9afcca6a9d36c33071/image.png)

Il est donc indiqué que le conteneur utilise le port 80

Mais comment je peux accèder à ce port 80 ? Je ne connais pas l'adresse IP du conteneur. 
Si je tente d'accéder à http://localhost:80 , je suis en erreur

![image](uploads/9a34815449c0913a0cf32e7bb17e7446/image.png)

#### Mapper les ports
En réalité, les conteneurs sont isolés de la machine hôte. Si l'on veut pouvoir accéder à un conteneur depuis une machine, il faut déclarer un mapping entre un port de la machine hôte et un port du conteneur.

La syntaxe est :

```docker run -p "port local":"port dans le conteneur" nom_de_l_image```

Donc je dois stopper mon conteneur et en relancer un avec ce mapping de port.

Pour stopper mon conteneur, je peux utiliser la commande :
```docker stop ID_CONTENEUR```

Pour le stopper et le supprimer, je peux utiliser la commande :

```docker rm -f ID_CONTENEUR```


Lorsque je relance le conteneur, je vais ajouter quelques options à la ligne de commande.
L'option --rm  indique que si le conteneur est stoppé, je veux le supprimer
L'option -d indique que je veux lancer le conteneur en mode daemon, c'est à dire que je ne veux pas qu'il bloque mon terminal de commande.

Je relance donc avec :

```docker run -p 80:80  --rm -d httpd``` 

![image](uploads/4807b262eaabb1fe905510224d01e216/image.png)

On peut voir que le port indique maintenant 0.0.0.0:80 -> 80/tcp  , la redirection de port est donc en place

Donc si je lance mon navigateur sur http://localhost:80 ou http://127.0.0.1:80, j'obtiens :

![image](uploads/ebbf177b55ae54e95bccbaccd5c166aa/image.png)

![image](uploads/8f5beb339371e2bca27e39cf6331d22f/image.png)

#### Mapper les volumes

Pratique ce serveur Web sans installation, mais il ne diffuse que la page par défaut.
Je voudrais lui faire diffuser ma propre page.

ok , on peut se connecter au conteneur, trouver ou se situent les pages et modifier la page
Comme il est lancé en mode daemon, pour obtenir un terminal interactif, je peux utiliser :
```docker exec -ti "id conteneur" bash```

Si le conteneur dispose d'un shell bash cela va fonctionner ( on peut aussi essayer avec sh )

![image](uploads/a819f97695e6b0069841f9925b2c020c/image.png)

Par défaut, http va diffuser les fichiers qui se situent ici : /usr/local/apache2/htdocs
Il y a un fichier index.html, on peut l'éditer. .... non, il n'y a pas d'éditeur disponible sur le conteneur
On peut créer un fichier en local et le copier dans le conteneur pour remplacer le fichier index.html

```docker cp "fichier local"   "id conteneur":"chemin fichier sur le conteneur"```

Franchement, pour le développement ce n'est pas pratique. Je ne vais pas faire une copie à chaque fois que je vais modifier mes pages html. On doit pouvoir faire mieux.

De la même manière que l'on peut mapper un port de la machine hôte avec un port du conteneur, on peut mapper un répertoire de la machine hôte avec un répertoire du conteneur.

```docker run -p 80:80 -v "C:\Users\fred\monsite":"/usr/local/apache2/htdocs"  --rm -d httpd```

Cette fois cela fonctionne de façon assez pratique !

![image](uploads/aaa13bdf49726bdf0594ffb9312fcd74/image.png)

## Faire communiquer les conteneurs entre eux

Pour communiquer de notre machine vers un conteneur Docker, on a vu que l'on devait mapper des ports et appeler notre conteneur via localhost ou 127.0.0.1.
Mais alors si on lance plusieurs conteneurs sur notre machine, comment vont ils communiquer ensemble ?

Pour que 2 conteneurs puissent communiquer ensemble, cela va se passer, comme souvent avec Docker, lors du lancement du conteneur

Premièrement, il faut lancer un conteneur en lui donnant un nom grâce à l'option --name 

Exemple :  

docker run --name serveurhttp  -d httpd

![image](uploads/0214740f5231c22ce0b545490be26dd9/image.png)


Ensuite, si l'on veut lancer un conteneur qui doit pouvoir communiquer avec notre conteneur appelé serveurhttp, il faudra le lancer avec l'option --link serveurhttp:nom_du_lien

Dans l'exemple ci-dessous, je lance un busybox qui peut dialoguer avec le serveur httpd via l'alias apache

![image](uploads/c523cc3083b69c23b9f27fdede39ad25/image.png)

De plus, même si mon conteneur httpd n'a pas le port 80 mappé sur localhost, mon conteneur busybox est quand même capable de l'appeler

![image](uploads/f73348a1bf15a76ca62d4c5b768318e1/image.png)

Il y a donc un réseau interne au fonctionnement de docker qui propose du DNS.
L'étude du fonctionnement réseau de Docker sera par manque de temps hors du périmètre de ce td.
 
## Construire sa propre image Docker

Pour construire sa propre image Docker, il faut créer un fichier de directives nommé Dockerfile

Le Dockerfile précise une liste d'actions à réaliser les unes à la suite des autres permettant d'obtenir l'image que l'on veut.

La première ligne du Dockerfile est du type :   
```from : image_parent``` 

on part toujours d'une image existante, qui contient juste les outils nécessaires à construire notre image

Voici par exemple celle de Hello world
![image](uploads/ba035eb4b42d5c14443f1c4255b83417/image.png)

La première directive indique que l'image de base utilisée est __scratch__ , c'est à dire l'image la plus minimale que l'on puisse trouver.
La seconde directive indique qu'il faut copier le fichier hello à la racine du système de fichier
Le troisième directive indique que la commande qui sera lancée au démarrage d'un conteneur sera Hello


Quelques directives intéressantes :    




## Manipuler plusieurs conteneurs à la fois grâce à compose

Mapper les ports, mapper les volumes, préciser les liens de communication entre les conteneurs, ...
Cela commence à faire à devenir un peu complexe surtout si l'on veut démarrer un écosystème un peu élaboré ou l'on trouve par exemple : serveur web + serveur applicatif + base de données ....

Docker a un outil prévu pour cela, il s'agit de docker-compose.
Cet outil fonctionne avec un fichier de description docker-compose.yml


Créons donc un fichier de ce type :

```
version: '3'
services:
  web:
   image: nginx:latest
   ports:
   - "9090:80"

   volumes:
   - ./code:/code
   - ./site.conf:/etc/nginx/conf.d/site.conf
```


Les conteneurs peuvent se parler par leur nom et sans mapping des ports ???? 
C'est la magie de docker-compose.

Vous pouvez ajouter un fichier index.php dans le répertoire code et relance votre docker-compose up,  NGINX doit pouvoir le servir désormais.

![image](uploads/b1aac17fbc18145a81e6c271555c64d8/image.png)

Pour que notre env de développement soit complet, il ne nous manque plus que la base de donnée.

```
version: '3'
services:
  web:
   image: nginx:latest
   ports:
   - "9090:80"

   volumes:
   - ./code:/code
   - ./site.conf:/etc/nginx/conf.d/site.conf
 

  php:
   image: php:7-fpm
   volumes:
   - ./code:/code

  db:
   image: mysql:5.6
   environment:
    MYSQL_ROOT_PASSWORD: example
```

Vous noterez au passage l'utilisation de la directive __environment__ dans le fichier docker-compose.
Cela permet de passez des valeurs à des variables d'environnement connues à l'intérieur du conteneur. Pratique pour la configuration !!



Moi, je vais utiliser un dernier conteneur qui va me permettre de faire cette vérification 

```
version: '3'
services:
  web:
   image: nginx:latest
   ports:
   - "9090:80"

   volumes:
   - ./code:/code
   - ./site.conf:/etc/nginx/conf.d/site.conf
 

  php:
   image: php:7-fpm
   volumes:
   - ./code:/code

  db:
    image: mysql:5.6
    environment:
      MYSQL_ROOT_PASSWORD: example

  adminer:
    image: adminer
    ports:
      - 9091:8080

```

Maintenant que vous savez lire un fichier docker-compose, vous trouverez l'URL qui vous permettra d'accèder à la page de connexion à la base de données.

![image](uploads/96985e7ee470e18b62e17123f3760ccb/image.png)

## Utile à savoir 

__Manip3__ : Ou l'on comprend que l'on peut créer une nouvelle image à partir d'un conteneur en cours d'execution

* docker run -ti busybox
* touch hello.txt

dans un autre terminal:

* docker ps
![image](uploads/282e24bd3f6a42adeecc38b205cb7317/image.png)

* docker diff "container id"
-> on voit les différences entre le conteneur et l'image sur laquelle il est basé

* docker commit "container id" mybusybox
* docker images 
![image](uploads/136298577018f12f389391c8c266d430/image.png)

* docker run -ti mybusybox
* ls

On a toujours notre fichier hello.txt

Donc on peut se retrouver avec des dizaines d'images issues d'une même image originelle.
On va avoir besoin d'un moyen de faire le ménage.

* docker rmi "Id de l'image"

__Manip4__ : Ou l'on découvre comment qu'un repository n'est pas obligatoire
* docker save --output mybusybox.tar mybusybox
* docker rmi mybusybox
* docker images | grep busy
* docker load --input mybusybox.tar
* docker images | grep busy


