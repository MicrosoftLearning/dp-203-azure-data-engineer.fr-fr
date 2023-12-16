---
lab:
  title: Explorer Azure Databricks
  ilt-use: Suggested demo
---

# Explorer Azure Databricks

Azure Databricks est une version basée sur Microsoft Azure de la plateforme Databricks open source populaire.

De même qu’Azure Synapse Analytics, un *espace de travail* Azure Databricks permet la gestion des clusters, des données et des ressources Databricks sur Azure dans un emplacement unique.

Cet exercice devrait prendre environ **30** minutes.

## Avant de commencer

Vous avez besoin d’un [abonnement Azure](https://azure.microsoft.com/free) dans lequel vous avez un accès administratif.

## Configurer un espace de travail Azure Databricks.

Dans cet exercice, vous allez utiliser un script pour configurer un nouvel espace de travail Azure Databricks.

> **Conseil** : si vous disposez déjà d’un espace de travail Azure Databricks *standard* ou d’*essai*, vous pouvez ignorer cette procédure et utiliser votre espace de travail existant.

1. Dans un navigateur web, connectez-vous au [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com`.
2. Utilisez le bouton **[\>_]** à droite de la barre de recherche, en haut de la page, pour créer un environnement Cloud Shell dans le portail Azure, puis sélectionnez un environnement ***PowerShell*** et créez le stockage si vous y êtes invité. Cloud Shell fournit une interface de ligne de commande dans un volet situé en bas du portail Azure, comme illustré ici :

    ![Portail Azure avec un volet Cloud Shell](./images/cloud-shell.png)

    > **Remarque** : si vous avez créé une instance Cloud Shell qui utilise un environnement *Bash*, utilisez le menu déroulant en haut à gauche du volet Cloud Shell pour le remplacer par ***PowerShell***.

3. Notez que vous pouvez redimensionner le volet Cloud Shell en faisant glisser la barre de séparation en haut du volet. Vous pouvez aussi utiliser les icônes **&#8212;** , **&#9723;** et **X** situées en haut à droite du volet pour réduire, agrandir et fermer le volet. Pour plus d’informations sur l’utilisation d’Azure Cloud Shell, consultez la [documentation Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. Dans le volet PowerShell, entrez les commandes suivantes pour cloner ce référentiel :

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Une fois que le référentiel a été cloné, entrez les commandes suivantes pour accéder au dossier de ce labo et exécutez le script **setup.ps1** qu’il contient :

    ```
    cd dp-203/Allfiles/labs/23
    ./setup.ps1
    ```

6. Si vous y êtes invité, choisissez l’abonnement que vous souhaitez utiliser (cela se produit uniquement si vous avez accès à plusieurs abonnements Azure).

7. Attendez que le script se termine. Cette opération prend généralement environ 5 minutes, mais dans certains cas, elle peut être plus longue. Pendant que vous patientez, consultez l’article [Présentation d’Azure Databricks](https://learn.microsoft.com/azure/databricks/introduction/) dans la documentation d’Azure Databricks.

## Créer un cluster

Azure Databricks est une plateforme de traitement distribuée qui utilise des *clusters* Apache Spark pour traiter des données en parallèle sur plusieurs nœuds. Chaque cluster se compose d’un nœud de pilote pour coordonner le travail et les nœuds Worker pour effectuer des tâches de traitement.

Dans cet exercice, vous allez créer un cluster à *nœud unique* pour réduire les ressources de calcul utilisées dans l’environnement du labo (dans lequel les ressources peuvent être limitées). Dans un environnement de production, vous créez généralement un cluster avec plusieurs nœuds Worker.

> **Conseil** : si vous disposez déjà d’un cluster avec une version du runtime 13.3 LTS dans votre espace de travail Azure Databricks, vous pouvez l’utiliser pour effectuer cet exercice et ignorer cette procédure.

1. Dans le portail Azure, accédez au groupe de ressources **dp203-*xxxxxxx*** créé par le script (ou le groupe de ressources contenant votre espace de travail Azure Databricks existant)
1. Sélectionnez votre ressource de service Azure Databricks (nommée **databricks*xxxxxxx*** si vous avez utilisé le script d’installation pour la créer).
1. Dans la page **Vue d’ensemble** de votre espace de travail, utilisez le bouton **Lancer l’espace de travail** pour ouvrir votre espace de travail Azure Databricks dans un nouvel onglet de navigateur et connectez-vous si vous y êtes invité.

    > **Conseil** : lorsque vous utilisez le portail de l’espace de travail Databricks, plusieurs conseils et notifications peuvent s’afficher. Ignorez-les et suivez les instructions fournies pour effectuer les tâches de cet exercice.

1. Affichez le portail de l’espace de travail Azure Databricks et notez que la barre latérale de gauche contient des liens pour les différents types de tâches que vous pouvez effectuer.

1. Sélectionnez le lien **(+) Nouveau** dans la barre latérale, puis **Cluster**.
1. Dans la page **Nouveau cluster**, créez un cluster assorti des paramètres suivants :
    - **Nom du cluster** : cluster du *nom d’utilisateur* (nom de cluster par défaut)
    - **Mode cluster** : nœud unique
    - **Mode d’accès ** : utilisateur unique (*avec votre compte d’utilisateur sélectionné*)
    - **Version du runtime Databricks** : 13.3 LTS (Spark 3.4.1, Scala 2.12)
    - **Utiliser l’accélération photon** : sélectionné
    - **Type de nœud** : Standard_DS3_v2
    - **Terminer après** *30* **minutes d’inactivité**

1. Attendez que le cluster soit créé. Cette opération peut prendre une à deux minutes.

> **Remarque** : si votre cluster ne démarre pas, il se peut que le quota de votre abonnement soit insuffisant dans la région où votre espace de travail Azure Databricks est configuré. Consultez l’article [La limite de cœurs du processeur empêche la création du cluster](https://docs.microsoft.com/azure/databricks/kb/clusters/azure-core-limit) pour plus de détails. Si cela se produit, vous pouvez essayer de supprimer votre espace de travail et d’en créer un autre dans une région différente. Vous pouvez spécifier une région comme paramètre pour le script d’installation comme suit : `./setup.ps1 eastus`

## Utiliser Spark pour analyser un fichier de données

Comme dans de nombreux environnements Spark, Databricks prend en charge l’utilisation de notebooks pour combiner des notes et des cellules de code interactives que vous pouvez utiliser pour explorer les données.

1. Dans la barre latérale, cliquez sur le lien **(+) Nouveau** pour créer un **notebook**.
1. Remplacez le nom du notebook par défaut (**Untitled Notebook *[date]***) par **Explorer les produits** et dans la liste déroulante **Connecter**, sélectionnez votre cluster si ce n’est pas déjà le cas. Si le cluster n’est pas en cours d’exécution, le démarrage peut prendre une minute.
1. Téléchargez le fichier [**products.csv**](https://raw.githubusercontent.com/MicrosoftLearning/dp-203-azure-data-engineer/master/Allfiles/labs/23/adventureworks/products.csv) sur votre ordinateur local et enregistrez-le en tant que **products.csv**. Ensuite, dans le notebook **Explorer les produits**, à partir du menu **Fichier**, sélectionnez **Charger des données dans DBFS**.
1. Dans la boîte de dialogue **Charger des données**, notez le **répertoire cible DBFS** dans lequel le fichier sera chargé. Sélectionnez ensuite la zone **Fichiers**, puis chargez le fichier **products.csv** que vous avez téléchargé sur votre ordinateur. Une fois le fichier chargé, sélectionnez **Suivant**.
1. Dans le volet **Accéder aux fichiers à partir des notebooks**, sélectionnez l’exemple de code PySpark et copiez-le dans le presse-papiers. Vous l’utiliserez pour charger les données à partir du fichier dans un DataFrame. Ensuite, sélectionnez **Terminé**.
1. Dans le notebook **Explorer les produits**, dans la cellule de code vide, collez le code que vous avez copié, qui doit ressembler à ceci :

    ```python
    df1 = spark.read.format("csv").option("header", "true").load("dbfs:/FileStore/shared_uploads/user@outlook.com/products.csv")
    ```

1. Sélectionnez l’option de menu **▸ Exécuter la cellule** en haut à droite de la cellule pour l’exécuter, en démarrant et en attachant le cluster si vous y êtes invité.
1. Attendez que l’exécution de la tâche Spark par le code soit terminée. Le code a créé un objet *dataframe* nommé **df1** à partir des données du fichier que vous avez chargé.
1. Sous la cellule de code existante, sélectionnez l’icône **+** pour ajouter une nouvelle cellule de code. Dans la nouvelle cellule, entrez ensuite le code suivant :

    ```python
    display(df1)
    ```

1. Utilisez l’option de menu **▸ Exécuter la cellule** en haut à droite de la nouvelle cellule pour l’exécuter. Ce code affiche le contenu du dataframe, qui doit ressembler à ceci :

    | ProductID | ProductName | Catégorie | ListPrice |
    | -- | -- | -- | -- |
    | 771 | Mountain-100 Silver, 38 | VTT | 3399.9900 |
    | 772 | Mountain-100 Silver, 42 | VTT | 3399.9900 |
    | … | … | … | … |

1. Au-dessus du tableau des résultats, sélectionnez **+**, puis **Visualisation** pour afficher l’éditeur de visualisation et appliquer les options suivantes :
    - **Type de visualisation** : barre
    - **Colonne X** : catégorie
    - **Colonne Y** : *ajoutez une nouvelle colonne et sélectionnez* **ProductID**. *Appliquez l’**agrégation* **Count**.

    Enregistrez la visualisation et observez qu’elle s’affiche dans le notebook comme suit :

    ![Graphique à barres montrant les quantités de produits par catégorie](./images/databricks-chart.png)

## Créer et interroger une table

Bien que de nombreuses analyses de données utilisent des langages tels que Python ou Scala pour traiter les données contenues dans des fichiers, de nombreuses solutions d’analytique données reposent sur des bases de données relationnelles, dans lesquelles les données sont stockées dans des tables et manipulées à l’aide de SQL.

1. Dans le notebook **Explorer les produits**, sous la sortie de graphique de la cellule de code précédemment exécutée, sélectionnez l’icône **+** pour ajouter une nouvelle cellule.
2. Entrez et exécutez le code suivant dans la nouvelle cellule :

    ```python
    df1.write.saveAsTable("products")
    ```

3. Une fois la cellule terminée, ajoutez une nouvelle cellule sous celle-ci avec le code suivant :

    ```sql
    %sql

    SELECT ProductName, ListPrice
    FROM products
    WHERE Category = 'Touring Bikes';
    ```

4. Exécutez la nouvelle cellule, qui contient du code SQL pour retourner le nom et le prix des produits dans la catégorie *Touring Bikes*.
5. Dans la barre latérale, sélectionnez le lien **Catalogue** et vérifiez que la table **produits** a été créée dans le schéma de base de données par défaut (dont le nom est évidemment **default**). Il est possible d’utiliser du code Spark pour créer des schémas de base de données personnalisés et un schéma de tables relationnelles que les analystes de données peuvent utiliser pour explorer les données et générer des rapports analytiques.

## Supprimer les ressources Azure Databricks

Maintenant que vous avez terminé d’explorer Azure Databricks, vous devez supprimer les ressources que vous avez créées pour éviter les coûts Azure inutiles et libérer de la capacité dans votre abonnement.

1. Fermez l’onglet de l’espace de travail Azure Databricks dans votre navigateur et retournez dans le portail Azure.
2. Dans le portail Azure, dans la page **Accueil**, sélectionnez **Groupes de ressources**.
3. Sélectionnez le groupe de ressources **dp203-*xxxxxxx*** (et non le groupe de ressources managé) et vérifiez qu’il contient votre espace de travail Azure Databricks.
4. Au sommet de la page **Vue d’ensemble** de votre groupe de ressources, sélectionnez **Supprimer le groupe de ressources**.
5. Entrez le nom du groupe de ressources **dp203-*xxxxxxx*** pour confirmer que vous souhaitez le supprimer, puis sélectionnez **Supprimer**.

    Après quelques minutes, votre groupe de ressources et les groupes de ressources de l’espace de travail managés qui lui sont associés seront supprimés.
