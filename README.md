### Creation de projet Symfony ###

- Pour créer un nouveau projet Symfony, utilisé la commande : ```symfony new --webApp {Nom de votre projet}```
- Créer un fichier .env.local afin de configurer la base de
  données : ```DATABASE_URL="mysql://{username:password}@127.0.0.1:3306/{database_name}?serverVersion=8&charset=utf8mb4"```
- Créer la base de données configurée précédemment avec : ```symfony console doctrine:database:create```
- Lancer votre serveur avec : ```symfony serve```

### Creation d'entitées avec Symfony ###

- Pour créer ou modifier une entitée : ```symfony console make:entity```
    - Il ne vous reste plus qu'a suivre les instructions
    - Lors de la demande du type, n'hésitez pas à utiliser ```?``` pour voir tous les types disponibles.

### Executer des migrations ###

- Quand vous voulez aappliquer les modification de vos entitées, vous devez créer une migration et ensuite l'executer,
  donc dans l'ordre :
- ```symfony console make:migration```
- ```symfony console doctrine:migration:migrate```

### Pour faire des fixtures avec Symfony ###

- Installer la librairie alice bundle avec : ```composer require --dev hautelook/alice-bundle```
    - Un dossier fictures à du se créer dans votre application
- Créer bien un fichier par entitée. Vous pouvez avoir des exemples sur la doc ici (dans basic
  usage) : https://github.com/theofidry/AliceBundle#installation
- Pour executer la génération de vos fixtures : ```symfony console hautelook:fixtures:load --purge-with-truncate```

### Creation de Controller ###

- Afin de créer des controller, utiliser la commande : ```symfony console make:controller```
- Cette commande va vous générer un Controller et un template associé.
    - Vous pouvez ajouter autant de route et créer autant de template que vous voulez dans un controller.
- Chaque fonction de vos controller doit avoir une Route configurée, comme ceci :
  ```#[Route('/movie', name: 'app_movie')]```
  ```public function index(): Response```
- Dans notre cas /movie représente l'url de votre route, app_movie représente le NOM DE LA ROUTE
    - Dans une application symfony le nom de la route est utilisé pour les redirections, les liens etc.
- Pour plus d'informations sur les controller et les routes, rendez vous sur la doc de
  Symfony : https://symfony.com/doc/current/controller.html

### Faire des redirections avec Symfony ###

- Afin de faire des redirections directement dans un controller, vous pouvez utiliser la methode RedirectToRoute,
  exemple :
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
- Pour plus d'information sur les repository, rendez vous
  ici : https://symfony.com/doc/current/doctrine.html#fetching-objects-from-the-database
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

### Les formulaires avec Symfony ###

- Un formulaire peux être basé sur une entité ou non, il sera basé sur une entité si vous avez besoin de CREER ou de
  MODIFIER une entité.
- commande qui permet de générer un formulaire : ```symfony console make:form```
    - La commande va vous demander ensuite le nom de votre formulaire (par convention ils doivent finir par Type), ex :
      MovieType
    - Ensuite il vous demande sur quelle entitée il doit etre basé (laissé vide si il n'est pas relié à une entité).
- Votre formulaire est maintenant créer vous pouvez le retrouver dans src/Form
- Dans votre formulaire vous pouvez gérer le types de champs de vos paramètres, rendez vous sur cette
  doc : https://symfony.com/doc/current/reference/forms/types.html
- Voici un exemple formulaire simple :
    - ```
    $builder
              // BIEN CELUI CI : use Symfony\Component\Form\Extension\Core\Type\TextType;
              ->add('label', TextType::class, [
                  'label' => "Je suis un nouveau label",
                  'attr' => [
                      'placeholder' => 'Truc',
                      'class' => 'truc'
                  ]
              ]);```
- Pour traité le formulaire dans votre controller suivez l'exemple suivant :
- ```
    //creation d'un nouveau genre
        $genre = new Genre();


        //use App\Form\GenreType;
        $form = $this->createForm(GenreType::class, $genre);

        //dire au formulaire d'écouter la request
        // (récupérer les paramètres POST GET etc)
        $form->handleRequest($request);

        //gerer la soumission et validation de notre formulaire
        if ($form->isSubmitted() && $form->isValid()) {


            $this->entityManager->persist($genre);
            $this->entityManager->flush();


            $this->redirectToRoute('app_genre_add');
        }
  ```
- pensez bien à envoyer votre formulaire à votre vue, puis l'affiché, comme ceci :
- ```
  //App/Controller/MyController.php
     return $this->render('genre/index.html.twig', [
            //creer la vue du formulaire et le donner à twig
            'form' => $form->createView()
        ]);
  ```
- Dans votre vue :
    - ```
    //templates/genre/index.html.twig
    {% block body %}
      {#    commencer l'affichage du formulaire #}
      {{ form_start(form) }}


      {#    Ajout d'un bouton pour pouvoir submit le form #}
      <button type="submit">Valider</button>


      {#    finir l'affichage du formulaire #}
      {{ form_end(form) }}
    {% endblock %}
    ```

### Gestion des entitées avec un formulaire Symfony ###

- Afin de gérer les relation avec vos entitées dans votre formulaire, vous pouvez suivre cet exemple :
    - Tout va se passer au sein de votre Type.
- ```
  //App\Form\MovieType
              ->add('genre', EntityType::class, [
                'class' => Genre::class,
                'choice_label' => 'label',
            ])
  ```
    - Dans l'exemple, genre est une entitée, vous devez utiliser EntityType pour dire à Symfony que vous allez relier
      une entitée, puis dans les options :
        - la class : qui représente les entitées qui vont etre affiché
        - le choice_label : qui représente quelle propriété de votre entitée doit être affiché
- Dans le cas d'une collections, ajouter l'option multiple à true pour gérer des tableaux.

### Gestion de la connexion utilisateur ###

- La connexion/ déconnexion/enregistrement d'utilisateur est automatique dans Symfony, il vous faire ces 3 commandes :
- ```symfony console make:user```
- ```symfony console make:auth (!!!`Lors de l'utilisation de make:auth, mettez "no" quand il vous demande l'envoi de mail)```
- ```symfony console make:registration-form```

- Ces 3 commandes vont vous générer :
    - Toute la logique permettant la connexion l'enregistrement et la deconnexion
    - Un formulaire d'enregistrement et un formulaire de connexion
    - Ainsi que 3 routes :
        - app_login : qui vous permet d'arriver à votre formulaire de connexion
        - app_logout : qui vous permet de vous déconnecter
        - app_register : qui vous permet d'enregistrer un utilisateur

### Utiliser le user connecté ou déconnecté avec twig et controller ###

- En twig vous pouvez récupérer l'utilisateur connecté de cette manière :
- ```app.user```
- Un petit exemple d'utilisation :
- ```
  {% if app.user %}
    <p>je suis connecté en tant que {{ app.user.email }}</p>
  {% else %}
    <p>je ne suis pas connecté</p>
  {% endif %}
  ```
- Dans un controller, vous pouvez récupérer l'utilisateur connecté de cette manière :
- ``` $user = $this->getUser() ```
- Comme en twig si l'utilisateur est connecté ça va vous rendre votre Objet User sinon ça vous rend null
- Exemple concret dans un controller :
- ```
  $user = $this->getUser();
  if($user === null){
    return $this->redirectToRoute('app_login')
  }
  ```
    - Dans l'exemple ci dessus, si l'utilisateur n'est pas connecté on le redirige vers le formulaire de connexion.

### gérer les roles avec Symfony ###

- Pour déclarer un rôle à un utilisateur, vous pouvez vous rendre directement en base de données et lui attribuer son
  rôle à la main dans un tableau.
    - Un role doit tout le temps commencé par le préfix : ROLE_, exemple : ROLE_ADMIN
- Pour empecher les accès sur un niveau global dans votre application, rendez vous dans le
  fichier ```config/package/security.yaml```
    - rechercher la partie ```access_control```
        - ici, vous pouvez rajouter des préfix de route relié à des roles,
          ex : ```- { path: ^/admin, roles: ROLE_ADMIN }```
        - Dans ce cas, pour accéder aux routes /admin vous DEVEZ avoir le ROLE_ADMIN sur votre utilisateur
- Pour empecher les routes sur un niveau plus petit, vous pouvez utiliser la méthode isGranted dans votre controller,
  exemple :
- ```
          if (!$this->isGranted('ROLE_ADMIN')) {
            return $this->redirectToRoute('some_route');
        }
  ```
    - Dans le cas ci dessus, si la personne n'a pas le ROLE_ADMIN, elle sera redirigé vers la route "some_route"
- D'un point de vue Twig, vous pouvez aussi utilisé le isGranted, exemple :
- ```
      {% if is_granted('ROLE_ADMIN') %}
        <p>J'ai le role admin !!!</p>
    {% else %}
        <p>Je n'ai pas le role admin</p>
    {% endif %}

  ```

### Gérer une pagination avec le bundle knp paginator ###
- pour gérer la pagination, nous allons utiliser le bundle knpPaginator : https://github.com/KnpLabs/KnpPaginatorBundle
- Une fois installé, vous devez l'importer dans votre constructeur (même chose que pour un répository) : 
- ```
      public function __construct(
        private AlbumRepository    $albumRepository,

        //use Knp\Component\Pager\PaginatorInterface;
        private PaginatorInterface $paginator
    )
    {
    }
  ```
  - N'oubliez pas le USE !!!
- Maintenant, vous pouvez l'utiliser dans vos controller, de cette facon : 
- ```
          $query = $this->albumRepository->getAll();

        $albums = $this->paginator->paginate(
            $query, //ici je veux bien une QUERY pas les résultats
            $request->query->getInt('page', 1), //il va se baser sur le param $_GET['page']
            5 //le nombre d'éléments à afficher par page
        );
  
  ```
- Dernier étape, dans votre vue, appelez le component qui vous permet de gérer la pagination, de cette facon :
- ```
    <div class="navigation">
        {{ knp_pagination_render(albums) }}
    </div>
  
  ```
  
### Gerer les assets avec symfony ###
- Installation du bundle webpack-encore de symfony : https://symfony.com/doc/current/frontend/encore/installation.html
- ```composer require symfony/webpack-encore-bundle```
- ```yarn```
- Une fois les commandes effectué, supprimer tous vos fichiers et dossier dans le dossier assets.
- Ajouter vous le fichier de script : ```assets/script/script.js```
- Ajouter vous le fichier de style scss : ```assets/style/index.scss```
- Rendez vous maintenant dans le fichier ```webpack.config.js```
  - Le ```.addEntry()``` vous permet d'ajouter des script, si on suit notre config, mettez : 
    - ```.addEntry('main_script', './assets/script/index.js')```
  - Pour le css, vous avez besoin de ```.addStyleEntry()```, exemple : 
      - ```.addStyleEntry('main_style', './assets/style/index.scss')```
- Afin que webpack gère le scss, décommenté la ligne : ```.enableSassLoader()```
- Dernière étape, rendez-vous dans votre base.html.twig et ajouté les entries configurées juste avant, à savoir : 
- ```
      {% block stylesheets %}
        {{ encore_entry_link_tags('main_style') }}
    {% endblock %}

    {% block javascripts %}
        {{ encore_entry_script_tags('main_script') }}
    {% endblock %}
  ```
- Lancer la commande : ```yarn watch```
  - Il va surement vous dire d'installer la dépendance pour le sass, copier coller la commande, puis relancer ```yarn watch```.
