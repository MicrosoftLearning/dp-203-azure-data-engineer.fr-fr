---
lab:
  title: "Utiliser Azure\_Synapse\_Link pour Azure\_Cosmos\_DB"
  ilt-use: Lab
---

# Utiliser Azure Synapse Link pour Azure Cosmos DB

Azure Synapse Link pour Azure Cosmos DB est une technologie de *traitement analytique transactionnel hybride (HTAP)* native cloud qui vous permet de procéder à une analytique en quasi-temps réel des données opérationnelles stockées dans Azure Cosmos DB à partir d’Azure Synapse Analytics.

Cet exercice devrait prendre environ **35** minutes.

## Avant de commencer

Vous avez besoin d’un [abonnement Azure](https://azure.microsoft.com/free) dans lequel vous avez un accès administratif.

## Approvisionner des ressources Azure

Pour explorer Azure Synapse Link pour Azure Cosmos DB, vous aurez besoin d’un espace de travail Azure Synapse Analytics et d’un compte Azure Cosmos DB. Dans cet exercice, vous allez utiliser la combinaison d’un script PowerShell et d’un modèle ARM pour approvisionner ces ressources dans votre abonnement Azure.

1. Connectez-vous au [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com`.
2. Utilisez le bouton **[\>_]** à droite de la barre de recherche, en haut de la page, pour créer un environnement Cloud Shell dans le portail Azure, puis sélectionnez un environnement ***PowerShell*** et créez le stockage si vous y êtes invité. Cloud Shell fournit une interface de ligne de commande dans un volet situé en bas du portail Azure, comme illustré ici :

    ![Portail Azure avec un volet Cloud Shell](./images/cloud-shell.png)

    > **Remarque** : si vous avez créé un shell cloud qui utilise un environnement *Bash*, utilisez le menu déroulant en haut à gauche du volet Cloud Shell pour le remplacer par ***PowerShell***.

3. Notez que vous pouvez redimensionner le volet Cloud Shell en faisant glisser la barre de séparation en haut du volet. Vous pouvez aussi utiliser les icônes **&#8212;** , **&#9723;** et **X** situées en haut à droite du volet pour réduire, agrandir et fermer le volet. Pour plus d’informations sur l’utilisation d’Azure Cloud Shell, consultez la [documentation Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. Dans le volet PowerShell, entrez les commandes suivantes pour cloner ce référentiel :

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Une fois que le référentiel a été cloné, entrez les commandes suivantes pour accéder au dossier de cet exercice et exécutez le script **setup.ps1** qu’il contient :

    ```
    cd dp-203/Allfiles/labs/14
    ./setup.ps1
    ```

6. Si vous y êtes invité, choisissez l’abonnement à utiliser (uniquement si vous avez accès à plusieurs abonnements Azure).
7. Quand vous y êtes invité, entrez un mot de passe approprié à définir pour votre pool Azure Synapse SQL.

    > **Remarque** : veillez à mémoriser ce mot de passe.

8. Attendez que le script se termine. Cela prend généralement environ 10 minutes, mais dans certains cas, cela peut prendre plus de temps. Pendant que vous attendez, consultez l’article [Qu’est-ce qu’Azure Synapse Link pour Azure Cosmos DB ?](https://docs.microsoft.com/azure/cosmos-db/synapse-link) dans la documentation Azure Synapse Analytics.

## Configurer Synapse Link dans Azure Cosmos DB

Avant de pouvoir utiliser Synapse Link pour Azure Cosmos DB, vous devez l’activer dans votre compte Azure Cosmos DB et configurer un conteneur en tant que magasin analytique.

### Activer la fonctionnalité Azure Synapse Link dans votre compte Cosmos DB

1. Dans le [portail Azure](https://portal.azure.com), accédez au groupe de ressources **dp203-*xxxxxxx*** créé par le script d’installation et identifiez votre compte Cosmos DB **cosmos*xxxxxxxx***.

    > **Remarque** : dans certains cas, le script a peut-être essayé de créer des comptes Cosmos DB dans plusieurs régions. Il peut donc y avoir un ou plusieurs comptes dans un état de *suppression*. Le compte actif doit être celui avec le plus grand nombre à la fin de son nom, par exemple **cosmos*xxxxxxx*3**.

2. Ouvrez votre compte Azure Cosmos DB, puis sélectionnez la page **Data Explorer** à gauche de son panneau.

    *Si une boîte de dialogue **Bienvenue** s’affiche, fermez-la.*

3. En haut de la page **Data Explorer**, utilisez le bouton **Activer Azure Synapse Link** pour activer Synapse Link.

    ![Cosmos DB Data Explorer avec le bouton Activer Azure Synapse Link mis en surbrillance](./images/cosmos-enable-synapse-link.png)

4. À gauche de la page, dans la section **Intégrations**, sélectionnez la page **Azure Synapse Link** et vérifiez que l’état du compte est *Activé*.

### Créer un conteneur de magasin analytique

1. Revenez à la page **Data Explorer** et utilisez le bouton (ou la vignette) de **nouveau conteneur** pour créer un conteneur avec les paramètres suivants :
    - **ID de base de données** : *(Créer)* AdventureWorks
    - **Partager le débit entre les conteneurs** : <u>Non</u> sélectionné
    - **ID de conteneur** : Sales
    - **Clé de partition** : /customerid
    - **Débit du conteneur (mise à l’échelle automatique)**  : mise à l’échelle automatique
    - **Nombre maximal de RU/s du conteneur** : 4 000
    - **Magasin analytique** : activé

    > **Remarque** : dans ce scénario, **customerid** est utilisé comme clé de partition, car il est susceptible d’être utilisé dans la plupart des requêtes pour extraire les informations sur les clients et les commandes dans une application hypothétique. Il a une cardinalité relativement élevée (nombre de valeurs uniques), il permettra ainsi au conteneur d’évoluer à mesure que le nombre de clients et de commandes augmentera. L’utilisation de la mise à l’échelle automatique et la définition de la valeur maximale sur 4 000 RU/s convient à une nouvelle application avec des volumes de requête initialement faibles. Une valeur maximale de 4 000 RU/s permettra au conteneur de s’adapter automatiquement entre cette valeur et 10 % de cette valeur maximale (400 RU/s) si elle n’est pas nécessaire.

2. Une fois le conteneur créé, dans la page **Data Explorer**, développez la base de données **AdventureWorks** et son dossier **Sales**, puis sélectionnez le dossier **Items**.

    ![Le dossier AdventureWorks, Sales, Items dans Data Explorer](./images/cosmos-items-folder.png)

3. Utilisez le bouton **Nouvel élément** pour créer un élément client en fonction du code JSON suivant. Enregistrez ensuite le nouvel élément (certains champs de métadonnées supplémentaires seront ajoutés lorsque vous enregistrez l’élément).

    ```json
    {
        "id": "SO43701",
        "orderdate": "2019-07-01",
        "customerid": 123,
        "customerdetails": {
            "customername": "Christy Zhu",
            "customeremail": "christy12@adventure-works.com"
        },
        "product": "Mountain-100 Silver, 44",
        "quantity": 1,
        "price": 3399.99
    }
    ```

4. Ajoutez un deuxième élément avec le code JSON suivant :

    ```json
    {
        "id": "SO43704",
        "orderdate": "2019-07-01",
        "customerid": 124,
        "customerdetails": {
            "customername": "Julio Ruiz",
            "customeremail": "julio1@adventure-works.com"
        },
        "product": "Mountain-100 Black, 48",
        "quantity": 1,
        "price": 3374.99
    }
    ```

5. Ajoutez un troisième élément avec le code JSON suivant :

    ```json
    {
        "id": "SO43707",
        "orderdate": "2019-07-02",
        "customerid": 125,
        "customerdetails": {
            "customername": "Emma Brown",
            "customeremail": "emma3@adventure-works.com"
        },
        "product": "Road-150 Red, 48",
        "quantity": 1,
        "price": 3578.27
    }
    ```

> **Remarque** : en réalité, le magasin analytique contient un volume de données beaucoup plus important, écrit dans le magasin par une application. Ces quelques éléments seront suffisants pour démontrer le principe dans cet exercice.

## Configurer Synapse Link dans Azure Synapse Analytics

Maintenant que vous avez préparé votre compte Azure Cosmos DB, vous pouvez configurer Azure Synapse Link pour Azure Cosmos DB dans votre espace de travail Azure Synapse Analytics.

1. Dans le portail Azure, fermez le panneau de votre compte Cosmos DB s’il est toujours ouvert et revenez au groupe de ressources **dp203-*xxxxxxx***.
2. Ouvrez l’espace de travail Sypnase **synapse*xxxxxxx*** et, dans sa page **Vue d’ensemble**, sur la fiche **Ouvrir Synapse Studio**, sélectionnez **Ouvrir** pour ouvrir Synapse Studio dans un nouvel onglet de navigateur ; connectez-vous si vous y êtes invité.
3. Sur le côté gauche de Synapse Studio, utilisez l’icône **&rsaquo;&rsaquo;** pour développer le menu. Cela permet d’afficher les différentes pages de Synapse Studio.
4. Dans la page **Données**, affichez l’onglet **Lié**. Votre espace de travail doit déjà inclure un lien vers votre compte de stockage Azure Data Lake Storage Gen2, mais aucun lien vers votre compte Cosmos DB.
5. Dans le menu **+**, sélectionnez **Se connecter aux données externes**, puis sélectionnez **Azure Cosmos DB pour NoSQL**.

    ![Ajout d’un lien de données externes de l’API NoSQL Azure Cosmos DB](./images/add-cosmos-db-link.png)

6. Poursuivez et créez une connexion Cosmos DB avec les paramètres suivants :
    - **Nom** : AdventureWorks
    - **Description** : base de données AdventureWorks Cosmos DB
    - **Se connecter via un runtime d'intégration** : AutoResolveIntegrationRuntime
    - **Type d’authentification** : clé de compte
    - **Chaîne de connexion** : *sélectionnée*
    - **Méthode de sélection du compte** : À partir de l’abonnement
    - **Abonnement Azure** : *sélectionnez votre abonnement Azure*
    - **Nom du compte Azure Cosmos DB** : *sélectionnez votre compte **cosmosxxxxxxx***
    - **Nom de la base de données** : AdventureWorks
7. Après avoir créé la connexion, utilisez le bouton **↻** en haut à droite de la page **Données** pour actualiser l’affichage jusqu’à ce qu’une catégorie **Azure Cosmos DB** soit répertoriée dans le volet **Lié**.
8. Développez la catégorie **Azure Cosmos DB** pour voir la connexion **AdventureWorks** que vous avez créée et le conteneur **Sales** qu’elle contient.

    ![Ajout d’un lien de données externes de l’API SQL Azure Cosmos DB](./images/cosmos-linked-connection.png)

## Interroger Azure Cosmos DB à partir d’Azure Synapse Analytics

Vous êtes maintenant prêt à interroger votre base de données Cosmos DB à partir d’Azure Synapse Analytics.

### Interroger Azure Cosmos DB à partir d’un pool Spark

1. Dans le volet **Données**, sélectionnez le conteneur **Sales** puis, dans son menu **...**, sélectionnez **Nouveau notebook** > **Charger dans DataFrame**.
2. Dans le nouvel onglet **Notebook 1** qui s’ouvre, dans la liste **Joindre à**, sélectionnez votre pool Spark (**spark*xxxxxxx***). Utilisez ensuite le bouton **▷ Exécuter tout** pour exécuter toutes les cellules du notebook (il n’y en a actuellement qu’une seule !).

    Remarque : comme c’est la première fois que vous exécutez du code Spark dans cette session, le pool Spark doit être démarré. Cela signifie que la première exécution dans la session peut prendre quelques minutes. Les exécutions suivantes seront plus rapides.

3. En attendant que la session Spark s’initialise, passez en revue le code généré (vous pouvez utiliser le bouton **Propriétés**, qui ressemble à **<sub>*</sub>**, à l’extrémité droite de la barre d’outils pour fermer le volet **Propriétés** afin de voir le code plus clairement). Le code devrait ressembler à ceci :

    ```python
    # Read from Cosmos DB analytical store into a Spark DataFrame and display 10 rows from the DataFrame
    # To select a preferred list of regions in a multi-region Cosmos DB account, add .option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

    df = spark.read\
        .format("cosmos.olap")\
        .option("spark.synapse.linkedService", "AdventureWorks")\
        .option("spark.cosmos.container", "Sales")\
        .load()

    display(df.limit(10))
    ```

4. Lorsque l’exécution du code est terminée, passez en revue la sortie sous la cellule du notebook. Les résultats doivent inclure trois enregistrements, un pour chacun des éléments que vous avez ajoutés à la base de données Cosmos DB. Chaque enregistrement inclut les champs que vous avez entrés lors de la création des éléments, ainsi que certains des champs de métadonnées générés automatiquement.
5. Sous les résultats de la cellule précédente, utilisez l’icône **+ Code** pour ajouter une nouvelle cellule au notebook, puis entrez le code suivant dans celle-ci :

    ```python
    customer_df = df.select("customerid", "customerdetails")
    display(customer_df)
    ```

6. Utilisez l’icône **▷** à gauche de la cellule pour l’exécuter, puis affichez les résultats, qui doivent ressembler à ceci :

    | customerid | customerdetails |
    | -- | -- |
    | 124 | "{"customername": "Julio Ruiz","customeremail": "julio1@adventure-works.com"}" |
    | 125 | "{"customername": "Emma Brown","customeremail": "emma3@adventure-works.com"}" |
    | 123 | "{"customername": "Christy Zhu","customeremail": "christy12@adventure-works.com"}" |

    Cette requête a créé un dataframe contenant uniquement les colonnes **customerid** et **customerdetails**. Notez que la colonne **customerdetails** contient la structure JSON des données imbriquées dans l’élément source. Dans la table des résultats qui s’affiche, vous pouvez utiliser l’icône **►** en regard de la valeur JSON pour la développer et voir les champs individuels qu’elle contient.

7. Ajoutez une autre nouvelle cellule de code et entrez le code suivant :

    ```python
    customerdetails_df = df.select("customerid", "customerdetails.*")
    display(customerdetails_df)
    ```

8. Exécutez la cellule et passez en revue les résultats, qui doivent inclure le **customername** et le **customeremail** à partir de la valeur **customerdetails** sous forme de colonnes :

    | customerid | customername | customeremail |
    | -- | -- | -- |
    | 124 | Julio Ruiz |julio1@adventure-works.com |
    | 125 | Emma Brown |emma3@adventure-works.com |
    | 123 | Christy Zhu | christy12@adventure-works.com |

    Spark vous permet d’exécuter du code de manipulation de données complexe pour restructurer et explorer les données de Cosmos DB. Dans ce cas, le langage PySpark vous permet de naviguer dans la hiérarchie des propriétés JSON pour récupérer les champs enfants du champ **customerdetails**.

9. Ajoutez une autre nouvelle cellule de code et entrez le code suivant :

    ```sql
    %%sql

    -- Create a logical database in the Spark metastore
    CREATE DATABASE salesdb;

    USE salesdb;

    -- Create a table from the Cosmos DB container
    CREATE TABLE salesorders using cosmos.olap options (
        spark.synapse.linkedService 'AdventureWorks',
        spark.cosmos.container 'Sales'
    );

    -- Query the table
    SELECT *
    FROM salesorders;
    ```

10. Exécutez la nouvelle cellule pour créer une base de données contenant une table qui inclut des données du magasin analytique Cosmos DB.
11. Ajoutez une autre nouvelle cellule de code, puis entrez et exécutez le code suivant :

    ```sql
    %%sql

    SELECT id, orderdate, customerdetails.customername, product
    FROM salesorders
    ORDER BY id;
    ```

    Les résultats de cette requête doivent ressembler à ceci :

    | id | orderdate | customername | product |
    | -- | -- | -- | -- |
    | SO43701 | 2019-07-01 | Christy Zhu | Mountain-100 Silver, 44 |
    | SO43704 | 2019-07-01 | Julio Ruiz |Mountain-100 Black, 48 |
    | SO43707 | 2019-07-02 | Emma Brown |Road-150 Red, 48 |

    Notez que lors de l’utilisation de Spark SQL, vous pouvez récupérer les propriétés nommées d’une structure JSON sous forme de colonnes.

12. Conservez l’onglet **Notebook 1** ouvert. Vous y retournerez ultérieurement.

### Interroger Azure Cosmos DB à partir d’un pool SQL serverless

1. Dans le volet **Données**, sélectionnez le conteneur **Sales** puis, dans son menu **...**, sélectionnez **Nouveau script SQL** > **Sélectionner les 100 premières lignes**.
2. Dans le volet **Script SQL 1** qui s’ouvre, masquez le volet **Propriétés** et affichez le code qui a été généré, qui doit ressembler à ceci :

    ```sql
    IF (NOT EXISTS(SELECT * FROM sys.credentials WHERE name = 'cosmosxxxxxxxx'))
    THROW 50000, 'As a prerequisite, create a credential with Azure Cosmos DB key in SECRET option:
    CREATE CREDENTIAL [cosmosxxxxxxxx]
    WITH IDENTITY = ''SHARED ACCESS SIGNATURE'', SECRET = ''<Enter your Azure Cosmos DB key here>''', 0
    GO

    SELECT TOP 100 *
    FROM OPENROWSET(PROVIDER = 'CosmosDB',
                    CONNECTION = 'Account=cosmosxxxxxxxx;Database=AdventureWorks',
                    OBJECT = 'Sales',
                    SERVER_CREDENTIAL = 'cosmosxxxxxxxx'
    ) AS [Sales]
    ```

    Le pool SQL nécessite des informations d’identification à utiliser lors de l’accès à Cosmos DB, qui sont basées sur une clé d’autorisation pour votre compte Cosmos DB. Le script inclut une instruction `IF (NOT EXISTS(...` initiale qui vérifie ces informations d’identification et qui renvoie une erreur si elles n’existent pas.

3. Remplacez l’instruction `IF (NOT EXISTS(...` dans le script par le code suivant pour créer des informations d’identification, en remplaçant *cosmosxxxxxx* par le nom de votre compte Cosmos DB :

    ```sql
    CREATE CREDENTIAL [cosmosxxxxxxxx]
    WITH IDENTITY = 'SHARED ACCESS SIGNATURE',
    SECRET = '<Enter your Azure Cosmos DB key here>'
    GO
    ```

    Le script dans son ensemble doit maintenant ressembler à ce qui suit :

    ```sql
    CREATE CREDENTIAL [cosmosxxxxxxxx]
    WITH IDENTITY = 'SHARED ACCESS SIGNATURE',
    SECRET = '<Enter your Azure Cosmos DB key here>'
    GO

    SELECT TOP 100 *
    FROM OPENROWSET(PROVIDER = 'CosmosDB',
                    CONNECTION = 'Account=cosmosxxxxxxxx;Database=AdventureWorks',
                    OBJECT = 'Sales',
                    SERVER_CREDENTIAL = 'cosmosxxxxxxxx'
    ) AS [Sales]
    ```

4. Basculez vers l’onglet du navigateur contenant le portail Azure (ou ouvrez un nouvel onglet et connectez-vous au portail Azure à l’adresse [https://portal.azure.com](https://portal.azure.com)). Ensuite, dans le groupe de ressources **dp203-*xxxxxxx***, ouvrez votre compte Azure Cosmos DB **cosmos*xxxxxxxx***.
5. Dans le volet de gauche, dans la section **Paramètres**, sélectionnez la page **Clés**. Copiez ensuite la valeur **Clé primaire** dans le Presse-papiers.
6. Revenez à l’onglet du navigateur contenant le script SQL dans Azure Synapse Studio et collez la clé dans le code en remplaçant l’espace ***\<Enter your Azure Cosmos DB key here\>*** réservé afin que le script ressemble à ceci :

    ```sql
    CREATE CREDENTIAL [cosmosxxxxxxxx]
    WITH IDENTITY = 'SHARED ACCESS SIGNATURE',
    SECRET = '1a2b3c....................................=='
    GO

    SELECT TOP 100 *
    FROM OPENROWSET(PROVIDER = 'CosmosDB',
                    CONNECTION = 'Account=cosmosxxxxxxxx;Database=AdventureWorks',
                    OBJECT = 'Sales',
                    SERVER_CREDENTIAL = 'cosmosxxxxxxxx'
    ) AS [Sales]
    ```

7. Utilisez le bouton **▷ Exécuter** pour exécuter le script et passez en revue les résultats, qui doivent inclure trois enregistrements : un pour chacun des éléments que vous avez ajoutés à la base de données Cosmos DB.

    Maintenant que vous avez créé les informations d’identification, vous pouvez les utiliser dans n’importe quelle requête sur la source de données Cosmos DB.

8. Remplacez tout le code dans le script (instructions CREATE CREDENTIAL et SELECT) par le code suivant (en remplaçant *cosmosxxxxxx* par le nom de votre compte Azure Cosmos DB) :

    ```sql
    SELECT *
    FROM OPENROWSET(PROVIDER = 'CosmosDB',
                    CONNECTION = 'Account=cosmosxxxxxxxx;Database=AdventureWorks',
                    OBJECT = 'Sales',
                    SERVER_CREDENTIAL = 'cosmosxxxxxxxx'
    )
    WITH (
        OrderID VARCHAR(10) '$.id',
        OrderDate VARCHAR(10) '$.orderdate',
        CustomerID INTEGER '$.customerid',
        CustomerName VARCHAR(40) '$.customerdetails.customername',
        CustomerEmail VARCHAR(30) '$.customerdetails.customeremail',
        Product VARCHAR(30) '$.product',
        Quantity INTEGER '$.quantity',
        Price FLOAT '$.price'
    )
    AS sales
    ORDER BY OrderID;
    ```

9. Exécutez le script et passez en revue les résultats, qui doivent correspondre au schéma défini dans la clause `WITH` :

    | OrderID | OrderDate | IDClient | CustomerName | CustomerEmail | Produit | Quantity | Prix |
    | -- | -- | -- | -- | -- | -- | -- | -- |
    | SO43701 | 2019-07-01 | 123 | Christy Zhu | christy12@adventure-works.com | Mountain-100 Silver, 44 | 1 | 3399.99 |
    | SO43704 | 2019-07-01 | 124 | Julio Ruiz | julio1@adventure-works.com | Mountain-100 Black, 48 | 1 | 3374.99 |
    | SO43707 | 2019-07-02 | 125 | Emma Brown | emma3@adventure-works.com | Road-150 Red, 48 | 1 | 3578.27 |

10. Conservez l’onglet **Script SQL 1** ouvert. Vous y retournerez ultérieurement.

### Vérifier que les modifications de données dans Cosmos DB sont reflétées dans Synapse 

1. En laissant l’onglet du navigateur contenant Synapse Studio ouvert, revenez à l’onglet contenant le portail Azure, qui doit être ouvert sur la page **Clés** de votre compte Cosmos DB.
2. Dans la page **Data Explorer**, développez la base de données **AdventureWorks** et son dossier **Sales**, puis sélectionnez le dossier **Items**.
3. Utilisez le bouton **Nouvel élément** pour créer un élément client en fonction du code JSON suivant. Enregistrez ensuite le nouvel élément (certains champs de métadonnées supplémentaires seront ajoutés lorsque vous enregistrez l’élément).

    ```json
    {
        "id": "SO43708",
        "orderdate": "2019-07-02",
        "customerid": 126,
        "customerdetails": {
            "customername": "Samir Nadoy",
            "customeremail": "samir1@adventure-works.com"
        },
        "product": "Road-150 Black, 48",
        "quantity": 1,
        "price": 3578.27
    }
    ```

4. Revenez à l’onglet Synapse Studio et, dans l’onglet **Script SQL 1**, réexécutez la requête. Initialement, il peut afficher les mêmes résultats qu’auparavant, mais attendez une minute, puis réexécutez la requête jusqu’à ce que les résultats incluent la vente à Samir Nadoy le 2019-07-02.
5. Revenez à l’onglet **Notebook 1** et réexécutez la dernière cellule du notebook Spark pour vérifier que la vente à Samir Nadoy est désormais incluse dans les résultats de la requête.

## Supprimer les ressources Azure

Si vous avez fini d’explorer Azure Synapse Analytics, vous devriez supprimer les ressources que vous avez créées afin d’éviter des coûts Azure inutiles.

1. Fermez l’onglet du navigateur Synapse Studio et revenez dans le portail Azure.
2. Dans le portail Azure, dans la page **Accueil**, sélectionnez **Groupes de ressources**.
3. Sélectionnez le groupe de ressources **dp203-*xxxxxxx*** de votre espace de travail Synapse Analytics (et non le groupe de ressources managé) et vérifiez qu’il contient l’espace de travail Synapse, le compte de stockage et le pool Spark de votre espace de travail, ainsi que votre compte Azure Cosmos DB.
4. Au sommet de la page **Vue d’ensemble** de votre groupe de ressources, sélectionnez **Supprimer le groupe de ressources**.
5. Entrez le nom du groupe de ressources **dp203-*xxxxxxx*** pour confirmer que vous souhaitez le supprimer, puis sélectionnez **Supprimer**.

    Après quelques minutes, le groupe de ressources de l’espace de travail Azure Synapse et le groupe de ressources managé de l’espace de travail qui lui est associé seront supprimés.
