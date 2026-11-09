## Contexte

Le projet consiste à concevoir un **serveur FTP** accompagné de son **client**, permettant l'envoi et le téléchargement de **fichiers et de répertoires** entre une machine cliente et une machine serveur. Les transferts s'effectuent au travers du protocole **TCP**.

Le serveur et le client sont écrits en langage **C**. La bibliothèque **`ncurses`** est utilisée pour réaliser les affichages interactifs côté client.

L'objectif est de fournir :
- un serveur capable de recevoir, de stocker et de servir des éléments (fichiers et dossiers) en respectant les droits d'accès,
- un client doté d'une interface interactive permettant de parcourir le contenu, de sélectionner des éléments et de suivre visuellement l'avancement des transferts.


## Architecture

- **network** : implémente les fonctionnalités de communication réseau (établissement de la connexion TCP, envoi et réception des données entre le client et le serveur).
- **server** : gère l'écoute des connexions entrantes, l'authentification par mot de passe, la gestion des droits sur les éléments et le traitement des requêtes (envoi, téléchargement, suppression, listing).
- **client** : gère la connexion au serveur, l'interface utilisateur `ncurses`, la sélection des éléments et l'affichage animé des transferts.
- **transfer** : implémente l'envoi des données sous forme de blocs (*chunks*) au format binaire, ainsi que la compression optionnelle des éléments avant envoi.


## Modes de connexion

Le serveur FTP doit pouvoir fonctionner selon deux modes de connexion :
- **Connexion unique** : une seule connexion cliente est acceptée à la fois sur le serveur.
- **Connexions simultanées** : plusieurs clients peuvent être connectés en même temps sur le serveur.


## Modes d'envoi et de transfert

Plusieurs modes de transfert sont possibles. Ils sont sélectionnés au moyen d'**options `getopt`** passées en ligne de commande :

- **Envoi brut** : les éléments sont envoyés sur le serveur tels quels, sans traitement particulier.
- **Téléchargement et actions sur le serveur** : il est possible de récupérer plusieurs éléments depuis le serveur et, en fonction des droits, d'effectuer les actions suivantes :
    - télécharger un fichier ou un dossier,
    - supprimer un élément sur le serveur.
- **Envoi compressé** : les éléments sont compressés avant envoi.
    - Un algorithme de compression doit être utilisé (de type *zip* ou équivalent).
    - Le transfert s'effectue au format binaire, découpé en blocs (*chunks*).


## Options du programme

Le programme (client) est piloté par des options en ligne de commande. On attend au minimum :
- la spécification du **mot de passe** de connexion,
- le **chemin du fichier ou dossier source** (utilisé pour l'envoi ou le téléchargement),
- le **mode de transfert** (brut, téléchargement/suppression, compressé).

### Liste des options (client)

Les options suivantes sont gérées au moyen de `getopt`/`getopt_long` :

- `-h` ou `--help` : affiche l'aide du programme ainsi que la syntaxe d'exécution.
- `-s` ou `--server` : spécifie l'adresse (IP ou nom DNS) du serveur FTP auquel se connecter.
- `-P` ou `--port` : spécifie le port TCP du serveur. En l'absence de cette option, un port par défaut est utilisé (ex. `2121`).
- `-p` ou `--password` : spécifie le mot de passe de connexion. S'il n'est pas fourni, il est demandé interactivement.
- `-f` ou `--source` : spécifie le chemin du fichier ou dossier source (utilisé pour l'envoi ou comme destination de téléchargement).
- `-u` ou `--upload` : envoie l'élément source vers le serveur (**mode brut**).
- `-z` ou `--compress` : envoie l'élément source vers le serveur en le **compressant** au préalable (transfert binaire par blocs).
- `-d` ou `--download` : télécharge un ou plusieurs éléments depuis le serveur.
- `-r` ou `--remove` : supprime un ou plusieurs éléments sur le serveur (selon les droits).
- `-i` ou `--interactive` : lance l'interface interactive `ncurses` (menu) après connexion.

**NB** : si aucune des options d'action (`-u`, `-z`, `-d`, `-r`) n'est fournie, le programme bascule par défaut en **mode interactif** après la connexion.

### Liste des options (serveur)

- `-h` ou `--help` : affiche l'aide et la syntaxe d'exécution.
- `-P` ou `--port` : spécifie le port TCP d'écoute (défaut : `2121`).
- `-p` ou `--password` : spécifie le mot de passe exigé des clients.
- `-r` ou `--root` : spécifie le répertoire racine servi par le serveur.
- `-m` ou `--multi` : autorise plusieurs connexions simultanées (par défaut, une seule connexion à la fois).

Deux manières d'utiliser le programme sont attendues :

### Mode ligne de commande direct

L'ensemble des paramètres est fourni directement sur la ligne de commande (options correspondantes + mot de passe). Le transfert est alors lancé sans passer par le menu interactif.

### Mode interactif

L'utilisateur se connecte au serveur en saisissant le mot de passe et le chemin du fichier source. Une fois connecté, un **menu interactif** (à la manière des menus de jeux d'arcade) apparaît et permet de :
- lister le contenu du serveur,
- se déplacer dans le contenu qui lui appartient (aussi bien côté source que côté destination),
- sélectionner des fichiers de la source à envoyer,
- télécharger des fichiers ou des dossiers depuis le serveur,
- supprimer des éléments sur le serveur.


## Affichage et animations

Les affichages côté client reposent sur `ncurses` et doivent illustrer visuellement les transferts :

- **Envoi d'un fichier (mode brut)** : le client dessine un personnage de type *Mario* qui transporte (lance ou porte) des colis d'un point A (la source) vers un point B (la destination).
- **Envoi en mode compressé** : le client dessine plutôt un personnage de type *Sonic* qui effectue des allers-retours entre le point A et le point B pour transporter les colis.
- Les allers-retours doivent être **calculés en fonction du débit et du temps de transport estimé**, afin de déterminer la vitesse du personnage ainsi que le nombre de colis à dessiner.


## Modalités d'évaluation

L'évaluation porte sur 2 éléments :
- La soutenance
    - une présentation de 10 minutes de tout le groupe (en équilibrant les temps de parole) dans laquelle :
        - vous rappellerez le contexte du projet, les grandes lignes de son fonctionnement (2 minutes)
        - vous expliquerez les choix que vous avez eu à faire et pourquoi (3 minutes)
        - vous présenterez les difficultés les plus importantes rencontrées et vos analyse à ce sujet (2 minutes)
        - vous proposerez un RETEX (retour d'expérience) dans lequel vous répondrez du mieux que possible à la question suivante : "_fort(e)s de notre expérience sur le projet, que ferions-nous différemment si nous devions le recommencer aujourd'hui ?_"
        - vous ferez une brève conclusion (1 minute)
    - vous répondrez ensuite à des questions posées par les enseignants pendant une dizaine de minutes.
- Le code
    - fourni dans un dépôt git avec l'historique des contributions de tous les membres,
    - avec un Makefile qui permet la compilation simplement avec la commande `make`
    - capacité à effectuer les traitements demandés dans le sujet,
    - capacité à traiter les cas particuliers sujets à erreur (pointeurs NULL, etc.)
    - Respect des conventions d'écriture de code
    - Documentation du code
        - Avec des commentaires au format doxygen en entêtes de fonction (si nécessaire)
        - Des commentaires pertinents sur le flux d'une fonction (astuces, cas limites, détails de l'algorithme, etc.)
    - Absence ou faible quantité de fuites mémoire (vérifiables avec `valgrind`)
    - **ATTENTION !** le code doit compiler sous Linux ! Un code non compatible avec un système Linux sera pénalisé de 5 points sur 20.


## Annexes

### Convention de code
Il est attendu dans ce projet que le code rendu satisfasse un certain nombre de conventions (ce ne sont pas des contraintes du langages mais des choix au début d'un projet) :
- **indentations** : les indentations seront faites sur un nombre d'espaces à votre discrétion, mais ce nombre **doit être cohérent dans l'ensemble du code**.
- **Déclaration des pointeurs** : l'étoile du pointeur est séparée du type pointé par un espace, et collée au nom de la variable, ainsi :
    - `int *a` est correct
    - `int* a`, `int*a` et `int * a` sont incorrects
- **Nommage des variables, des types et des fonctions** : vous utiliserez le _snake case_, i.e. des noms sans majuscule et dont les parties sont séparées par des underscores `_`, par exemple :
    - `ma_variable`, `variable`, `variable_1` et `variable1` sont corrects
    - `maVariable`, `Variable`, `VariableUn` et `Variable1` sont incorrects
- **Position des accolades** : une accolade s'ouvre sur la ligne qui débute son bloc (fonction, if, for, etc.) et est immédiatement suivie d'un saut de ligne. Elle se ferme sur la ligne suivant la dernière instruction. L'accolade fermante n'est jamais suivie d'instructions à l'exception du `else` ou du `while` (structure `do ... while`) qui suit l'accolade fermante. Par exemple :
```c
for (...) {
	/*do something*/
}

if (true) {
	/*do something*/
} else {
	/*do something else*/
}

int max(int a, int b) {
	return a;
}
```

sont corrects mais :

```c
for (int i=0; i<5; ++i)
{ printf("%d\n", i);
}

for (int i=0; i<5; ++i) {
	printf("%d\n", i); }

if () {/*do something*/
}
else
{
	/*do something else*/}
```

sont incorrects.

- Espacement des parenthèses : la parenthèse ouvrante après `if`, `for`, et `while` est séparée de ces derniers par un espace. Après un nom de fonction, l'espace est collé au dernier caractère du nom. Il n'y a pas d'espace après une parenthèse ouvrante, ni avant une parenthèse fermante :
    - `while (a == 3)`, `for (int i=0; i<3; ++i)`, `if (a == 3)` et `void ma_fonction(void)` sont corrects
    - `while(a == 3 )`, `for ( i=0;i<3 ; ++i)`, `if ( a==3)` et `void ma_fonction (void )` sont incorrects
- Basé sur les exemples ci dessus, également, les opérateurs sont précédés et suivis d'un espace, sauf dans la définition d'une boucle `for` où ils sont collés aux membres de droite et de gauche.
- Le `;` qui sépare les termes de la boucle `for` ne prend pas d'espace avant, mais en prend un après.
