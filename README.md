# symfony-base-fixture

**todo** Build, downloads ...

## Langage

Français

## Description

`symfony-base-fixture` est une librairie permettant de créer des [Fixtures](https://symfony.com/doc/current/bundles/DoctrineFixturesBundle/index.html) plus rapidement avec quelques fonctionnalités supplémentaires

Cette librairie embarque [faker](https://github.com/fzaninotto/Faker) 

## Documentation

### Requirements

Pour installer ce package, il est nécessaire de l'intégrer à un projet symfony ayant le package orm-fixture

- `php` `>=7.3.0`
- `doctrine/doctrine-fixtures-bundle` `^3.3`
- `symfony/framework-bundle` `5.0.*`
    
### Installation

Commande d'installation de la librairie :

`composer req mansartesteban/symfony-base-fixture`

### Méthodes

`public function BaseFixture::createMany(String $groupName, callable $factory, $count);`

Créé un nombre d'**entity** (défini par `$count`) selon un processus défini dans $factory

**Paramètres**

`$groupeName` Chaîne de caractères définissant un nom arbitraire d'un groupe de données (Ex: "User", "Posts")

`$factory` Function de callback appelée à l'intérieur de `BaseFixture::createMany`, la variable *$i* peut être utilisée dans cette function comme incrémenteur 
Cette function doit retourner un objet de type `Entity` (Voir exemple plus bas)

`$count` Le nombre d'*Entity* à créer

**Retour**

Aucun

**Exception**

Jette une `LogicException` si `$factory` ne retourne pas d'objet
____

`public function BaseFixture::getRandomReference(string $groupName);`

Récupére un objet de type `Entity` aléatoire d'un groupe

**Paramètres**

`$groupeName` Le nom du groupe comme défini dans `BaseFixture::$createMany`

**Retour**

Retourne un objet de type `Entity` si trouvé

**Exception**

Jette une `InvalidArgumentException` dans le cas où aucune `Entity` est trouvée

____

`public function BaseFixture::getRandomReferences(string $groupName, int $count);`

Récupère plusieurs objets de type `Entity` aléatoirement d'un groupe

**Paramètres**

`$groupName` Le nom du groupe comme défini dans `BaseFixture::$createMany`

`$count` Le nombre d'objets à retourner

**Retour**

Retourne un tableau d'objets de type `Entity` si trouvés

**Exception**

Jette une `InvalidArgumentException` dans le cas où aucune `Entity` est trouvée


### Utilisation et exemples

`INFO` La class BaseFixture permet d'accéder à Fzaninotto/Faker depuis `$this->faker`

Dans le dossier src, si le dossier DataFixtures n'existe pas, le créer

Dans ce dossier créer un fichier de fixture, nous allons prendre comme exemple l'**entity** `User`

Créer le fichier `UserFixture` et y insérer le code suivant

```php

namespace App\DataFixtures;

use App\Entity\User;
use mansartesteban\symfony\BaseFixture;
use Doctrine\Persistence\ObjectManager;

class UserFixture extends BaseFixture {

    public function loadData(ObjectManager $manager) { // Implémenter cette function
    
        $this->createMany("User", function ($index) {
            $user = new User();
            $user->setFirstname($this->faker->firstname);
            
            return $user;
        }, 250);
    
    }

}

```

Et voilà, nous avons créé notre première fixture simplement !

Créons un deuxième fichier de fixtures pour générer les posts des internautes par exemple

`
    namespace App\DataFixtures;
    
    use App\Entity\Post;
    use mansartesteban\symfony\BaseFixture;
    use Doctrine\Common\DataFixtures\DependentFixtureInterface;
    use Doctrine\Persistence\ObjectManager;
    
    class PostFixture extends BaseFixture implements DependentFixtureInterface {
    
        public function loadData(ObjectManager $manager) { // Implémenter cette function
        
            $this->createMany("Post", function ($index) {
                $post = new Post();
                
                $post
                    ->setMessage("Lorem ipsum ...")
                    ->setUser($this->getRandomReference("User");
                
                return $post;
            }, 5000);
        
        }
        
        public function getDependencies() {
        
            return [
                UserFixture::class
            ]; // Retourne le tableau des classes dont notre Fixture dépend
        
        }
    
    }
`