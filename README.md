# wp-autoload
Autoloader simple pour WordPress

## Introduction
Il s'agit d'un script php qui fait en sorte que vos classes PHP se chargent automatiquement lorsqu'elles sont lancées. 
Ainsi, vous n'avez pas besoin de les requérir manuellement.

L'utilisation du script autoloader est assez simple.
* Collez ce fichier à la racine de votre plugin ou thème WordPress. 
* Nommer les classes PHP en fonction de la structure de leur dossier. 
* Les classes sont maintenant automatiquement requises lorsqu'elles sont initiées.
* Utilisez une classe par fichier !

## Utiliser WP Autoload
Supposons que vous construisiez un nouveau thème WordPress. Dans ce thème, vous utilisez un dossier spécifique nommé classes pour stocker toutes vos classes. Pour cet exemple, le thème a deux classes qui sont placées dans les dossiers suivants et un functions.php à la racine.
- functions.php
- classes\theme.php
- classes\models\model.php

Dans le fichier functions.php, nous incluons l'autoloader et appelons nos classes :

```php
spl_autoload_register( function($classname) {
    
    $class      = str_replace( '\\', DIRECTORY_SEPARATOR, str_replace( '_', '-', strtolower($classname) ) );
    $classes    = dirname(__FILE__) .  DIRECTORY_SEPARATOR . 'classes' . DIRECTORY_SEPARATOR . $class . '.php'; 
    
    if( file_exists($classes) ) {
        require_once( $classes );
    }    
} );

$theme = new Theme();
$model = new Models\Model.php

```
Comme nous avons indiqué le dossier "classes" dans notre autochargeur, il demandera automatiquement les classes à partir du dossier classes et nous n'aurons pas à les demander manuellement.

La classe du thème n'a pas d'espace de noms, car elle se trouve à la racine de notre dossier classes et l'autochargeur commence à la charger. Cela peut ressembler à ceci (pour commencer) :

```php
<?php
class Theme {
  public function __construct() {
  }
}
```

La classe de modèle a un espace de noms, car elle se trouve dans un sous-dossier. Le fichier model.php peut ressembler à ceci :
```php
<?php
namespace Models;
use WP_Query as WP_Query;

class Model {
  public function __construct() {
  }
}
```
Veuillez également noter que si nous voulons utiliser des classes de WordPress ou d'autres applications, nous devons les indiquer en haut du document en utilisant la clause ``use`` comme dans l'exemple de model.php.

## Utiliser WP Autoload avec Composer
Nous pouvons ajouter des composants à notre thème ou à notre plugin WordPress en utilisant Composer. En d'autres termes, Composer est une sorte de gestionnaire de paquets que vous pouvez utiliser pour enregistrer et ajouter de nouveaux modules à votre code.

Par exemple, nous pouvons vouloir inclure le code d'un dépôt public dans l'un des thèmes que nous développons. Par exemple, le dépôt [que nous construisons pour ajouter des champs personnalisés] (https://github.com/ahmedguesmi/customwpfields). 

Nous pouvons le faire en plaçant un fichier composer.json à la racine de notre thème. Le fichier peut ressembler à ceci :
```
{
    "name": "ahmedguesmi/waterfall",
    "description": "Waterfall is a theme with extensive customizer options and very easy to modify or extend for developers.",
    "repositories": [              
        {
            "type": "vcs",
            "url": "https://github.com/ahmedguesmi/customwpfields.git"
        }       
    ],
    "require": {
        "php": ">=7.0",
        "ahmedguesmi/customwpfields": "dev-master",
    },
    "authors": [ 
        {
            "name": "Guesmi Ahmed",
            "email": "hello@ahmedguesmi.com",
            "homepage": " "

        } 
    ]          
}
```

Si nous plaçons notre terminal dans le dossier racine du thème que nous développons, nous pouvons utiliser ``composer update --prefer-dist`` pour inclure ce dépôt. L'option --prefer-dist ignorera tous les fichiers liés à GIT pour les dépôts donnés et préférera une version de distribution. A moins que vous ne vouliez contribuer au dépôt, c'est la meilleure façon d'inclure des modules supplémentaires. Par défaut, composer place tous les fichiers dans un dossier vendor.

Maintenant, dans notre fichier functions.php, nous pouvons ajouter l'autloader suivant :

```php
spl_autoload_register( function($classname) {
    
    $class      = str_replace( '\\', DIRECTORY_SEPARATOR, str_replace( '_', '-', strtolower($classname) ) );
    $classes    = dirname(__FILE__) .  DIRECTORY_SEPARATOR . 'classes' . DIRECTORY_SEPARATOR . $class . '.php';
    
    $vendor     = str_replace( 'ahmedguesmi' . DIRECTORY_SEPARATOR, '', $class );
    $vendor     = 'ahmedguesmi' . DIRECTORY_SEPARATOR . preg_replace( '/\//', '/src/', $vendor, 1 ); // Replace the first slash for the src folder
    $vendors    = dirname(__FILE__) .  DIRECTORY_SEPARATOR . 'vendor' . DIRECTORY_SEPARATOR . $vendor . '.php';

    if( file_exists($classes) ) {
        require_once( $classes );
    } elseif( file_exists($vendors) ) {
        require_once( $vendors );    
    }
   
} );
```

Cela permettra de charger automatiquement toutes les classes appelées depuis le dossier classes, si elles sont correctement nommées, ainsi que tous les modules ajoutés par composer. Comme nous utilisons des composants de ahmedguesmi, ceux-ci sont placés dans le dossier vendor/ahmedguesmi.

Veuillez noter que composer ajoute également ses propres fonctions de chargement automatique dans les dossiers respectifs des fournisseurs. Vous pouvez également avoir besoin de ces fonctions et les utiliser pour charger automatiquement les composants.
