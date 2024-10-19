---
lab:
  title: Utiliser Delta Lake dans Azure Synapse Analytics
  ilt-use: Lab
---

# Utiliser Delta Lake avec Spark dans Azure Synapse Analytics

Delta Lake est un projet open source pour créer une couche de stockage de données transactionnelles au-dessus d’un lac de données. Delta Lake ajoute la prise en charge de la sémantique relationnelle pour les opérations de données par lots et de streaming, et permet la création d’une architecture *Lakehouse*, dans laquelle Apache Spark peut être utilisé pour traiter et interroger des données dans des tables basées sur des fichiers sous-jacents dans le lac de données.

Cet exercice devrait prendre environ **40** minutes.

## Avant de commencer

Vous avez besoin d’un [abonnement Azure](https://azure.microsoft.com/free) dans lequel vous avez un accès administratif.

## Provisionner un espace de travail Azure Synapse Analytics

Vous aurez besoin d’un espace de travail Azure Synapse Analytics avec accès au stockage du lac de données et d’un pool Apache Spark que vous pouvez utiliser pour interroger et traiter des fichiers dans le lac de données.

Dans cet exercice, vous allez utiliser la combinaison d’un script PowerShell et d’un modèle ARM pour approvisionner un espace de travail Azure Synapse Analytics.

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
    cd dp-203/Allfiles/labs/07
    ./setup.ps1
    ```

6. Si vous y êtes invité, choisissez l’abonnement à utiliser (uniquement si vous avez accès à plusieurs abonnements Azure).
7. Quand vous y êtes invité, entrez un mot de passe approprié à définir pour votre pool Azure Synapse SQL.

    > **Remarque** : veillez à mémoriser ce mot de passe.

8. Attendez que le script se termine. Cela prend généralement environ 10 minutes, mais dans certains cas, cela peut prendre plus de temps. Pendant que vous attendez, consultez l’article [Présentation de Delta Lake](https://docs.microsoft.com/azure/synapse-analytics/spark/apache-spark-what-is-delta-lake) dans la documentation Azure Synapse Analytics.

## Créer des tables delta

Le script approvisionne un espace de travail Azure Synapse Analytics et un compte Stockage Azure pour héberger le lac de données, puis charge un fichier de données dans le lac de données.

### Explorer les données dans le lac de données

1. Une fois le script terminé, dans le portail Azure, accédez au groupe de ressources **dp203-*xxxxxxx*** qu’il a créé, puis sélectionnez votre espace de travail Synapse.
2. Dans la page **Vue d’ensemble** de votre espace de travail Synapse, dans la carte **Ouvrir Synapse Studio**, sélectionnez **Ouvrir** pour ouvrir Synapse Studio dans un nouvel onglet de navigateur. Connectez-vous si vous y êtes invité.
3. Sur le côté gauche de Synapse Studio, utilisez l’icône **&rsaquo;&rsaquo;** pour développer le menu. Cela permet d’afficher les différentes pages de Synapse Studio qui vous permettront de gérer les ressources et d’effectuer des tâches d’analytique de données.
4. Dans la page **Données**, affichez l’onglet **Lié** et vérifiez que votre espace de travail inclut un lien vers votre compte de stockage Azure Data Lake Storage Gen2, qui doit avoir un nom similaire à **synapse*xxxxxxx* (Primary - datalake*xxxxxxx*)**.
5. Développez votre compte de stockage et vérifiez qu’il contient un conteneur de système de fichiers nommé **files**.
6. Sélectionnez le conteneur **files** et notez qu’il contient un dossier nommé **products**. Ce dossier contient les données avec lesquelles vous allez travailler dans cet exercice.
7. Ouvrez le dossier **products** et notez qu’il contient un fichier nommé **products.csv**.
8. Sélectionnez **products.csv** puis, dans la liste **Nouveau notebook** de la barre d’outils, sélectionnez **Charger dans DataFrame**.
9. Dans le volet **Notebook 1** qui s’ouvre, dans la liste **Attacher à**, sélectionnez le pool Spark **sparkxxxxxxx** et assurez-vous que le **Langage** est défini sur **PySpark (Python)**.
10. Examinez le code dans la première (et unique) cellule du notebook, qui doit se présenter comme suit :

    ```Python
    %%pyspark
    df = spark.read.load('abfss://files@datalakexxxxxxx.dfs.core.windows.net/products/products.csv', format='csv'
    ## If header exists uncomment line below
    ##, header=True
    )
    display(df.limit(10))
    ```

11. Décommentez la ligne *,header=True* (car le fichier products.csv contient les en-têtes de colonnes dans la première ligne), afin que votre code ressemble à ceci :

    ```Python
    %%pyspark
    df = spark.read.load('abfss://files@datalakexxxxxxx.dfs.core.windows.net/products/products.csv', format='csv'
    ## If header exists uncomment line below
    , header=True
    )
    display(df.limit(10))
    ```

12. Utilisez l’icône **&#9655;** à gauche de la cellule de code pour l’exécuter, et attendez les résultats. La première fois que vous exécutez une cellule dans un notebook, le pool Spark démarre. Il peut falloir environ une minute avant que des résultats soient renvoyés. Au final, les résultats devraient apparaître sous la cellule et ressembler à ceci :

    | ProductID | ProductName | Catégorie | ListPrice |
    | -- | -- | -- | -- |
    | 771 | Mountain-100 Silver, 38 | VTT | 3399.9900 |
    | 772 | Mountain-100 Silver, 42 | VTT | 3399.9900 |
    | ... | ... | ... | ... |

### Charger les données du fichier dans une table delta

1. Sous les résultats retournés par la première cellule de code, utilisez le bouton **+ Code** pour ajouter une nouvelle cellule de code. Entrez ensuite le code suivant dans la nouvelle cellule et exécutez-le :

    ```Python
    delta_table_path = "/delta/products-delta"
    df.write.format("delta").save(delta_table_path)
    ```

2. Sous l’onglet **Fichiers**, utilisez l’icône **↑** dans la barre d’outils pour revenir à la racine du conteneur **files** et notez qu’un nouveau dossier nommé **delta** a été créé. Ouvrez ce dossier et la table **products-delta** qu’il contient, où vous devez voir le ou les fichiers de format Parquet contenant les données.

3. Revenez à l’onglet **Notebook 1** et ajoutez une autre nouvelle cellule de code. Ensuite, dans la nouvelle cellule, ajoutez le code suivant et exécutez-le :

    ```Python
    from delta.tables import *
    from pyspark.sql.functions import *

    # Create a deltaTable object
    deltaTable = DeltaTable.forPath(spark, delta_table_path)

    # Update the table (reduce price of product 771 by 10%)
    deltaTable.update(
        condition = "ProductID == 771",
        set = { "ListPrice": "ListPrice * 0.9" })

    # View the updated data as a dataframe
    deltaTable.toDF().show(10)
    ```

    Les données sont chargées dans un objet **DeltaTable** et mises à jour. La mise à jour est reflétée dans les résultats de la requête.

4. Ajoutez une nouvelle cellule de code avec le code suivant et exécutez-la :

    ```Python
    new_df = spark.read.format("delta").load(delta_table_path)
    new_df.show(10)
    ```

    Le code charge les données de table Delta dans une trame de données à partir de son emplacement dans le lac de données, en vérifiant que la modification que vous avez apportée via un objet **DeltaTable** a été conservée.

5. Modifiez le code que vous venez d’exécuter comme suit, en spécifiant l’option permettant d’utiliser la fonctionnalité de *voyage dans le temps* de Delta Lake pour afficher une version précédente des données.

    ```Python
    new_df = spark.read.format("delta").option("versionAsOf", 0).load(delta_table_path)
    new_df.show(10)
    ```

    Lorsque vous exécutez le code modifié, les résultats affichent la version d’origine des données.

6. Ajoutez une nouvelle cellule de code avec le code suivant et exécutez-la :

    ```Python
    deltaTable.history(10).show(20, False, True)
    ```

    L’historique des 20 dernières modifications apportées à la table s’affiche : il doit y en avoir deux (la création d’origine et la mise à jour que vous avez effectuée).

## Créer des tables de catalogue

Jusqu’à présent, vous avez travaillé avec des tables delta en chargeant les données du dossier contenant les fichiers Parquet sur lesquels la table est basée. Vous pouvez définir des *tables de catalogue* qui encapsulent les données et fournissent une entité de table nommée que vous pouvez référencer dans le code SQL. Spark prend en charge deux types de tables de catalogue pour delta lake :

- Les tables *externes* définies par le chemin d’accès aux fichiers Parquet contenant les données de la table.
- Les tables *managées* définies dans le metastore Hive pour le pool Spark.

### Créer une table externe

1. Dans une nouvelle cellule de code, ajoutez et exécutez le code suivant :

    ```Python
    spark.sql("CREATE DATABASE AdventureWorks")
    spark.sql("CREATE TABLE AdventureWorks.ProductsExternal USING DELTA LOCATION '{0}'".format(delta_table_path))
    spark.sql("DESCRIBE EXTENDED AdventureWorks.ProductsExternal").show(truncate=False)
    ```

    Ce code crée une base de données nommée **AdventureWorks**, puis crée une table externe nommée **ProductsExternal** dans cette base de données en fonction du chemin d’accès aux fichiers Parquet que vous avez définis précédemment. Il affiche ensuite une description des propriétés de la table. Notez que la propriété **Emplacement** correspond au chemin d’accès que vous avez spécifié.

2. Ajoutez une nouvelle cellule de code, puis entrez et exécutez le code suivant :

    ```sql
    %%sql

    USE AdventureWorks;

    SELECT * FROM ProductsExternal;
    ```

    Le code utilise SQL pour basculer le contexte vers la base de données **AdventureWorks** (qui ne retourne aucune donnée), puis interroger la table **ProductsExternal** (qui retourne un jeu de résultats contenant les données des produits dans la table Delta Lake).

### Créer une table managée

1. Dans une nouvelle cellule de code, ajoutez et exécutez le code suivant :

    ```Python
    df.write.format("delta").saveAsTable("AdventureWorks.ProductsManaged")
    spark.sql("DESCRIBE EXTENDED AdventureWorks.ProductsManaged").show(truncate=False)
    ```

    Ce code crée une table managée nommée **ProductsManaged** basée sur le DataFrame que vous avez chargé à l’origine à partir du fichier **products.csv** (avant de mettre à jour le prix du produit 771). Vous ne spécifiez pas de chemin d’accès pour les fichiers Parquet utilisés par la table. Celui-ci est géré pour vous dans le metastore Hive et affiché dans la propriété **Emplacement** dans la description de la table (dans **files/synapse/workspaces/synapsexxxxxxx/warehouse**).

2. Ajoutez une nouvelle cellule de code, puis entrez et exécutez le code suivant :

    ```sql
    %%sql

    USE AdventureWorks;

    SELECT * FROM ProductsManaged;
    ```

    Le code utilise SQL pour interroger la table **ProductsManaged**.

### Comparer des tables externes et managées

1. Dans une nouvelle cellule de code, ajoutez et exécutez le code suivant :

    ```sql
    %%sql

    USE AdventureWorks;

    SHOW TABLES;
    ```

    Ce code répertorie les tables de la base de données **AdventureWorks**.

2. Modifiez la cellule de code comme suit et exécutez-la :

    ```sql
    %%sql

    USE AdventureWorks;

    DROP TABLE IF EXISTS ProductsExternal;
    DROP TABLE IF EXISTS ProductsManaged;
    ```

    Ce code supprime les tables du metastore.

3. Revenez à l’onglet **files** et affichez le dossier **files/delta/products-delta**. Notez que les fichiers de données existent toujours à cet emplacement. La suppression de la table externe a supprimé la table du metastore, mais a laissé les fichiers de données intacts.
4. Affichez le dossier **files/synapse/workspaces/synapsexxxxxxx/warehouse**. Notez qu’il n’existe aucun dossier pour les données de table **ProductsManaged**. La suppression d’une table gérée entraîne celle de la table du metastore et des fichiers de données de cette table.

### Créer une table via SQL

1. Ajoutez une nouvelle cellule de code, puis entrez et exécutez le code suivant :

    ```sql
    %%sql

    USE AdventureWorks;

    CREATE TABLE Products
    USING DELTA
    LOCATION '/delta/products-delta';
    ```

2. Ajoutez une nouvelle cellule de code, puis entrez et exécutez le code suivant :

    ```sql
    %%sql

    USE AdventureWorks;

    SELECT * FROM Products;
    ```

    Notez que la nouvelle table de catalogue a été créée pour le dossier de table Delta Lake existant, qui tient compte des modifications apportées précédemment.

## Utiliser des tables delta pour les données de streaming

Delta Lake prend en charge les données de diffusion en continu. Les tables delta peuvent être un *récepteur* ou une *source* pour des flux de données créés en utilisant l’API Spark Structured Streaming. Dans cet exemple, vous allez utiliser une table delta comme récepteur pour des données de streaming dans un scénario IoT (Internet des objets) simulé.

1. Revenez à l’onglet **Notebook 1** et ajoutez une nouvelle cellule de code. Ensuite, dans la nouvelle cellule, ajoutez le code suivant et exécutez-le :

    ```python
    from notebookutils import mssparkutils
    from pyspark.sql.types import *
    from pyspark.sql.functions import *

    # Create a folder
    inputPath = '/data/'
    mssparkutils.fs.mkdirs(inputPath)

    # Create a stream that reads data from the folder, using a JSON schema
    jsonSchema = StructType([
    StructField("device", StringType(), False),
    StructField("status", StringType(), False)
    ])
    iotstream = spark.readStream.schema(jsonSchema).option("maxFilesPerTrigger", 1).json(inputPath)

    # Write some event data to the folder
    device_data = '''{"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev2","status":"error"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"error"}
    {"device":"Dev2","status":"ok"}
    {"device":"Dev2","status":"error"}
    {"device":"Dev1","status":"ok"}'''
    mssparkutils.fs.put(inputPath + "data.txt", device_data, True)
    print("Source stream created...")
    ```

    Vérifiez que le message *Flux source créé...* est affiché. Le code que vous venez d’exécuter a créé une source de données de streaming basée sur un dossier dans lequel des données ont été enregistrées, représentant les lectures d’appareils IoT hypothétiques.

2. Dans une nouvelle cellule de code, ajoutez et exécutez le code suivant :

    ```python
    # Write the stream to a delta table
    delta_stream_table_path = '/delta/iotdevicedata'
    checkpointpath = '/delta/checkpoint'
    deltastream = iotstream.writeStream.format("delta").option("checkpointLocation", checkpointpath).start(delta_stream_table_path)
    print("Streaming to delta sink...")
    ```

    Ce code écrit les données des appareils de streaming au format delta.

3. Dans une nouvelle cellule de code, ajoutez et exécutez le code suivant :

    ```python
    # Read the data in delta format into a dataframe
    df = spark.read.format("delta").load(delta_stream_table_path)
    display(df)
    ```

    Ce code lit les données diffusées en continu au format delta dans un dataframe. Notez que le code permettant de charger des données de streaming n’est pas différent de celui utilisé pour charger des données statiques à partir d’un dossier delta.

4. Dans une nouvelle cellule de code, ajoutez et exécutez le code suivant :

    ```python
    # create a catalog table based on the streaming sink
    spark.sql("CREATE TABLE IotDeviceData USING DELTA LOCATION '{0}'".format(delta_stream_table_path))
    ```

    Ce code crée une table de catalogue nommée **IotDeviceData** (dans la base de données **par défaut**) à partir du dossier delta. Là encore, ce code est identique à celui utilisé pour les données qui ne sont pas diffusées en continu.

5. Dans une nouvelle cellule de code, ajoutez et exécutez le code suivant :

    ```sql
    %%sql

    SELECT * FROM IotDeviceData;
    ```

    Ce code interroge la table **IotDeviceData**, qui contient les données des appareils provenant de la source de streaming.

6. Dans une nouvelle cellule de code, ajoutez et exécutez le code suivant :

    ```python
    # Add more data to the source stream
    more_data = '''{"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"error"}
    {"device":"Dev2","status":"error"}
    {"device":"Dev1","status":"ok"}'''

    mssparkutils.fs.put(inputPath + "more-data.txt", more_data, True)
    ```

    Ce code écrit plus de données d’appareils hypothétiques dans la source de streaming.

7. Dans une nouvelle cellule de code, ajoutez et exécutez le code suivant :

    ```sql
    %%sql

    SELECT * FROM IotDeviceData;
    ```

    Ce code interroge à nouveau la table **IotDeviceData**, qui doit maintenant inclure les données supplémentaires qui ont été ajoutées à la source de streaming.

8. Dans une nouvelle cellule de code, ajoutez et exécutez le code suivant :

    ```python
    deltastream.stop()
    ```

    Ce code arrête le flux.

## Interroger une table delta à partir d’un pool SQL serverless

Outre les pools Spark, Azure Synapse Analytics inclut un pool SQL sans serveur intégré. Vous pouvez utiliser le moteur de base de données relationnelle dans ce pool pour interroger des tables delta via SQL.

1. Dans l’onglet **Fichiers**, accédez au dossier **dfiles/delta**.
2. Sélectionnez le dossier **products-delta**, puis, dans la barre d’outils, dans la liste déroulante **Nouveau script SQL**, sélectionnez **Sélectionner les 100 premières lignes**.
3. Dans le volet **Sélectionner les 100 premières lignes**, dans la liste **Type de fichier**, sélectionnez **Format delta**, puis **Appliquer**.
4. Passez en revue le code SQL généré, qui doit ressembler à ceci :

    ```sql
    -- This is auto-generated code
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/delta/products-delta/',
            FORMAT = 'DELTA'
        ) AS [result]
    ```

5. Utilisez l’icône **▷ Exécuter** pour exécuter le script et passer en revue les résultats. L’application doit ressembler à ceci :

    | ProductID | ProductName | Catégorie | ListPrice |
    | -- | -- | -- | -- |
    | 771 | Mountain-100 Silver, 38 | VTT | 3059.991 |
    | 772 | Mountain-100 Silver, 42 | VTT | 3399.9900 |
    | ... | ... | ... | ... |

    Cela explique comment utiliser un pool SQL serverless pour interroger des fichiers de format Delta créés à l’aide de Spark et utiliser les résultats pour la création de rapports ou l’analyse.

6. Remplacez la requête par le code SQL suivant :

    ```sql
    USE AdventureWorks;

    SELECT * FROM Products;
    ```

7. Exécutez le code et notez que vous pouvez également utiliser le pool SQL serverless pour interroger les données Delta Lake dans les tables de catalogue définies par le metastore Spark.

## Supprimer les ressources Azure

Si vous avez fini d’explorer Azure Synapse Analytics, vous devriez supprimer les ressources que vous avez créées afin d’éviter des coûts Azure inutiles.

1. Fermez l’onglet du navigateur Synapse Studio et revenez dans le portail Azure.
2. Dans le portail Azure, dans la page **Accueil**, sélectionnez **Groupes de ressources**.
3. Sélectionnez le groupe de ressources **dp203-*xxxxxxx*** de votre espace de travail Synapse Analytics (et non le groupe de ressources managé) et vérifiez qu’il contient l’espace de travail Synapse, le compte de stockage et le pool Spark de votre espace de travail.
4. Au sommet de la page **Vue d’ensemble** de votre groupe de ressources, sélectionnez **Supprimer le groupe de ressources**.
5. Entrez le nom du groupe de ressources **dp203-*xxxxxxx*** pour confirmer que vous souhaitez le supprimer, puis sélectionnez **Supprimer**.

    Après quelques minutes, le groupe de ressources de l’espace de travail Azure Synapse et le groupe de ressources managé de l’espace de travail qui lui est associé seront supprimés.
