### Creation de projet Symfony ###
- Pour créer un nouveau projet Symfony, utilisé la commande : ```symfony new --webApp {Nom de votre projet}```
- Créer un fichier .env.local afin de configurer la base de données : ```DATABASE_URL="mysql://{username:password}@127.0.0.1:3306/{database_name}?serverVersion=8&charset=utf8mb4"```
- Créer la base de données configurée précédemment avec : ```symfony console doctrine:database:create```
- Lancer votre serveur avec : ```symfony serve```

### Creation d'entitées avec Symfony ###
- Pour créer ou modifier une entitée : ```symfony console make:entity```
  - Il ne vous reste plus qu'a suivre les instructions
  - Lors de la demande du type, n'hésitez pas à utiliser ```?``` pour voir tous les types disponibles.

### Executer des migrations ###
- Quand vous voulez aappliquer les modification de vos entitées, vous devez créer une migration et ensuite l'executer, donc dans l'ordre : 
- ```symfony console make:migration```
- ```symfony console doctrine:migration:migrate```

### Pour faire des fixtures avec Symfony ###
- Installer la librairie alice bundle avec : ```composer require --dev hautelook/alice-bundle```
  - Un dossier fictures à du se créer dans votre application
- Créer bien un fichier par entitée. Vous pouvez avoir des exemples sur la doc ici (dans basic usage) : https://github.com/theofidry/AliceBundle#installation
- Pour executer la génération de vos fixtures : ```symfony console hautelook:fixtures:load --purge-with-truncate```

### Creation de Controller ###
-  Afin de créer des controller, utiliser la commande : ```symfony console make:controller```
  - Cette commande va vous générer un Controller et un template associé.
    - Vous pouvez ajouter autant de route et créer autant de template que vous voulez dans un controller.
- Chaque fonction de vos controller doit avoir une Route configurée, comme ceci : 
```#[Route('/movie', name: 'app_movie')]```
```public function index(): Response```
- Dans notre cas /movie représente l'url de votre route, app_movie représente le NOM DE LA ROUTE
  - Dans une application symfony le nom de la route est utilisé pour les redirections, les liens etc.
- Pour plus d'informations sur les controller et les routes, rendez vous sur la doc de Symfony : https://symfony.com/doc/current/controller.html

### Faire des redirections avec Symfony ###
- Afin de faire des redirections directement dans un controller, vous pouvez utiliser la methode RedirectToRoute, exemple :
- ``` return $this->redirectToRoute(MA_SUPER_ROUTE)```
- Si votre route a besoin de paramètres, ajouter un tableau, comme ceci :
- ```return $this->redirectToRoute(MA_SUPER_ROUTE, ['id' => 5])```

### Faire des lien avec Twig ###
- Dans votre href, vous devez utiliser la fonction path, exemple : 
- ```<a href="{{ path('MA_SUPER_ROUTE') }}">Mon super lien</a>```
- Si votre route a besoin de paramètres, ajouter un objet, comme ceci :
- ```<a href="{{ path('MA_SUPER_ROUTE', {'id' : 5}) }}">Mon super lien</a>```

### Récupérer des entitées de notre base de données (Repository) ###
- Afin de faire des SELECT avec symfony, nous devons utiliser le repository de l'entitées associé
- A chaque nouvelle entitée créer, un repository associé est créé aussi.
- Dans votre Controller, importé le repository de cette façon : 
- ```
  public function __construct(
        private CreatorRepository      $creatorRepository,
    )
    {
    }```
- N'oubliez pas le use ! : ```use App\Repository\CreatorRepository;```
- Maintenant que votre Repository est importé, vous pouvez l'utiliser partout dans votre Controller
  - Un repository vient avec 4 méthodes de base : findAll(), find(), findOneBy(), findBy()
    - Rendez vous dans votre repository pour voir ce que vous rends vos méthodes et comment les utiliser !
- Pour plus d'information sur les repository, rendez vous ici : https://symfony.com/doc/current/doctrine.html#fetching-objects-from-the-database
- exemple simple d'utilisation : ```$mySuperEntity = $this->mySuperEntityRepository->find(1)```
### Faire des INSERT et UPDATE en base de données ###
- Avec Symfony, vous pouvez faire des INSERT et des UPDATES que d'une entitée.
- Pour ce faire vous devez importer EntityManagerInterface dans votre Controller (ou autre), ex :
- ```
  public function __construct(
        private EntityManagerInterface      $entityManager,
    )
    {
    }```
- N'oubliez pas le use ! : ```use Doctrine\ORM\EntityManagerInterface;```
- Exemple d'utilisation : 
- ```
  //nouvelle entitée
  $creator = new Creator();
  
  //lui ajouter ce que l'on a besoin
  $creator->setTruc('allo')
  
  //L'ajouter à la file d'attente grace à l'entity manager
  $this->entityManager->persist($creator)
  
  //Puis executer la file d'attente
  $this->entityManager->flush()
  ```
  
- Si vous avez besoin de supprimer quelque chose de la base donnée : 
  - Récupérer l'entité que vous voulez supprimer grace à son repository
  - Puis utiliser l'entityManager : $entityManager->remove()
  - N'oubliez pas le flush à la fin : $entityManager->flush()