---
lab:
  title: Transformer des données avec un pool SQL serverless
  ilt-use: Lab
---

# Transformer des fichiers avec un pool SQL serverless

Les *analystes* de données utilisent souvent SQL pour interroger des données à des fins d’analyse et de génération de rapports. Les *ingénieurs* de données peuvent également utiliser SQL pour manipuler et transformer des données, souvent dans le cadre d’un pipeline d’ingestion de données ou d’un processus d’extraction, transformation et chargement (ETL).

Dans cet exercice, vous allez utiliser un pool SQL serverless dans Azure Synapse Analytics pour transformer des données dans des fichiers.

Cet exercice devrait prendre environ **30** minutes.

## Avant de commencer

Vous avez besoin d’un [abonnement Azure](https://azure.microsoft.com/free) dans lequel vous avez un accès administratif.

## Provisionner un espace de travail Azure Synapse Analytics

Vous aurez besoin d’un espace de travail Azure Synapse Analytics avec accès à Data Lake Storage. Vous pouvez utiliser le pool SQL serverless intégré pour interroger des fichiers dans le lac de données.

Dans cet exercice, vous allez utiliser la combinaison d’un script PowerShell et d’un modèle ARM pour approvisionner un espace de travail Azure Synapse Analytics.

1. Connectez-vous au [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com`.
2. Utilisez le bouton **[\>_]** à droite de la barre de recherche, en haut de la page, pour créer un environnement Cloud Shell dans le portail Azure, en sélectionnant un environnement ***PowerShell*** et en créant le stockage si vous y êtes invité. Cloud Shell fournit une interface de ligne de commande dans un volet situé en bas du portail Azure, comme illustré ici :

    ![Portail Azure avec un volet Cloud Shell](./images/cloud-shell.png)

    > **Remarque** : si vous avez déjà créé un interpréteur de commandes cloud qui utilise un environnement *Bash*, utilisez le menu déroulant en haut à gauche du volet de l’interpréteur de commandes cloud pour le remplacer par ***PowerShell***.

3. Notez que vous pouvez redimensionner le volet Cloud Shell en faisant glisser la barre de séparation en haut du volet. Vous pouvez aussi utiliser les icônes **&#8212;** , **&#9723;** et **X** situées en haut à droite du volet pour réduire, agrandir et fermer le volet. Pour plus d’informations sur l’utilisation d’Azure Cloud Shell, consultez la [documentation Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. Dans le volet PowerShell, entrez les commandes suivantes pour cloner ce référentiel :

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Une fois que le référentiel a été cloné, entrez les commandes suivantes pour accéder au dossier de cet exercice et exécutez le script **setup.ps1** qu’il contient :

    ```
    cd dp-203/Allfiles/labs/03
    ./setup.ps1
    ```

6. Si vous y êtes invité, choisissez l’abonnement à utiliser (uniquement si vous avez accès à plusieurs abonnements Azure).
7. Lorsque vous y êtes invité, entrez un mot de passe approprié à définir pour votre pool SQL Azure Synapse.

    > **Remarque** : veillez à mémoriser ce mot de passe.

8. Attendez que le script se termine. Cela prend généralement environ 10 minutes, mais dans certains cas, cela peut prendre plus de temps. Pendant que vous attendez, consultez l’article [CETAS avec Synapse SQL](https://docs.microsoft.com/azure/synapse-analytics/sql/develop-tables-cetas) dans la documentation Azure Synapse Analytics.

## Interroger des données dans des fichiers

Le script approvisionne un espace de travail Azure Synapse Analytics et un compte Stockage Azure pour héberger le lac de données, puis charge certains fichiers de données dans le lac de données.

### Afficher les fichiers dans le lac de données

1. Une fois le script terminé, dans le portail Azure, accédez au groupe de ressources **dp203-*xxxxxxx*** qu’il a créé, puis sélectionnez votre espace de travail Synapse.
2. Dans la page **Vue d’ensemble** de votre espace de travail Synapse, dans la carte **Ouvrir Synapse Studio**, sélectionnez **Ouvrir** pour ouvrir Synapse Studio dans un nouvel onglet de navigateur. Connectez-vous si vous y êtes invité.
3. Sur le côté gauche de Synapse Studio, utilisez l’icône **&rsaquo;&rsaquo;** pour développer le menu. Cela permet d’afficher les différentes pages de Synapse Studio qui vous permettront de gérer les ressources et d’effectuer des tâches d’analytique de données.
4. Dans la page **Données**, affichez l’onglet **Lié** et vérifiez que votre espace de travail inclut un lien vers votre compte de stockage Azure Data Lake Storage Gen2, qui doit avoir un nom similaire à **synapse*xxxxxxx* (Primary - datalake*xxxxxxx*)**.
5. Développez votre compte de stockage et vérifiez qu’il contient un conteneur de système de fichiers nommé **files**.
6. Sélectionnez le conteneur **files**, et constatez qu’il contient un dossier nommé **sales**. Ce dossier contient les fichiers de données que vous allez interroger.
7. Ouvrez le dossier **sales** et le dossier **csv** qu’il contient, puis observez que ce dossier contient des fichiers .csv correspondant à trois années de données de vente.
8. Cliquez avec le bouton droit sur l’un des fichiers et sélectionnez **Aperçu** pour afficher les données qu’il contient. Notez que les fichiers contiennent une ligne d’en-tête.
9. Fermez l’aperçu, puis utilisez le bouton **↑** pour revenir au dossier **sales**.

### Utiliser SQL pour interroger des fichiers CSV

1. Sélectionnez le dossier **csv** puis, dans la liste **Nouveau script SQL** de la barre d’outils, sélectionnez **Sélectionner les 100 premières lignes**.
2. Dans la liste **Type de fichier**, sélectionnez **Format texte**, puis appliquez les paramètres pour ouvrir un nouveau script SQL qui interroge les données du dossier.
3. Dans le volet **Propriétés** du **Script SQL 1** créé, remplacez le nom par **Interroger les fichiers CSV de ventes** et modifiez les paramètres de résultat pour afficher **Toutes les lignes**. Ensuite, dans la barre d’outils, sélectionnez **Publier** pour enregistrer le script et utilisez le bouton **Propriétés** (qui ressemble à **<sub>*</sub>**) à droite de la barre d’outils pour masquer le volet **Propriétés**.
4. Passez en revue le code SQL qui a été généré, qui doit ressembler à ceci :

    ```SQL
    -- This is auto-generated code
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/sales/csv/**',
            FORMAT = 'CSV',
            PARSER_VERSION='2.0'
        ) AS [result]
    ```

    Ce code utilise OPENROWSET pour lire les données des fichiers CSV dans le dossier sales et récupère les 100 premières lignes de données.

5. Dans ce cas, les fichiers de données incluent les noms de colonnes dans la première ligne. Modifiez donc la requête pour ajouter un paramètre `HEADER_ROW = TRUE` à la clause `OPENROWSET`, comme illustré ici (n’oubliez pas d’ajouter une virgule après le paramètre précédent) :

    ```SQL
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/sales/csv/**',
            FORMAT = 'CSV',
            PARSER_VERSION='2.0',
            HEADER_ROW = TRUE
        ) AS [result]
    ```

6. Dans la liste **Se connecter à**, vérifiez que l’option **Intégré** est cochée : cela représente le pool SQL intégré qui a été créé avec votre espace de travail. Ensuite, dans la barre d’outils, utilisez le bouton **▷ Exécuter** pour exécuter le code SQL, puis passez en revue les résultats, qui doivent ressembler à ceci :

    | SalesOrderNumber | SalesOrderLineNumber | OrderDate | CustomerName | EmailAddress | Élément | Quantité | UnitPrice | TaxAmount |
    | -- | -- | -- | -- | -- | -- | -- | -- | -- |
    | SO43701 | 1 | 2019-07-01 | Christy Zhu | christy12@adventure-works.com |Mountain-100 Silver, 44 | 1 | 3399.99 | 271,9992 |
    | ... | ... | ... | ... | ... | ... | ... | ... | ... |

7. Publiez les modifications apportées à votre script, puis fermez le volet de script.

## Transformer des données à l’aide d’une instruction CREATE EXTERNAL TABLE AS SELECT (CETAS)

Un moyen simple d’utiliser SQL pour transformer des données dans un fichier et conserver les résultats dans un autre fichier consiste à utiliser une instruction CREATE EXTERNAL TABLE AS SELECT (CETAS). Cette instruction crée une table basée sur les demandes d’une requête, mais les données de la table sont stockées en tant que fichiers dans un lac de données. Les données transformées peuvent ensuite être interrogées via la table externe ou accessibles directement dans le système de fichiers (par exemple, pour inclusion dans un processus en aval pour charger les données transformées dans un entrepôt de données).

### Créer une source de données et un format de fichier externes

En définissant une source de données externe dans une base de données, vous pouvez l’utiliser pour référencer l’emplacement du lac de données auquel vous souhaitez stocker des fichiers pour des tables externes. Un format de fichier externe vous permet de définir le format de ces fichiers (Parquet ou CSV, par exemple). Pour utiliser ces objets pour travailler avec des tables externes, vous devez les créer dans une base de données autre que la base de données **master** par défaut.

1. Dans Synapse Studio, dans la page **Développer**, dans le menu **+**, sélectionnez **Script SQL**.
2. Dans le volet du nouveau script, ajoutez le code suivant (en remplaçant *datalakexxxxxxxxx* par le nom de votre compte Data Lake Storage) pour créer une base de données et y ajouter une source de données externe.

    ```sql
    -- Database for sales data
    CREATE DATABASE Sales
      COLLATE Latin1_General_100_BIN2_UTF8;
    GO;
    
    Use Sales;
    GO;
    
    -- External data is in the Files container in the data lake
    CREATE EXTERNAL DATA SOURCE sales_data WITH (
        LOCATION = 'https://datalakexxxxxxx.dfs.core.windows.net/files/'
    );
    GO;
    
    -- Format for table files
    CREATE EXTERNAL FILE FORMAT ParquetFormat
        WITH (
                FORMAT_TYPE = PARQUET,
                DATA_COMPRESSION = 'org.apache.hadoop.io.compress.SnappyCodec'
            );
    GO;
    ```

3. Modifiez les propriétés du script pour remplacer son nom par **Create Sales DB** et publiez-le.
4. Vérifiez que le script est connecté au pool SQL **Intégré** et à la base de données **master**, puis exécutez-le.
5. Revenez à la page **Données** et utilisez le bouton **↻** en haut à droite de Synapse Studio pour actualiser la page. Affichez ensuite l’onglet **Espace de travail** dans le volet **Données**, qui affiche à présent une liste de **bases de données SQL**. Développez cette liste pour vérifier que la base de données **Sales** a été créée.
6. Développez la base de données **Sales**, son dossier **External Resources** et le dossier **External data sources** sous celui-ci pour afficher la source de données externe **sales_data** que vous avez créée.

### Créer une table externe

1. Dans Synapse Studio, dans la page **Développer**, dans le menu **+**, sélectionnez **Script SQL**.
2. Dans le volet du nouveau script, ajoutez le code suivant pour récupérer et agréger des données à partir des fichiers de ventes CSV à l’aide de la source de données externe. Notez que le chemin **BULK** est relatif à l’emplacement de dossier auquel la source de données est définie :

    ```sql
    USE Sales;
    GO;
    
    SELECT Item AS Product,
           SUM(Quantity) AS ItemsSold,
           ROUND(SUM(UnitPrice) - SUM(TaxAmount), 2) AS NetRevenue
    FROM
        OPENROWSET(
            BULK 'sales/csv/*.csv',
            DATA_SOURCE = 'sales_data',
            FORMAT = 'CSV',
            PARSER_VERSION = '2.0',
            HEADER_ROW = TRUE
        ) AS orders
    GROUP BY Item;
    ```

3. Exécutez le script. Les résultats doivent ressembler à ceci :

    | Produit | ItemsSold | NetRevenue |
    | -- | -- | -- |
    | AWC Logo Cap | 1063 | 8791.86 |
    | ... | ... | ... |

4. Modifiez le code SQL pour enregistrer les résultats de la requête dans une table externe, comme suit :

    ```sql
    CREATE EXTERNAL TABLE ProductSalesTotals
        WITH (
            LOCATION = 'sales/productsales/',
            DATA_SOURCE = sales_data,
            FILE_FORMAT = ParquetFormat
        )
    AS
    SELECT Item AS Product,
        SUM(Quantity) AS ItemsSold,
        ROUND(SUM(UnitPrice) - SUM(TaxAmount), 2) AS NetRevenue
    FROM
        OPENROWSET(
            BULK 'sales/csv/*.csv',
            DATA_SOURCE = 'sales_data',
            FORMAT = 'CSV',
            PARSER_VERSION = '2.0',
            HEADER_ROW = TRUE
        ) AS orders
    GROUP BY Item;
    ```

5. Exécutez le script. Cette fois, il n’y a pas de sortie, mais le code doit avoir créé une table externe en fonction des résultats de la requête.
6. Nommez le script **Créer une table ProductSalesTotals** et publiez-le.
7. Dans la page **Données**, sous l’onglet **Espace de travail**, affichez le contenu du dossier **External tables** de la base de données SQL **Sales** pour vérifier qu’une table nommée **ProductSalesTotals** a été créée.
8. Dans le menu **...** de la table **ProductSalesTotals**, sélectionnez **Nouveau script SQL** > **Sélectionner les 100 premières lignes**. Exécutez ensuite le script résultant et vérifiez qu’il retourne les données de ventes de produits agrégées.
9. Sous l’onglet **Fichiers** contenant le système de fichiers de votre lac de données, affichez le contenu du dossier **Sales** (actualisez la vue si nécessaire) et vérifiez qu’un dossier **productsales** a été créé.
10. Dans le dossier **productsales**, observez qu’un ou plusieurs fichiers portant des noms similaires à ABC123DE----.parquet ont été créés. Ces fichiers contiennent les données de ventes de produits agrégées. Pour le confirmer, vous pouvez sélectionner l’un des fichiers et utiliser le menu **Nouveau script SQL** > **Sélectionner les 100 premières lignes** pour l’interroger directement.

## Encapsuler une transformation de données dans une procédure stockée

Si vous devez fréquemment transformer des données, vous pouvez utiliser une procédure stockée pour encapsuler une instruction CETAS.

1. Dans Synapse Studio, dans la page **Développer**, dans le menu **+**, sélectionnez **Script SQL**.
2. Dans le volet du nouveau script, ajoutez le code suivant pour créer une procédure stockée dans la base de données **Sales** qui agrège les ventes par année et enregistre les résultats dans une table externe :

    ```sql
    USE Sales;
    GO;
    CREATE PROCEDURE sp_GetYearlySales
    AS
    BEGIN
        -- drop existing table
        IF EXISTS (
                SELECT * FROM sys.external_tables
                WHERE name = 'YearlySalesTotals'
            )
            DROP EXTERNAL TABLE YearlySalesTotals
        -- create external table
        CREATE EXTERNAL TABLE YearlySalesTotals
        WITH (
                LOCATION = 'sales/yearlysales/',
                DATA_SOURCE = sales_data,
                FILE_FORMAT = ParquetFormat
            )
        AS
        SELECT YEAR(OrderDate) AS CalendarYear,
                SUM(Quantity) AS ItemsSold,
                ROUND(SUM(UnitPrice) - SUM(TaxAmount), 2) AS NetRevenue
        FROM
            OPENROWSET(
                BULK 'sales/csv/*.csv',
                DATA_SOURCE = 'sales_data',
                FORMAT = 'CSV',
                PARSER_VERSION = '2.0',
                HEADER_ROW = TRUE
            ) AS orders
        GROUP BY YEAR(OrderDate)
    END
    ```

3. Exécutez le script pour créer la procédure stockée.
4. Sous le code que vous venez d’exécuter, ajoutez le code suivant pour appeler la procédure stockée :

    ```sql
    EXEC sp_GetYearlySales;
    ```

5. Sélectionnez uniquement l’instruction `EXEC sp_GetYearlySales;` que vous venez d’ajouter et utilisez le bouton **▷ Exécuter** pour l’exécuter.
6. Sous l’onglet **Fichiers** contenant le système de fichiers de votre lac de données, affichez le contenu du dossier **sales** (actualisez la vue si nécessaire) et vérifiez qu’un dossier **yearlysales** a été créé.
7. Dans le dossier **yearlysales**, observez qu’un fichier Parquet contenant les données de ventes annuelles agrégées a été créé.
8. Revenez au script SQL et réexécutez l’instruction `EXEC sp_GetYearlySales;`. Observez qu’une erreur se produit.

    Même si le script supprime la table externe, le dossier contenant les données n’est pas supprimé. Pour réexécuter la procédure stockée (par exemple, dans le cadre d’un pipeline de transformation de données planifiée), vous devez supprimer les anciennes données.

9. Revenez à l’onglet **Fichiers** et affichez le dossier **sales**. Sélectionnez ensuite le dossier **yearlysales** et supprimez-le.
10. Revenez au script SQL et réexécutez l’instruction `EXEC sp_GetYearlySales;`. Cette fois, l’opération réussit et un nouveau fichier de données est généré.

## Supprimer les ressources Azure

Si vous avez fini d’explorer Azure Synapse Analytics, vous devriez supprimer les ressources que vous avez créées afin d’éviter des coûts Azure inutiles.

1. Fermez l’onglet du navigateur Synapse Studio et revenez dans le portail Azure.
2. Dans le portail Azure, dans la page **Accueil**, sélectionnez **Groupes de ressources**.
3. Sélectionnez le groupe de ressources **dp203-*xxxxxxx*** pour votre espace de travail Synapse Analytics (et non le groupe de ressources managé) et vérifiez qu’il contient l’espace de travail Synapse et le compte de stockage pour votre espace de travail.
4. Au sommet de la page **Vue d’ensemble** de votre groupe de ressources, sélectionnez **Supprimer le groupe de ressources**.
5. Entrez le nom du groupe de ressources **dp203-*xxxxxxx*** pour confirmer que vous souhaitez le supprimer, puis sélectionnez **Supprimer**.

    Après quelques minutes, le groupe de ressources de votre espace de travail Azure Synapse et le groupe de ressources de l’espace de travail managé qui lui est associé seront supprimés.
