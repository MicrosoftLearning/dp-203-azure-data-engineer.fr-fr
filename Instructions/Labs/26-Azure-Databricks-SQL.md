---
lab:
  title: Exercice - Utiliser un entrepôt SQL dans Azure Databricks
  ilt-use: Optional demo
---

# Exercice - Utiliser un entrepôt SQL dans Azure Databricks

SQL est un langage standard pour interroger et manipuler des données. De nombreux analystes de données effectuent des analyses de données à l’aide de SQL pour interroger des tables dans une base de données relationnelle. Azure Databricks inclut des fonctionnalités SQL qui s’appuient sur les technologies Spark et Delta Lake pour fournir une couche de base de données relationnelle sur des fichiers dans un lac de données.

Cet exercice devrait prendre environ **30** minutes.

## Avant de commencer

Vous aurez besoin d’un abonnement[ Azure dans lequel vous disposez d’un ](https://azure.microsoft.com/free)accès au niveau administratif et d’un quota suffisant dans au moins une région pour approvisionner un entrepôt SQL Azure Databricks.

## Provisionner un espace de travail Azure Databricks.

Dans cet exercice, vous aurez besoin d’un espace de travail Azure Databricks de niveau Premium.

1. Dans un navigateur, connectez-vous au portail Azure.
2. Utilisez le bouton **[\>_]** à droite de la barre de recherche, en haut de la page, pour créer un environnement Cloud Shell dans le portail Azure, en sélectionnant un environnement ***Bash*** et en créant le stockage si vous y êtes invité. Cloud Shell fournit une interface de ligne de commande dans un volet situé en bas du portail Azure, comme illustré ici :

    ![Portail Azure avec un volet Cloud Shell](./images/cloud-shell.png)

    > **Remarque** : Si vous avez créé un interpréteur de commandes cloud qui utilise un *environnement Bash* , utilisez le menu déroulant en haut à gauche du volet Cloud Shell pour le remplacer par ***PowerShell***.

3. Notez que vous pouvez redimensionner le volet Cloud Shell en faisant glisser la barre de séparation en haut du volet. Vous pouvez aussi utiliser les icônes **&#8212;** , **&#9723;** et **X** situées en haut à droite du volet pour réduire, agrandir et fermer le volet. Pour plus d’informations sur l’utilisation d’Azure Cloud Shell, consultez la [documentation Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. Dans le terminal, entrez les commandes suivantes pour cloner ce dépôt :

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Une fois que le dépôt a été cloné, entrez les commandes suivantes pour accéder au dossier de ce labo et exécutez le script **setup.sh** qu’il contient :

    ```
    cd dp-203/Allfiles/labs/26
    ./setup.ps1
    ```

6. Si vous y êtes invité, choisissez l’abonnement que vous souhaitez utiliser (cela se produit uniquement si vous avez accès à plusieurs abonnements Azure).

7. Attendez que le script se termine, ce qui prend généralement entre 5 et 10 minutes. Pendant que vous attendez, consultez l’article Qu’est-ce que l’entreposage [de données sur Azure Databricks ?](https://learn.microsoft.com/azure/databricks/sql/) dans la documentation Azure Databricks.

## Afficher et démarrer un SQL Warehouse

1. Une fois la ressource d’espace de travail Azure Databricks déployée, accédez-y dans le Portail Azure.
2. Dans la **page Vue d’ensemble** de votre espace de travail Azure Databricks, utilisez le bouton Lancer l’espace **de** travail pour ouvrir votre espace de travail Azure Databricks dans un nouvel onglet de navigateur ; connectez-vous si vous y êtes invité.
3. Si un **message De vos données actuelles s’affiche,** sélectionnez **Terminer** pour le fermer. Affichez ensuite le portail de l’espace de travail Azure Databricks et notez que la barre latérale du côté gauche contient les noms des catégories de tâches.

    >**Conseil** : Lorsque vous utilisez le portail Databricks Workspace, différents conseils et notifications peuvent être affichés. Ignorez ces instructions et suivez les instructions fournies pour effectuer les tâches de cet exercice.

1. Dans la barre latérale, sous **SQL**, sélectionnez **SQL Warehouses**.
1. Notez que l’espace de travail inclut déjà un entrepôt SQL Warehouse nommé **Starter Warehouse**.
1. Dans le **menu Actions** (**&#8285 ;**) de SQL Warehouse, sélectionnez **Modifier**. Définissez ensuite la propriété **taille** du **cluster sur 2X-Small** et enregistrez vos modifications.
1. Utilisez le **bouton Démarrer** pour démarrer SQL Warehouse (qui peut prendre une minute ou deux).

> **Remarque** : Si votre sql Warehouse ne démarre pas, votre abonnement peut avoir un quota insuffisant dans la région où votre espace de travail Azure Databricks est provisionné. Pour plus d’informations, consultez [Quota de processeurs virtuels Azure requis](https://docs.microsoft.com/azure/databricks/sql/admin/sql-endpoints#required-azure-vcpu-quota). Si cela se produit, vous pouvez essayer de demander une augmentation de quota comme indiqué dans le message d’erreur lorsque l’entrepôt ne parvient pas à démarrer. Vous pouvez également essayer de supprimer votre espace de travail et de en créer un dans une autre région. Vous pouvez spécifier une région comme paramètre pour le script d’installation comme suit : `./setup.ps1 eastus`

## Créer un schéma de base de données

1. Lorsque votre sql Warehouse est en cours d’exécution**, sélectionnez **l’Éditeur** SQL dans la barre latérale.
2. Dans le **volet Du navigateur** de schémas, observez que le *catalogue hive_metastore* contient une base de données nommée **par défaut**.
3. Dans le volet **Requête 1**, entrez le code SQL suivant :

    ```sql
    CREATE SCHEMA adventureworks;
    ```

4. Utiliser le **&#9658 ; Bouton Exécuter (1000)** pour exécuter le code SQL.
5. Lorsque le code a été correctement exécuté, dans le **volet du navigateur** de schéma, utilisez le bouton d’actualisation en bas du volet pour actualiser la liste. Développez **ensuite hive_metastore** et **adventureworks**, puis observez que la base de données a été créée, mais ne contient aucune table.

Vous pouvez utiliser la **base de données par défaut** pour vos tables, mais lors de la création d’un magasin de données analytiques, il est préférable de créer des bases de données personnalisées pour des données spécifiques.

## Créer une table

1. Téléchargez le [**fichier products.csv**](https://raw.githubusercontent.com/MicrosoftLearning/dp-203-azure-data-engineer/master/Allfiles/labs/26/data/products.csv) sur votre ordinateur local, en l’enregistrant en tant que **products.csv**.
1. Dans le portail de l’espace de travail Azure Databricks, dans la barre latérale, sélectionnez **(+) Nouveau**, puis téléchargez **** le **fichier products.csv** que vous avez téléchargé sur votre ordinateur.
1. Dans la **page Charger des données** , sélectionnez le **schéma adventureworks** et définissez le nom de la table sur **les produits**. **Sélectionnez Ensuite Créer un tableau** dans le coin inférieur gauche de la page.
1. Une fois la table créée, passez en revue ses détails.

La possibilité de créer une table en important des données à partir d’un fichier facilite la remplissage d’une base de données. Vous pouvez également utiliser Spark SQL pour créer des tables à l’aide du code. Les tables elles-mêmes sont des définitions de métadonnées dans le metastore hive, et les données qu’elles contiennent sont stockées au format Delta dans le stockage DBFS (Databricks File System).

## Créer une requête

1. Cliquez sur **New** (Nouveau) dans la barre latérale et sélectionnez **Query** (Requête).
2. Dans le **volet du navigateur** schéma, développez **hive_metastore** et **adventureworks**, puis vérifiez que la **table des produits** est répertoriée.
3. Dans le volet **Requête 1**, entrez le code SQL suivant :

    ```sql
    SELECT ProductID, ProductName, Category
    FROM adventureworks.products; 
    ```

4. Utiliser le **&#9658 ; Bouton Exécuter (1000)** pour exécuter le code SQL.
5. Une fois la requête terminée, passez en revue la table des résultats.
6. Utilisez le **bouton Enregistrer** en haut à droite de l’éditeur de requête pour enregistrer la requête en tant que **produits et catégories**.

L’enregistrement d’une requête facilite la récupération des mêmes données ultérieurement.

## Créer un tableau de bord

1. Cliquez sur **New** (Nouveau) dans la barre latérale et sélectionnez **Dashboard** (Tableau de bord).
2. Dans la **boîte de dialogue Nouveau tableau de bord** , entrez le nom **Adventure Works Products** , puis sélectionnez **Enregistrer**.
3. Dans le **tableau de bord Adventure Works Products** , dans la **liste déroulante Ajouter** , sélectionnez **Visualisation**.
4. Dans la boîte de dialogue Ajouter un **widget** de visualisation, sélectionnez la **requête Produits et Catégories** . **Sélectionnez Ensuite Créer une visualisation**, définissez le titre sur **Produits par catégorie**. et sélectionnez **Créer une visualisation**.
5. Dans l’éditeur de visualisation, définissez les propriétés suivantes :
    - **Type de visualisation carte**
    - **Graphique** horizontal : sélectionné
    - Catégorie de colonne
    - **Colonnes** X : ID de produit : Nombre
    - Grouper par : Catégorie.
    - **Placement** de légende : Automatique (flexible)
    - **Ordre** des éléments de légende : Normal
    - **Empilement** : pile
    - **Normaliser les valeurs en pourcentage** : <u>Non</u>sélectionné
    - **Valeurs** manquantes et NULL : Ne pas afficher dans le graphique

6. Enregistrez la visualisation et affichez-la dans le tableau de bord.
7. Sélectionnez **Terminé la modification** pour afficher le tableau de bord en tant qu’utilisateurs le verra.

Les tableaux de bord constituent un excellent moyen de partager des tables de données et des visualisations avec des utilisateurs professionnels. Vous pouvez planifier l’actualisation périodique des tableaux de bord et envoyer des e-mails aux abonnés.

## Ressources Azure Databricks

Maintenant que vous avez terminé d’explorer SQL Warehouses dans Azure Databricks, vous devez supprimer les ressources que vous avez créées pour éviter les coûts Azure inutiles et libérer de la capacité dans votre abonnement.

1. Fermez l’onglet du navigateur et retournez sous l’onglet Portail Azure.
2. Dans le portail Azure, dans la page **Accueil**, sélectionnez **Groupes de ressources**.
3. Sélectionnez le groupe de ressources contenant votre espace de travail Azure Databricks (et non le groupe de ressources managé).
4. Au sommet de la page **Vue d’ensemble** de votre groupe de ressources, sélectionnez **Supprimer le groupe de ressources**.
5. Entrez le nom du groupe de ressources pour confirmer que vous souhaitez le supprimer, puis sélectionnez **Supprimer**.

    Après quelques minutes, votre espace de travail Azure Synapse et l’espace de travail managé qui lui est associé seront supprimés.
