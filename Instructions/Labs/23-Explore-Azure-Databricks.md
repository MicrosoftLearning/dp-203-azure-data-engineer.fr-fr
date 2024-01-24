---
lab:
  title: Explorer Azure Databricks
  ilt-use: Suggested demo
---

# Explorer Azure Databricks

Azure Databricks est une version basée sur Microsoft Azure de la plateforme Databricks open source populaire.

De même qu’Azure Synapse Analytics, un espace de travail* Azure Databricks *fournit un point central pour la gestion des clusters, des données et des ressources Databricks sur Azure.

Cet exercice devrait prendre environ **30** minutes.

## Avant de commencer

Vous avez besoin d’un [abonnement Azure](https://azure.microsoft.com/free) dans lequel vous avez un accès administratif.

## Provisionner un espace de travail Azure Databricks.

Dans cet exercice, vous allez utiliser un script pour provisionner un nouvel espace de travail Azure Databricks.

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
    cd dp-203/Allfiles/labs/23
    ./setup.ps1
    ```

6. Si vous y êtes invité, choisissez l’abonnement que vous souhaitez utiliser (cela se produit uniquement si vous avez accès à plusieurs abonnements Azure).

7. Attendez que le script se termine, ce qui prend généralement entre 5 et 10 minutes. Pendant que vous attendez, consultez l’article [Qu’est-ce qu’Azure Databricks ?](https://learn.microsoft.com/azure/databricks/introduction/) dans la documentation Azure Databricks.

## Créer un cluster

Azure Databricks est une plateforme de traitement distribuée qui utilise des clusters* Apache Spark *pour traiter des données en parallèle sur plusieurs nœuds. Chaque cluster se compose d’un nœud de pilote pour coordonner le travail et les nœuds Worker pour effectuer des tâches de traitement.

> **Remarque** : dans cet exercice, vous allez créer un *cluster à nœud* unique pour réduire les ressources de calcul utilisées dans l’environnement lab (dans lequel les ressources peuvent être limitées). Dans un environnement de production, vous créez généralement un cluster avec plusieurs nœuds Worker.

1. Dans le Portail Azure, accédez au **groupe de ressources dp203-*xxxxxxx*** créé par le script que vous avez exécuté.
2. Sélectionnez la **ressource Databricks*xxxxxxx*** Azure Databricks Service.
3. Dans la **page Vue d’ensemble** de **databricks*xxxxxxx***, utilisez le bouton Lancer l’espace **de** travail pour ouvrir votre espace de travail Azure Databricks dans un nouvel onglet de navigateur ; connectez-vous si vous y êtes invité.
4. Si un **message De vos données actuelles s’affiche,** sélectionnez **Terminer** pour le fermer. Affichez ensuite le portail de l’espace de travail Azure Databricks et notez que la barre latérale du côté gauche contient des icônes pour les différentes tâches que vous pouvez effectuer.

    >**Conseil** : Lorsque vous utilisez le portail Databricks Workspace, différents conseils et notifications peuvent être affichés. Ignorez ces instructions et suivez les instructions fournies pour effectuer les tâches de cet exercice.

1. Sélectionnez la **nouvelle tâche (+),** puis sélectionnez **Cluster**.
1. Dans la **page Nouveau cluster** , créez un cluster avec les paramètres suivants :
    - **Nom du cluster : *cluster du nom** d’utilisateur (nom de* cluster par défaut)
    - **Mode de cluster à nœud unique**
    - **Mode** d’accès : utilisateur unique (*avec votre compte d’utilisateur sélectionné*)
    - Définissez Databricks runtime version sur Runtime: 11.3 LTS (Scala 2.12, Spark 3.3.0) ou version ultérieure.
    - **Utiliser l’accélération** photon : sélectionné
    - Sélectionnez Type de nœud : Standard_DS3_v2.
    - Terminer après 120 minutes d’inactivité

7. Attendez que le compte de stockage soit créé. Cela peut prendre une à deux minutes.

> **Remarque** : Si votre cluster ne démarre pas, votre abonnement peut avoir un quota insuffisant dans la région où votre espace de travail Azure Databricks est provisionné. Pour obtenir de l’aide, consultez [La limite de cœurs du processeur empêche la création du cluster](https://docs.microsoft.com/azure/databricks/kb/clusters/azure-core-limit). Si cela se produit, vous pouvez essayer de supprimer votre espace de travail et de en créer un dans une autre région. Vous pouvez spécifier une région comme paramètre pour le script d’installation comme suit : `./setup.ps1 eastus`

## Utiliser un pool Spark pour analyser des données

Comme dans de nombreux environnements Spark, Databricks prend en charge l’utilisation de notebooks pour combiner des notes et des cellules de code interactives que vous pouvez utiliser pour explorer les données.

1. Dans la barre latérale, utilisez la **tâche (+) Nouvelle** tâche pour créer un **bloc-notes**.
1. Modifiez le nom du bloc-notes** par défaut (**Bloc-notes sans titre *[date]***) pour **explorer les produits** et dans la **liste déroulante Connecter, sélectionnez votre cluster (qui peut prendre une minute ou plus pour démarrer).
1. Téléchargez le [**fichier products.csv**](https://raw.githubusercontent.com/MicrosoftLearning/dp-203-azure-data-engineer/master/Allfiles/labs/23/adventureworks/products.csv) sur votre ordinateur local, en l’enregistrant en tant que **products.csv**. Ensuite, dans le bloc-notes Explorer les **produits** , dans le **menu Fichier** , sélectionnez **Charger des données dans DBFS**.
1. Dans la **boîte de dialogue Charger des données** , notez le **répertoire** cible DBFS dans lequel le fichier sera chargé. Sélectionnez ensuite la **zone Fichiers** , puis chargez le **fichier products.csv** que vous avez téléchargé sur votre ordinateur. Une fois le fichier chargé, sélectionnez **Suivant**
1. Dans le **volet Fichiers Access du volet notebooks** , sélectionnez l’exemple de code PySpark et copiez-le dans le Presse-papiers. Vous l’utiliserez pour charger les données du fichier dans un DataFrame. Ensuite, sélectionnez **Terminé**.
1. Dans le **bloc-notes Explorer les produits** , dans la cellule de code vide, collez le code que vous avez copié ; qui doit ressembler à ceci :

    ```python
    df1 = spark.read.format("csv").option("header", "true").load("dbfs:/FileStore/shared_uploads/user@outlook.com/products.csv")
    ```

1. Utilisez le **fichier &#9656 ; Exécutez l’option de menu Cellule** en haut à droite de la cellule pour l’exécuter, en commençant et en attachant le cluster si vous y êtes invité.
1. Attendez que le travail Spark s’exécute par le code. Le code a créé un *objet dataframe* nommé **df1** à partir des données du fichier que vous avez chargé.
1. Sous la cellule de code existante, utilisez l’icône **+** pour ajouter une nouvelle cellule de code. Dans la nouvelle cellule, entrez le code suivant.

    ```python
    display(df1)
    ```

1. Utilisez le **fichier &#9656 ; Exécutez l’option de menu Cellule** en haut à droite de la nouvelle cellule pour l’exécuter. Ce code affiche le contenu du dataframe, qui doit ressembler à ceci :

    | ProductID | ProductName | Catégorie | ListPrice |
    | -- | -- | -- | -- |
    | 771 | Mountain-100 Silver, 38 | VTT | 3399.9900 |
    | 772 | Mountain-100 Silver, 42 | VTT | 3399.9900 |
    | ... | ... | ... | ... |

1. Au-dessus du tableau des résultats, sélectionnez ****+** Visualisation** pour afficher l’éditeur de visualisation, puis appliquez les options suivantes :
    - **Type de visualisation carte**
    - Catégorie de colonne
    - **Colonne** Y : *Ajoutez une nouvelle colonne et sélectionnez* **ProductID**. *Appliquez l’agrégation* ***Count.** *

    Enregistrez la visualisation et observez qu’elle est affichée dans le bloc-notes, comme suit :

    ![Graphique à barres montrant les quantités de produits par catégorie.](./images/databricks-chart.png)

## Étape 6 : Création et interrogation d’une table

Bien que de nombreuses analyses de données soient confortables à l’aide de langages tels que Python ou Scala pour travailler avec des données dans des fichiers, de nombreuses solutions d’analytique des données sont basées sur des bases de données relationnelles ; dans laquelle les données sont stockées dans des tables et manipulées à l’aide de SQL.

1. Dans le **bloc-notes Explorer les produits** , sous la sortie du graphique de la cellule de code précédemment exécutée, utilisez l’icône **+** pour ajouter une nouvelle cellule.
2. Entrez ensuite le code suivant dans la nouvelle cellule et exécutez-le :

    ```python
    df1.write.saveAsTable("products")
    ```

3. Une fois la cellule terminée, ajoutez une nouvelle cellule sous celle-ci avec le code suivant :

    ```sql
    %sql

    SELECT ProductName, ListPrice
    FROM products
    WHERE Category = 'Touring Bikes';
    ```

4. Exécutez la nouvelle cellule, qui contient du code SQL pour retourner le nom et le prix des produits dans la *catégorie Touring Bikes* .
5. Dans l’onglet de gauche, sélectionnez la **tâche Catalogue** et vérifiez que la **table des produits** a été créée dans le schéma de base de données par défaut (qui n’est pas nommé **par défaut**). Il est possible d’utiliser du code Spark pour créer des schémas de base de données personnalisés et un schéma de tables relationnelles que les analystes de données peuvent utiliser pour explorer les données et générer des rapports analytiques.

## Ressources Azure Databricks

Maintenant que vous avez terminé d’explorer Azure Databricks, vous devez supprimer les ressources que vous avez créées pour éviter les coûts Azure inutiles et libérer de la capacité dans votre abonnement.

1. Fermez l’onglet du navigateur et retournez sous l’onglet Portail Azure.
2. Dans le portail Azure, dans la page **Accueil**, sélectionnez **Groupes de ressources**.
3. Sélectionnez le **groupe de ressources dp203-*xxxxxxx*** (et non le groupe de ressources managés) et vérifiez qu’il contient votre espace de travail Azure Databricks.
4. Au sommet de la page **Vue d’ensemble** de votre groupe de ressources, sélectionnez **Supprimer le groupe de ressources**.
5. Entrez le nom du groupe de ressources pour confirmer que vous souhaitez le supprimer, puis sélectionnez Supprimer.

    Après quelques minutes, votre espace de travail Azure Synapse et l’espace de travail managé qui lui est associé seront supprimés.
