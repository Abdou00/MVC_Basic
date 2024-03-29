# Architecture MVC
Au fur et à mesure que votre site Web grandit, vous allez rencontrer des difficultés à organiser votre code. Le prochain TD visent à vous montrer une bonne façon de concevoir votre site web. On appelle design pattern (patron de conception) une série de bonnes pratiques pour l’organisation de votre site.
Un des plus célèbres design patterns s’appelle MVC.

### Présentation du design pattern MVC
Il s'agit d'abord d'un acronyme, signifiant "Model View Controller", ou "Modèle Vue Contrôleur" en français.

Il s'agit surtout d'une structure que nous donnerons à nos projets pour séparer clairement les principaux composants de notre application.

Le pattern MVC permet de bien organiser son code source. Jusqu’à présent, nous avons programmé de manière monolithique : nos pages Web mélangent traitement (PHP), accès aux données (SQL) et présentation (balises HTML). Nous allons maintenant séparer toutes ces parties pour plus de clarté.

En utilisant une structure MVC, nous allons séparer les requêtes en base de données de notre code HTML et de toute "l'intelligence" de l'application.

#### Les Modèles

Les modèles seront les éléments qui se chargeront des échanges avec la base de données (CRUD). On ne mettra pas de traitement dans ces fichiers, uniquement des requêtes.

#### Les vues

Les vues contiendront uniquement le code HTML destiné à structurer les pages.

#### Les contrôleurs

Les contrôleurs, pour leur part, contiendront toute l'intelligence de l'application, le traitement des données en vue de leur affichage, par exemple.

#### Le routeur

Dans la structure MVC, un seul et unique fichier est le point d'entrée de l'application, quelle que soit la page affichée. Il est systématiquement appelé, et envoie la demande au bon contrôleur. Il est chargé de trouver le bon chemin pour que l'utilisateur récupère la bonne page, d'où le nom de routeur.

Voici un schéma qui récapitule tout ceci.
![](assets/img/mvc.jpg)

#### Structure de notre projet

Notre projet aura donc la structure ci-dessous

![](assets/img/structure.jpg)

Voici l'utilité des différents dossiers et fichiers
 - .htaccess et index.php : ces fichiers seront notre routeur
 - app : ce dossier contiendra le coeur de l'application
 - controllers : contiendra, comme son nom l'indique, les contrôleurs, dont le nom commencera par une majuscule, par convention.
 - models : contiendra nos modèles, leur nom commencera également par une majuscule
 - views : contiendra nos fichiers de vues, dans des dossiers, un dossier par contrôleur.

##### Le point d'entrée : le routeur

Comme indiqué précédemment, le point d'entrée de notre application est le routeur.

Il s'agit "tout simplement" de notre fichier index.php situé à la racine publique de notre projet.

Ce routeur va nous servir à identifier quel contrôleur doit être utilisé pour générer la page demandée.

Dans cette introduction, nous allons faire un routeur simple, qui comprendra les adresses comme ci-dessous.

```http request
http://url_du_site/controleur/methode
```

Cette url permettra à notre routeur de comprendre qu'il doît pointer vers le contrôleur mentionné en premier paramètre et la méthode de ce contrôleur mentionnée en deuxième paramètre.
Le .htaccess

Afin de parvenir à ce fonctionnement, nous devons utiliser la réécriture d'URL proposée par les serveurs Apache au moyen d'un fichier .htaccess.

Nous allons donc créer ce fichier à la racine publique de notre projet, comme index.php.

Dans ce fichier nous allons ajouter ces lignes
```apacheconfig
RewriteEngine On
RewriteRule ^([a-zA-Z0-9\-\_\/]*)$ index.php?p=$1
```

Allons dans les détails

 - RewriteEngine On : permet de démarrer la réécriture d'URL
 - RewriteRule : permet de définir une règle de réécriture d'URL et fonctionne comme suit
 - ^([a-zA-Z0-9\-\_\/]*)$ : il s'agit des différents caractères pris en compte dans l'URL pour sa réécriture
  - a-z : caractères minuscules
  - A-Z : caractères majuscules
  - 0-9 : chiffres
  - \-\_\/ : tiret, underscore et / (caractère \ pour l'échappement)
   Tout ceci entre ^( pour le début de chaîne et )$ pour la fin de chaîne
  - index.php?p=$1 : $1 contiendra le résultat de la réécriture notre chaîne

Que contiendra notre URL finale ?
````http request
http://url_du_site/articles/lire
````
donnera
````http request
http://url_du_site/index.php?p=articles/lire
````

##### Le fichier index.php

Nous allons maintenant devoir gérer les données de l'URL dans le fichier index.php.

Nous allons revenir plusieurs fois sur ce fichier durant ce tutoriel pour y apporter des modifications et ajouts à mesure que l'application sera développée.
Les classes principales

Avant d'écrire notre routeur, nous allons appeler dans ce fichier le contrôleur et le modèle principaux qui serviront de base commune à tous les fichiers.

Afin d'assurer la portabilité du projet sur toutes les configurations, nous allons baser nos appels sur un chemin généré automatiquement. Nous allons le sauvegarder dans une constante que nous appellerons "ROOT".

Cette constante sera générée depuis une des informations stockées dans la super globale "$_SERVER" qui contient le chemin complet vers notre fichier.

Nous y enlèverons uniquement le nom de fichier "index.php" au moyen de la fonction php "str_replace".
````php
define('ROOT', str_replace('index.php','',$_SERVER['SCRIPT_FILENAME']));
````

Nous pouvons maintenant utiliser "ROOT" pour appeler nos fichiers
````php
require_once(ROOT.'app/Model.php');
require_once(ROOT.'app/Controller.php');
````
Les paramètres d'URL

Commençons par la gestion des paramètres d'URL. Dans l'exemple ci-dessus nous avons deux paramètres qui sont envoyés à notre routeur, un contrôleur et une méthode, que nous appellerons action dans la suite du tutoriel.

Il faudra garder en tête qu'une action dans l'URL correspondra à une méthode (fonction) dans un contrôleur.

Pour commencer, nous devons récupérer chacun des paramètres et les affecter à des variables, si ils existent. En effet, la page d'accueil n'aura pas de paramètre par exemple.

Nous allons utiliser la fonction explode de php pour séparer chacun des paramètres et générer un tableau. Puis nous allons tester les valeurs et enfin affecter les variables, si besoin.

Le code ainsi écrit sera le suivant
````php
// On sépare les paramètres et on les met dans le tableau $params
$params = explode('/', $_GET['p']);
// Si au moins 1 paramètre existe
if($params[0] != ""){
    // On sauvegarde le 1er paramètre dans $controller en mettant sa 1ère lettre en majuscule
    $controller = ucfirst($params[0]);
    // On sauvegarde le 2ème paramètre dans $action si il existe, sinon index
    $action = isset($params[1]) ? $params[1] : 'index';
}else{
    // Ici aucun paramètre n'est défini
}
````
Notre routeur est maintenant capable de lire une URL.

Il faut maintenant diriger la demande vers le bon contrôleur et dans ce contrôleur vers la bonne méthode.

Etant donné que nous développons en PHP orienté objet, chaque contrôleur correspondra à une classe.

Si nous prenons l'exemple d'un blog, nous manipulerons des articles. La page d'accueil des articles en affichera la liste complète.

Nous aurons donc une méthode "index" qui accèdera à cette liste.

Si nous souhaitions accéder à cette méthode, sans routeur, nous écririons le code suivant
````php
require_once(ROOT.'controllers/Articles.php');
$articles = new Articles();
$articles->index();
````
Ces 3 lignes permettent d'instancier la classe "Articles" et d'appeler la méthode "index".

Avec notre routeur, nous avons ces informations dans des variables
 - $controller correspond à "Articles"
 - $action correspond à "index"

Nous pourrions donc écrire
````php
require_once(ROOT.'controllers/'.$controller.'.php');
$controller = new $controller();
$controller->$action();
````
Ce qui fait que notre routeur est maintenant comme ceci
````php
// On sépare les paramètres et on les met dans le tableau $params
$params = explode('/', $_GET['p']);
// Si au moins 1 paramètre existe
if($params[0] != ""){
    // On sauvegarde le 1er paramètre dans $controller en mettant sa 1ère lettre en majuscule
    $controller = ucfirst($params[0]);
    // On sauvegarde le 2ème paramètre dans $action si il existe, sinon index
    $action = isset($params[1]) ? $params[1] : 'index';
    // On appelle le contrôleur
    require_once(ROOT.'controllers/'.$controller.'.php');
    // On instancie le contrôleur
    $controller = new $controller();
    // On appelle la méthode
    $controller->$action();    
}else{
    // Ici aucun paramètre n'est défini
}
````

Il nous reste maintenant à gérer les erreurs et l'absence de paramètres.

Une erreur, c'est un contrôleur ou une action qui n'existent pas.

Nous avons une fonction php qui nous permet de vérifier si une méthode existe dans une classe. C'est très pratique pour éviter d'instancier la classe si la méthode demandée n'existe pas. Dans ce cas, nous enverrons une erreur 404.

Cette fonction s'appelle "method_exists" et prend deux paramètres, la classe (contrôleur) et la méthode (action).

Elle s'utilisera donc comme ceci
````php
method_exists($controller, $action)
````
Nous obtiendrons un booléen qui nous permettra de savoir si la méthode existe dans le contrôleur demandé.

Intégrée à notre routeur, elle donne ceci
````php
if(method_exists($controller, $action)){
    // On appelle la méthode
    $controller->$action();    
}else{
    // On envoie le code réponse 404
    http_response_code(404);
    echo "La page recherchée n'existe pas";
}
````

En cas d'absence de paramètres, nous appellerons un contrôleur par défaut que nous appellerons "Main" et sa méthode "index".

Nous ajouterons donc en fin de fichier, dans le "else", les lignes suivantes
````php
// Ici aucun paramètre n'est défini
// On appelle le contrôleur par défaut
require_once(ROOT.'controllers/Main.php');

// On instancie le contrôleur
$controller = new Main();

// On appelle la méthode index
$controller->index();
````
Et voilà, notre routeur est complet à ce stade, voici son code
````php
<?php
// On génère une constante contenant le chemin vers la racine publique du projet
define('ROOT', str_replace('index.php','',$_SERVER['SCRIPT_FILENAME']));
// On appelle le modèle et le contrôleur principaux
require_once(ROOT.'app/Model.php');
require_once(ROOT.'app/Controller.php');
// On sépare les paramètres et on les met dans le tableau $params
$params = explode('/', $_GET['p']);
// Si au moins 1 paramètre existe
if($params[0] != ""){
    // On sauvegarde le 1er paramètre dans $controller en mettant sa 1ère lettre en majuscule
    $controller = ucfirst($params[0]);
    // On sauvegarde le 2ème paramètre dans $action si il existe, sinon index
    $action = isset($params[1]) ? $params[1] : 'index';
    // On appelle le contrôleur
    require_once(ROOT.'controllers/'.$controller.'.php');
    // On instancie le contrôleur
    $controller = new $controller();
    if(method_exists($controller, $action)){
        // On appelle la méthode
        $controller->$action();    
    }else{
        // On envoie le code réponse 404
        http_response_code(404);
        echo "La page recherchée n'existe pas";
    }
}else{
    // Ici aucun paramètre n'est défini
    // On appelle le contrôleur par défaut
    require_once(ROOT.'controllers/Main.php');
    // On instancie le contrôleur
    $controller = new Main();
    // On appelle la méthode index
    $controller->index();
}
````
##### Les contrôleurs

Composant essentiel de notre application, les contrôleurs en sont les véritables tours de contrôle. Ils se situent entre la base de données et les vues et sont chargés de demander et traiter les données avant de les envoyer vers nos vues.
Le contrôleur principal

Le contrôleur principal est le contrôleur qui contiendra les méthodes nécessaires à tous les autres. Ceci nous évitera de répéter les mêmes méthodes plusieurs fois.

Il sera situé dans le dossier "app" et s'appellera "Controller.php".

Nous allons commencer par un contrôleur vide mais nous devrons rapidement y inclure du contenu.
````php
abstract class Controller{

}
````
Vous aurez remarqué le "abstract", qui crée une classe "abstraite" que nous ne pourrons pas instancier directement, mais qui sera utilisée par héritage dans tous nos contrôleurs.
Le contrôleur "Articles"

Passons à notre contrôleur "Articles" qui nous permettra de gérer les pages des articles, comme son nom l'indique.

Ce contrôleur héritera du contrôleur principal et se chargera de traiter les informations et de les passer aux vues.

Pour commencer, nous déclarons l'héritage
````php
class Articles extends Controller{

}
````
Nous allons maintenant générer l'action par défaut qui est la méthode "index". Celle-ci contiendra pour l'instant un "echo".
````php
class Articles extends Controller{
    /**
     * Cette méthode affiche la liste des articles
     *
     * @return void
     */
    public function index(){
        echo "Ici nous aurons la liste des articles";
    }
}
````
A ce stade, ouvrir l'adresse ci-dessous doit vous afficher le contenu du "echo"
````http request
http://url_du_site/articles
````
Nous allons donc créer les contrôleurs en suivant ce modèle, mais nous aurons besoin de données, nous devrons donc charger des fichiers "model" pour pouvoir y accéder. Nous reviendrons un peu plus tard sur nos contrôleurs pour accéder aux données.

##### Les modèles

Les modèles nous permettront d'accéder à la base de données. La première chose à faire est donc d'initialiser une connexion avec cette base. Nous allons le faire dans le modèle principal.
Le modèle principal

Le modèle principal sera une classe abstraite "Model" qui sera incluse par héritage dans tous nos modèles.

Il servira principalement à initialiser la connexion à la base de données.
````php
abstract class Model{
    // Informations de la base de données
    private $host = "localhost";
    private $db_name = "nom_de_la_base";
    private $username = "nom_utilisateur";
    private $password = "mot_de_passe";

    // Propriété qui contiendra l'instance de la connexion
    protected $_connexion;

    // Propriétés permettant de personnaliser les requêtes
    public $table;
    public $id;

    /**
     * Fonction d'initialisation de la base de données
     *
     * @return void
     */
    public function getConnection(){
        // On supprime la connexion précédente
        $this->_connexion = null;

        // On essaie de se connecter à la base
        try{
            $this->_connexion = new PDO("mysql:host=" . $this->host . ";dbname=" . $this->db_name, $this->username, $this->password);
            $this->_connexion->exec("set names utf8");
        }catch(PDOException $exception){
            echo "Erreur de connexion : " . $exception->getMessage();
        }
    }   
}
````
Les méthodes communes

Il peut être utile de créer des méthodes communes à tous les modèles. Par exemple, il sera fréquent de vouloir obtenir un unique enregistrement ou au contraire tous les enregistrements d'une table donnée. Nous avons 2 propriétés publiques "table" et "id" qui nous permettront de créer ces requêtes.

Nous allons les appeler "getOne" pour un enregistrement et "getAll" pour tous les enregistrements.

Voici le code correspondant
````php
/**
 * Méthode permettant d'obtenir un enregistrement de la table choisie en fonction d'un id
 *
 * @return void
 */
public function getOne(){
    $sql = "SELECT * FROM ".$this->table." WHERE id=".$this->id;
    $query = $this->_connexion->prepare($sql);
    $query->execute();
    return $query->fetch();    
}

/**
 * Méthode permettant d'obtenir tous les enregistrements de la table choisie
 *
 * @return void
 */
public function getAll(){
    $sql = "SELECT * FROM ".$this->table;
    $query = $this->_connexion->prepare($sql);
    $query->execute();
    return $query->fetchAll();    
}
````

Ces méthodes seront donc disponibles depuis tous les modèles.

Le modèle "Article"

Une classe "Articles" existant déjà pour un contrôleur, nous allons nommer notre modèle "Article.php" et le stocker dans "models".

Cette classe "Article" va hériter du modèle principal.
````php
class Article extends Model{

}
````
Nous allons devoir utiliser le constructeur pour instancier la base de données dès que le modèle sera lui même instancié.

Pour ce faire nous allons ajouter la méthode magique "__construct"
````php
public function __construct()
{
    // Nous définissons la table par défaut de ce modèle
    $this->table = "articles";

    // Nous ouvrons la connexion à la base de données
    $this->getConnection();
}
````
Voici donc notre modèle complet
````php
class Article extends Model{

    public function __construct()
    {
        // Nous définissons la table par défaut de ce modèle
        $this->table = "articles";
    
        // Nous ouvrons la connexion à la base de données
        $this->getConnection();
    }
}
````
Faire "discuter" les contrôleurs et les modèles

Notre contrôleur "Articles" nécessite l'affichage des derniers articles lors de l'appel de la méthode "index".

Nous allons donc devoir demander à notre contrôleur d'utiliser le modèle "Article" et de récupérer tous les enregistrements.

Par extension, il sera courant de devoir instancier un modèle depuis un contrôleur. Nous allons donc commencer par ajouter une méthode dans notre contrôleur principal qui nous permettra de charger n'importe quel modèle à tout moment.

Cette méthode sera appelée "loadModel" et contiendra ce code
````php
/**
 * Permet de charger un modèle
 *
 * @param string $model
 * @return void
 */
public function loadModel(string $model){
    // On va chercher le fichier correspondant au modèle souhaité
    require_once(ROOT.'models/'.$model.'.php');
    
    // On crée une instance de ce modèle. Ainsi "Article" sera accessible par $this->Article
    $this->$model = new $model();
}
```
Dans tous les contrôleurs il nous suffira d'appeler un modèle de cette façon
````php
$this->loadModel('Article');
````
Notre contrôleur "Articles" va maintenant devoir demander à notre modèle la liste de tous les articles.

Nous allons donc instancier le modèle "Article" puis utiliser la méthode "getAll" pour obtenir cette liste. Nous allons juste, pour l'instant, faire un "var_dump" de cette liste pour vérifier que ça fonctionne.

Le code sera le suivant
````php
/**
 * Cette méthode affiche la liste des articles
 *
 * @return void
 */
public function index(){
    // On instancie le modèle "Article"
    $this->loadModel('Article');

    // On stocke la liste des articles dans $articles
    $articles = $this->Article->getAll();

    // On affiche les données
    var_dump($articles);
}
````
La liste des articles est effectivement affichée de façon brute.
Les vues

L'intérêt de toute application web est d'afficher les données de façon propre et structurée aux utilisateurs. Nous allons donc devoir générer du code html qui contiendra les données de la base de données.

Pour ce faire, nous allons créer des vues. Ces fichiers contiendront principalement du HTML mais également un peu de PHP pour afficher les données.

Pour commencer, nous allons créer un dossier "articles" dans "views" et un premier fichier "index.php" à l'intérieur. Nous utiliserons comme "convention" de nommer les fichiers de vue de la même façon que la méthode qui les appellera.

Ce fichier index.php devra donc être appelé par le contrôleur.
Préparer le contrôleur principal

Pour simplifier les choses, nous allons créer une méthode d'affichage des vues dans le contrôleur principal. Cette méthode permettra de charger toutes les vues, quel que soit le contrôleur qui va y faire appel.

Pour y parvenir nous utiliserons la fonction php "get_class" qui permet de récupérer la classe du contrôleur. Nous la passerons en minuscule, convention utilisée pour les noms de dossiers, et l'intègrerons au chemin d'accès au fichier de vues.

La méthode d'affichage s'appelle assez souvent "render", nous utiliserons donc ce nom. Elle devra également récupérer en entrée le nom de la vue, et les données, si il y en a.

Cette méthode s'écrira donc comme ceci
````php
/**
 * Afficher une vue
 *
 * @param string $fichier
 * @param array $data
 * @return void
 */
public function render(string $fichier, array $data = []){
    // Récupère les données et les extrait sous forme de variables
    extract($data);

    // Crée le chemin et inclut le fichier de vue
    require_once(ROOT.'views/'.strtolower(get_class($this)).'/'.$fichier.'.php');
}
````
La fonction php "extract" permet de prendre un tableau et de l'éclater en différentes variables.

Ainsi, avec ce tableau
````php
$data = [
    'id' => 1,
    'contenu' => 'Ceci est le contenu'
];
````
Après avoir fait un "extract" nous aurons
````php
$id = 1;
$contenu = 'Ceci est le contenu';
````
Nous pourrons donc utiliser nos variables dans notre vue.
Appeler la vue depuis le contrôleur

Revenons à notre contrôleur "Articles" et sa méthode "index".

Cette méthode devra appeler la vue "index" et lui passer les articles sous forme de tableau.

Nous allons donc appeler la méthode "render" et lui faire passer les informations.

Les données devront être passées sous forme de tableau. Nous pourrons utiliser la fonction php "compact" pour créer le tableau pour nous.

Sans la fonction compact, nous écrirons
````php
$this->render('index', ['articles' => $articles]);
````
Avec la fonction compact nous pourrons écrire
````php
$this->render('index', compact('articles'));
````
A vous de choisir. Notre méthode "index" finalisée ressemble donc à ceci
````php
/**
 * Cette méthode affiche la liste des articles
 *
 * @return void
 */
public function index(){
    // On instancie le modèle "Article"
    $this->loadModel('Article');
    // On stocke la liste des articles dans $articles
    $articles = $this->Article->getAll();
    // On envoie les données à la vue index
    $this->render('index', compact('articles'));
}
````
Le fichier de vue

Passons maintenant à l'affichage de nos données. Nous devons créer du code HTML et y insérer les données envoyées par le contrôleur.

Disons que nos articles contiennent un titre, un texte et un "slug", version simplifiée du titre.

Pour afficher les données, nous allons devoir boucler sur la variable "articles" et afficher les informations à chaque boucle.

Nous allons donc procéder comme ceci
````php
<?php foreach($articles as $article): ?>
  <h2><?= $article['titre'] ?></h2>
  <p><?= $article['contenu'] ?></p>
<?php endforeach ?>
````
Vous noterez plusieurs choses
 - Nous utilisons ici la syntaxe alternative du "foreach" pour des soucis de lisibilité, il est plus facile de retrouver le "endforeach" qu'une accolade
 - Nous utilisons la balise courte echo (<?=) pour les affichages, c'est plus simple également

A ce stade, la liste des articles présents en base de données est affichée. Bien sûr, nous devrons faire plus de présentation mais ce n'est pas l'objet de ce tutoriel.

Cependant, il nous manque une chose, le lien pour aller lire l'article. Nous allons donc devoir créer une nouvelle "route" pour y accéder, et cette route devra prendre un paramètre complémentaire.

Pour la lecture d'un article, nous définirons la route comme suit
````http request
http://url_du_site/articles/lire/slug-de-l-article
````
Notre vue finalisée sera donc la suivante
````php
<?php foreach($articles as $article): ?>
  <h2><a href="/articles/lire/<?= $article['slug'] ?>"><?= $article['titre'] ?></a></h2>
  <p><?= $article['contenu'] ?></p>
<?php endforeach ?>
````
##### Les routes avec paramètres

Pour accéder à la lecture d'un article, nous avons besoin de passer un paramètre supplémentaire à notre routeur, le "slug" de l'article.

##### Le routeur

Le problème est que notre routeur actuel ne prend que 2 paramètres. Comment faire pour en ajouter un 3ème ?

Nous allons utiliser la fonction php "call_user_func_array" qui permet d'appeler une fonction en lui faisant passer des paramètres sous forme de tableau.

Depuis PHP 5.3 nous pouvons passer à cette fonction une classe et une méthode sous forme de tableau en 1er paramètre également.

Ceci nous donnerait donc, pour notre routeur
````php
call_user_func_array([$controller,$action], $params);
// la ligne ci-dessus remplacera la ligne ci-dessous

$controller->$action();  
````
Le problème qui se pose maintenant est que les paramètres sont en doublon, pour les 2 premiers. Il faudra donc commencer par les supprimer.

Notre routeur ainsi finalisé sera celui-ci (les lignes 27 à 32 ont été ajoutées)
````php
<?php
// On génère une constante contenant le chemin vers la racine publique du projet
define('ROOT', str_replace('index.php','',$_SERVER['SCRIPT_FILENAME']));

// On appelle le modèle et le contrôleur principaux
require_once(ROOT.'app/Model.php');
require_once(ROOT.'app/Controller.php');

// On sépare les paramètres et on les met dans le tableau $params
$params = explode('/', $_GET['p']);

// Si au moins 1 paramètre existe
if($params[0] != ""){
    // On sauvegarde le 1er paramètre dans $controller en mettant sa 1ère lettre en majuscule
    $controller = ucfirst($params[0]);

    // On sauvegarde le 2ème paramètre dans $action si il existe, sinon index
    $action = isset($params[1]) ? $params[1] : 'index';

    // On appelle le contrôleur
    require_once(ROOT.'controllers/'.$controller.'.php');

    // On instancie le contrôleur
    $controller = new $controller();

    if(method_exists($controller, $action)){
        // On supprime les 2 premiers paramètres
        unset($params[0]);
        unset($params[1]);

        // On appelle la méthode $action du contrôleur $controller
        call_user_func_array([$controller,$action], $params);
    }else{
        // On envoie le code réponse 404
        http_response_code(404);
        echo "La page recherchée n'existe pas";
    }
}else{
    // Ici aucun paramètre n'est défini
    // On appelle le contrôleur par défaut
    require_once(ROOT.'controllers/Main.php');

    // On instancie le contrôleur
    $controller = new Main();

    // On appelle la méthode index
    $controller->index();
}
````
Le modèle

Nous pouvons donc maintenant créer la méthode "lire" avec un paramètre "slug" pour aller chercher un article.

Mais nous n'avons pas encore de méthode pour aller chercher un article à partir de son slug, ce qui nous pose problème.

Nous allons donc créer cette méthode dans le modèle "Article". Cette méthode prendra en entrée un "slug" et retournera un enregistrement de la base de données.

Pour ce tutoriel nous n'allons pas ajouter de contrôles mais il va de soi que nous devons contrôler les données pour éviter
 - Les injections SQL
 - Les failles XSS
 - Le chargement d'un article qui n'existe pas

Le modèle "Article" sera donc mis à jour comme ceci
````php
<?php
class Article extends Model{

    public function __construct()
    {
        // Nous définissons la table par défaut de ce modèle
        $this->table = "articles";
    
        // Nous ouvrons la connexion à la base de données
        $this->getConnection();
    }

    /**
     * Retourne un article en fonction de son slug
     *
     * @param string $slug
     * @return void
     */
    public function findBySlug(string $slug){
        $sql = "SELECT * FROM ".$this->table." WHERE `slug`='".$slug."'";
        $query = $this->_connexion->prepare($sql);
        $query->execute();
        return $query->fetch(PDO::FETCH_ASSOC);    
    }

}
````
##### Le contrôleur

Notre modèle étant à jour, nous pouvons maintenant créer notre méthode "lire" dans notre contrôleur "Articles".

Cette méthode s'écrira comme suit
````php
/**
 * Méthode permettant d'afficher un article à partir de son slug
 *
 * @param string $slug
 * @return void
 */
public function lire(string $slug){
    // On instancie le modèle "Article"
    $this->loadModel('Article');

    // On stocke l'article dans $article
    $article = $this->Article->findBySlug($slug);

    // On envoie les données à la vue lire
    $this->render('lire', compact('article'));
}
````
Notre contrôleur "Articles" finalisé est le suivant
````php
<?php

class Articles extends Controller{
    /**
     * Cette méthode affiche la liste des articles
     *
     * @return void
     */
    public function index(){
        // On instancie le modèle "Article"
        $this->loadModel('Article');

        // On stocke la liste des articles dans $articles
        $articles = $this->Article->getAll();

        // On envoie les données à la vue index
        $this->render('index', compact('articles'));
    }

    /**
     * Méthode permettant d'afficher un article à partir de son slug
     *
     * @param string $slug
     * @return void
     */
    public function lire(string $slug){
        // On instancie le modèle "Article"
        $this->loadModel('Article');

        // On stocke l'article dans $article
        $article = $this->Article->findBySlug($slug);

        // On envoie les données à la vue lire
        $this->render('lire', compact('article'));
    }
}
````
###### La vue

Vous remarquez que notre vue s'appelle "lire" et qu'elle utilisera la variable "article".

Le fichier "lire.php" sera créé dans le dossier "views/articles" et contiendra
````html
<h2><?= $article['titre'] ?></h2>

<p><?= $article['contenu'] ?></p>
````

C'est terminé.
Utiliser un "template" de vues

Il est très courant d'avoir des éléments qui s'intègrent dans toutes les pages (entête, pied de page...), il serait donc utile de pouvoir les intégrer automatiquement dans toutes nos vues.

Sur le principe, notre "template" contiendra tous les éléments répétitifs des pages, et nos vues viendront uniquement injecter leur contenu dans l'emplacement réservé à cet effet, comme indiqué sur le schéma ci-dessous

Schéma template

Nous allons donc préparer ce "template" et le stocker dans un dossier appelé communément "layout" dans le dossier "views".

Ce fichier contiendra tout ce que nous voulons intégrer dans toutees nos pages, et un espace réservé sous la forme d'une variable php (ici $content) pour le contenu envoyé par les vues.

Ce fichier s'appellera "default.php" et contiendra
````html
<header>
    <h1>Bienvenue sur mon blog</h1>
</header>
<main>
    <?= $content ?>
</main>
<footer>
    <p>Copyright 2019</p>
</footer>
```

Nous devons donc maintenant gérer ce "template" dans la fonction "render" du contrôleur principal.

C'est ici qu'entre en jeu la "temporisation de sortie", c'est à dire, la mise en cache du code généré le temps qu'on décide ce qu'on veut en faire.

Nous allons demander à la vue de générer son contenu, puis le stocker dans une variable et enfin l'envoyer au "template"

Pour ce faire nous utiliserons les fonctions php "ob_start" et "ob_get_clean".

Ici, ob signifie "Output Buffer".

"ob_start" crée ce "buffer" et stocke tout le code de sortie généré. "ob_get_clean" va vider ce buffer dans la variable de notre choix.

Le contrôleur terminé ressemblera à ceci (notez les modifications sur les lignes 13 à 23)
````php
<?php
abstract class Controller{
    /**
     * Afficher une vue
     *
     * @param string $fichier
     * @param array $data
     * @return void
     */
    public function render(string $fichier, array $data = []){
        extract($data);

        // On démarre le buffer de sortie
        ob_start();

        // On génère la vue
        require_once(ROOT.'views/'.strtolower(get_class($this)).'/'.$fichier.'.php');

        // On stocke le contenu dans $content
        $content = ob_get_clean();

        // On fabrique le "template"
        require_once(ROOT.'views/layout/default.php');
    }

    /**
     * Permet de charger un modèle
     *
     * @param string $model
     * @return void
     */
    public function loadModel(string $model){
        // On va chercher le fichier correspondant au modèle souhaité
        require_once(ROOT.'models/'.$model.'.php');
        
        // On crée une instance de ce modèle. Ainsi "Article" sera accessible par $this->Article
        $this->$model = new $model();
    }
}
