---
lab:
  title: Charger des données dans un entrepôt de données relationnel
  ilt-use: Lab
---

# Charger des données dans un entrepôt de données relationnel

Dans cet exercice, vous allez charger des données dans un pool SQL dédié.

Cet exercice devrait prendre environ **30** minutes.

## Avant de commencer

Vous avez besoin d’un [abonnement Azure](https://azure.microsoft.com/free) dans lequel vous avez un accès administratif.

## Provisionner un espace de travail Azure Synapse Analytics

Vous aurez besoin d’un espace de travail Azure Synapse Analytics avec accès au stockage data lake et à un pool SQL dédié hébergeant un entrepôt de données.

Dans cet exercice, vous allez utiliser une combinaison d’un script PowerShell et d’un modèle ARM pour approvisionner un espace de travail Azure Synapse Analytics.

1. Connectez-vous au [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com`.
2. Utilisez le bouton **[\>_]** à droite de la barre de recherche, en haut de la page, pour créer un environnement Cloud Shell dans le portail Azure, en sélectionnant un environnement ***Bash*** et en créant le stockage si vous y êtes invité. Cloud Shell fournit une interface de ligne de commande dans un volet situé en bas du portail Azure, comme illustré ici :

    ![Portail Azure avec un volet Cloud Shell](./images/cloud-shell.png)

    > **Remarque** : Si vous avez créé un interpréteur de commandes cloud qui utilise un *environnement Bash* , utilisez le menu déroulant en haut à gauche du volet Cloud Shell pour le remplacer par ***PowerShell***.

3. Notez que vous pouvez redimensionner le volet Cloud Shell en faisant glisser la barre de séparation en haut du volet. Vous pouvez aussi utiliser les icônes &#8212; , &#9723; et X situées en haut à droite du volet pour réduire, agrandir et fermer le volet. Pour plus d’informations sur l’utilisation d’Azure Cloud Shell, consultez la [documentation Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. Dans le terminal, entrez les commandes suivantes pour cloner ce dépôt :

    ```powershell
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Une fois que le dépôt a été cloné, entrez les commandes suivantes pour accéder au dossier de ce labo et exécutez le script **setup.sh** qu’il contient :

    ```powershell
    cd dp-203/Allfiles/labs/09
    ./setup.ps1
    ```

6. Si vous y êtes invité, choisissez l’abonnement que vous souhaitez utiliser (cette option se produit uniquement si vous avez accès à plusieurs abonnements Azure).
7. Lorsque vous y êtes invité, entrez un mot de passe approprié à définir pour votre pool Azure Synapse SQL.

    > Veillez à le mémoriser.

8. Attendez que le script se termine, ce qui prend généralement entre 5 et 10 minutes. Pendant que vous attendez, consultez les stratégies de chargement des données pour le [pool SQL dédié dans l’article Azure Synapse Analytics](https://learn.microsoft.com/azure/synapse-analytics/sql-data-warehouse/design-elt-data-loading) de la documentation Azure Synapse Analytics.

## Préparer le chargement des données

1. Une fois le script terminé, dans le Portail Azure, accédez au **groupe de ressources dp203-*xxxxxxx*** qu’il a créé, puis sélectionnez votre espace de travail Synapse.
2. Dans la **page** Vue d’ensemble de votre espace de travail Synapse, dans l’carte **Ouvrir Synapse Studio**, sélectionnez **Ouvrir** pour ouvrir Synapse Studio dans un nouvel onglet de navigateur ; connectez-vous si vous y êtes invité.
3. Sur le côté gauche de Synapse Studio, utilisez l’icône  pour développer le menu. Cela permet d’afficher les différentes pages de Synapse Studio qui vous permettront de gérer les ressources et d’effectuer des tâches d’analytique de données.
4. Dans la **page Gérer** , sous l’onglet **Pools** SQL, sélectionnez la ligne du **pool SQL dédié sql*xxxxxxx***, qui héberge l’entrepôt de données pour cet exercice et utilisez son **icône &#9655 ;** pour le démarrer ; confirmant que vous souhaitez le reprendre lorsque vous y êtes invité.

    La reprise d’un pool peut prendre plusieurs minutes. Vous pouvez utiliser le **&#8635 ; Bouton Actualiser** pour case activée son état régulièrement. L’état s’affiche en ligne **** quand il est prêt. Pendant que vous attendez, passez aux étapes ci-dessous pour afficher les fichiers de données que vous allez charger.

5. Dans la **page Données**, affichez l’onglet **Lié** et vérifiez que votre espace de travail inclut un lien vers votre compte de stockage Azure Data Lake Stockage Gen2, qui doit avoir un nom similaire à **synapsexxx (Primary - datalakexxxxxxxxx)**.
6. Développez votre compte de stockage et vérifiez qu’il contient un conteneur de système de fichiers nommé **fichiers (principal)** .
7. Sélectionnez le conteneur de fichiers, puis notez qu’il contient un dossier nommé **données**. Ce dossier contient les fichiers de données que vous allez charger dans l’entrepôt de données.
8. Ouvrez le **dossier de données** et observez qu’il contient des fichiers .csv de données client et produit.
9. Cliquez avec le bouton droit sur l’un des fichiers et sélectionnez **Aperçu** pour afficher les données qu’il contient. Notez que les fichiers contiennent une ligne d’en-tête. Vous pouvez donc sélectionner l’option permettant d’afficher les en-têtes de colonne.
10. Revenez à la **page Gérer** et vérifiez que votre pool SQL dédié est en ligne.

## Charger des tables d’entrepôt de données

Examinons certaines approches basées sur SQL pour charger des données dans l’entrepôt de données.

1. Dans la  **page Données** , sélectionnez l’onglet **Espace de travail** .
2. Développez **SQL Database** et sélectionnez votre **base de données sql*xxxxxxx***. Ensuite, dans son menu ...** , sélectionnez **Nouveau script** > 
**** SQL vide.**

Vous disposez maintenant d’une page SQL vide, qui est connectée à l’instance pour les exercices suivants. Vous allez utiliser ce script pour explorer plusieurs techniques SQL que vous pouvez utiliser pour charger des données.

### Charger des données à partir d’un lac de données à l’aide de l’instruction COPY

1. Copiez le code suivant dans la fenêtre de script.

    ```sql
    SELECT COUNT(1) 
    FROM dbo.StageProduct
    ```

2. Dans la barre d’outils, utilisez le **fichier &#9655 ; Bouton Exécuter** pour exécuter le code SQL et confirmer qu’il existe **0** lignes actuellement dans la **table StageProduct** .
3. Remplacez le code par l’instruction COPY suivante (modification **de datalake*xxxxxx*** au nom de votre lac de données) :

    ```sql
    COPY INTO dbo.StageProduct
        (ProductID, ProductName, ProductCategory, Color, Size, ListPrice, Discontinued)
    FROM 'https://datalakexxxxxx.blob.core.windows.net/files/data/Product.csv'
    WITH
    (
        FILE_TYPE = 'CSV',
        MAXERRORS = 0,
        IDENTITY_INSERT = 'OFF',
        FIRSTROW = 2 --Skip header row
    );


    SELECT COUNT(1) 
    FROM dbo.StageProduct
    ```

4. Exécutez la cellule et passez en revue les résultats. 11 lignes doivent avoir été chargées dans la **table StageProduct** .

    Nous allons maintenant utiliser la même technique pour charger une autre table, cette fois la journalisation des erreurs susceptibles de se produire.

5. Remplacez le code SQL dans le volet de script par le code suivant, en remplaçant **datalake*xxxxxx*** par le nom de votre lac de données dans les clauses et les ```FROM``` ```ERRORFILE``` clauses :

    ```sql
    COPY INTO dbo.StageCustomer
    (GeographyKey, CustomerAlternateKey, Title, FirstName, MiddleName, LastName, NameStyle, BirthDate, 
    MaritalStatus, Suffix, Gender, EmailAddress, YearlyIncome, TotalChildren, NumberChildrenAtHome, EnglishEducation, 
    SpanishEducation, FrenchEducation, EnglishOccupation, SpanishOccupation, FrenchOccupation, HouseOwnerFlag, 
    NumberCarsOwned, AddressLine1, AddressLine2, Phone, DateFirstPurchase, CommuteDistance)
    FROM 'https://datalakexxxxxx.dfs.core.windows.net/files/data/Customer.csv'
    WITH
    (
    FILE_TYPE = 'CSV'
    ,MAXERRORS = 5
    ,FIRSTROW = 2 -- skip header row
    ,ERRORFILE = 'https://datalakexxxxxx.dfs.core.windows.net/files/'
    );
    ```

6. Exécutez le script et passez en revue le message résultant. Le fichier source contient une ligne avec des données non valides. Une ligne est donc rejetée. Le code ci-dessus spécifie un maximum de **5** erreurs. Par conséquent, une seule erreur n’a pas empêché le chargement des lignes valides. Vous pouvez afficher les lignes chargées ** en exécutant la requête suivante.

    ```sql
    SELECT *
    FROM dbo.StageCustomer
    ```

7. Sous l’onglet **Fichiers** , affichez le dossier racine de votre lac de données et vérifiez qu’un nouveau dossier nommé **_rejectedrows** a été créé (si vous ne voyez pas ce dossier, dans le **menu Plus** , sélectionnez **Actualiser** pour actualiser la vue).
8. Ouvrez le **dossier _rejectedrows** et le sous-dossier spécifique de date et d’heure qu’il contient, et notez que les fichiers portant des noms similaires à ***QID123_1_2*. Error.Txt** et ***QID123_1_2*. Row.Txt** a été créé. Vous pouvez cliquer avec le bouton droit sur chacun de ces fichiers et sélectionner **Aperçu** pour afficher les détails de l’erreur et la ligne qui a été rejetée.

    L’utilisation de tables intermédiaires vous permet de valider ou de transformer des données avant de les déplacer ou de les utiliser pour les ajouter ou les upsert dans toutes les tables de dimension existantes. L’instruction COPY fournit une technique simple mais hautes performances que vous pouvez utiliser pour charger facilement des données à partir de fichiers d’un lac de données dans des tables intermédiaires, et comme vous l’avez vu, identifier et rediriger des lignes non valides.

### Utilisation d’une instruction CREATE TABLE AS (CTAS)

1. Revenez au volet de script et remplacez le code qu’il contient par le code suivant :

    ```sql
    CREATE TABLE dbo.DimProduct
    WITH
    (
        DISTRIBUTION = HASH(ProductAltKey),
        CLUSTERED COLUMNSTORE INDEX
    )
    AS
    SELECT ROW_NUMBER() OVER(ORDER BY ProductID) AS ProductKey,
        ProductID AS ProductAltKey,
        ProductName,
        ProductCategory,
        Color,
        Size,
        ListPrice,
        Discontinued
    FROM dbo.StageProduct;
    ```

2. Exécutez le script, qui crée une table nommée **DimProduct**  à partir des données de produit intermédiaires qui utilisent **ProductAltKey** comme clé de distribution de hachage et a un index columnstore cluster.
4. Exécutez la requête suivante pour afficher le contenu de la table .

    ```sql
    SELECT ProductKey,
        ProductAltKey,
        ProductName,
        ProductCategory,
        Color,
        Size,
        ListPrice,
        Discontinued
    FROM dbo.DimProduct;
    ```

    L’expression CREATE TABLE AS SELECT (CTAS) a diverses utilisations, notamment :

    - Redistribuer la clé de hachage d’une table pour s’aligner sur d’autres tables pour améliorer les performances des requêtes.
    - Affectation d’une clé de substitution à une table intermédiaire basée sur des valeurs existantes après avoir effectué une analyse delta.
    - Création rapide de tables d’agrégation à des fins de rapport.

### Combiner des instructions INSERT et UPDATE pour charger une table de dimension à variation lente

La **table DimCustomer** prend en charge le type 1 et le type 2 les dimensions à variation lente (SCD), où le type 1 change entraîne une mise à jour sur place d’une ligne existante, et le type 2 modifie une nouvelle ligne pour indiquer la dernière version d’une instance d’entité de dimension particulière. Le chargement de cette table nécessite une combinaison d’instructions INSERT (pour charger de nouveaux clients) et d’instructions UPDATE (pour appliquer les modifications de type 1 ou de type 2).

1. Remplacez le code existant dans le fichier  par l’exemple de code suivant.

    ```sql
    INSERT INTO dbo.DimCustomer ([GeographyKey],[CustomerAlternateKey],[Title],[FirstName],[MiddleName],[LastName],[NameStyle],[BirthDate],[MaritalStatus],
    [Suffix],[Gender],[EmailAddress],[YearlyIncome],[TotalChildren],[NumberChildrenAtHome],[EnglishEducation],[SpanishEducation],[FrenchEducation],
    [EnglishOccupation],[SpanishOccupation],[FrenchOccupation],[HouseOwnerFlag],[NumberCarsOwned],[AddressLine1],[AddressLine2],[Phone],
    [DateFirstPurchase],[CommuteDistance])
    SELECT *
    FROM dbo.StageCustomer AS stg
    WHERE NOT EXISTS
        (SELECT * FROM dbo.DimCustomer AS dim
        WHERE dim.CustomerAlternateKey = stg.CustomerAlternateKey);

    -- Type 1 updates (change name, email, or phone in place)
    UPDATE dbo.DimCustomer
    SET LastName = stg.LastName,
        EmailAddress = stg.EmailAddress,
        Phone = stg.Phone
    FROM DimCustomer dim inner join StageCustomer stg
    ON dim.CustomerAlternateKey = stg.CustomerAlternateKey
    WHERE dim.LastName <> stg.LastName OR dim.EmailAddress <> stg.EmailAddress OR dim.Phone <> stg.Phone

    -- Type 2 updates (address changes triggers new entry)
    INSERT INTO dbo.DimCustomer
    SELECT stg.GeographyKey,stg.CustomerAlternateKey,stg.Title,stg.FirstName,stg.MiddleName,stg.LastName,stg.NameStyle,stg.BirthDate,stg.MaritalStatus,
    stg.Suffix,stg.Gender,stg.EmailAddress,stg.YearlyIncome,stg.TotalChildren,stg.NumberChildrenAtHome,stg.EnglishEducation,stg.SpanishEducation,stg.FrenchEducation,
    stg.EnglishOccupation,stg.SpanishOccupation,stg.FrenchOccupation,stg.HouseOwnerFlag,stg.NumberCarsOwned,stg.AddressLine1,stg.AddressLine2,stg.Phone,
    stg.DateFirstPurchase,stg.CommuteDistance
    FROM dbo.StageCustomer AS stg
    JOIN dbo.DimCustomer AS dim
    ON stg.CustomerAlternateKey = dim.CustomerAlternateKey
    AND stg.AddressLine1 <> dim.AddressLine1;
    ```

2. Exécuter le programme et examiner la sortie

## Effectuer l’optimisation postchargement

Une fois les nouvelles données chargées dans l’entrepôt de données, il est recommandé de recréer les index des tables, et de mettre à jour les statistiques pour les colonnes fréquemment interrogées.

1. Remplacez le code ci-dessus dans le bloc  par le code suivant :

    ```sql
    ALTER INDEX ALL ON dbo.DimProduct REBUILD;
    ```

2. Exécutez le script pour reconstruire les index sur la **table DimProduct** .
3. Remplacez le code ci-dessus dans le bloc  par le code suivant :

    ```sql
    CREATE STATISTICS customergeo_stats
    ON dbo.DimCustomer (GeographyKey);
    ```

4. Exécutez le script pour créer ou mettre à jour des statistiques sur la **colonne GeographyKey** de la **table DimCustomer** .

## Supprimer les ressources Azure

Si vous avez fini d’explorer Azure Synapse Analytics, vous devriez supprimer les ressources que vous avez créées afin d’éviter des coûts Azure inutiles.

1. Fermez l’onglet du navigateur Synapse Studio et revenez dans le portail Azure.
2. Dans le portail Azure, dans la page **Accueil**, sélectionnez **Groupes de ressources**.
3. Sélectionnez le groupe de ressources pour votre espace de travail Synapse Analytics (et non le groupe de ressources managé) et vérifiez qu’il contient l’espace de travail Synapse, le compte de stockage et le pool Spark pour votre espace de travail.
4. Au sommet de la page **Vue d’ensemble** de votre groupe de ressources, sélectionnez **Supprimer le groupe de ressources**.
5. Entrez le nom du groupe de ressources pour confirmer que vous souhaitez le supprimer, puis sélectionnez Supprimer.

    Après quelques minutes, votre espace de travail Azure Synapse et l’espace de travail managé qui lui est associé seront supprimés.
