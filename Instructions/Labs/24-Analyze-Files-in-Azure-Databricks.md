---
lab:
  title: Utiliser Spark dans Azure Databricks
  ilt-use: Lab
---

# Utiliser Spark dans Azure Databricks

Azure Databricks est une version basée sur Microsoft Azure de la plateforme Databricks open source reconnue. Azure Databricks repose sur Apache Spark et offre une solution hautement évolutive pour les tâches d’ingénierie et d’analyse des données qui impliquent l’utilisation de données dans des fichiers. L’un des avantages de Spark est la prise en charge d’un large éventail de langages de programmation, notamment Java, Scala, Python et SQL. Cela fait de Spark une solution très flexible pour les charges de travail de traitement des données, y compris le nettoyage et la manipulation des données, l’analyse statistique et le Machine Learning, ainsi que l’analytique données et la visualisation des données.

Cet exercice devrait prendre environ **45** minutes.

## Avant de commencer

Vous avez besoin d’un [abonnement Azure](https://azure.microsoft.com/free) dans lequel vous avez un accès administratif.

## Provisionner un espace de travail Azure Databricks

Dans cet exercice, vous allez utiliser un script pour configurer un nouvel espace de travail Azure Databricks.

> **Conseil** : si vous disposez déjà d’un espace de travail Azure Databricks *Standard* ou d’*essai*, vous pouvez ignorer cette procédure et utiliser votre espace de travail existant.

1. Dans un navigateur web, connectez-vous au [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com`.
2. Utilisez le bouton **[\>_]** à droite de la barre de recherche, en haut de la page, pour créer un environnement Cloud Shell dans le portail Azure, en sélectionnant un environnement ***PowerShell*** et en créant le stockage si vous y êtes invité. Cloud Shell fournit une interface de ligne de commande dans un volet situé en bas du portail Azure, comme illustré ici :

    ![Portail Azure avec un volet Cloud Shell](./images/cloud-shell.png)

    > **Remarque** : si vous avez créé un shell cloud qui utilise un environnement *Bash*, utilisez le menu déroulant en haut à gauche du volet Cloud Shell pour le remplacer par ***PowerShell***.

3. Notez que vous pouvez redimensionner le volet Cloud Shell en faisant glisser la barre de séparation en haut du volet. Vous pouvez aussi utiliser les icônes **&#8212;** , **&#9723;** et **X** situées en haut à droite du volet pour réduire, agrandir et fermer le volet. Pour plus d’informations sur l’utilisation d’Azure Cloud Shell, consultez la [documentation Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. Dans le volet PowerShell, entrez les commandes suivantes pour cloner ce référentiel :

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Une fois le référentiel cloné, entrez les commandes suivantes pour accéder au dossier de ce labo et exécutez le script **setup.ps1** qu’il contient :

    ```
    cd dp-203/Allfiles/labs/24
    ./setup.ps1
    ```

6. Si vous y êtes invité, choisissez l’abonnement à utiliser (uniquement si vous avez accès à plusieurs abonnements Azure).

7. Attendez que le script se termine. Cette opération prend généralement environ 5 minutes, mais dans certains cas, elle peut être plus longue. Pendant que vous attendez, consultez l’article [Analyse exploratoire des données dans Azure Databricks](https://learn.microsoft.com/azure/databricks/exploratory-data-analysis/) dans la documentation Azure Databricks.

## Créer un cluster

Azure Databricks est une plateforme de traitement distribuée qui utilise des *clusters Apache Spark* pour traiter des données en parallèle sur plusieurs nœuds. Chaque cluster se compose d’un nœud de pilote pour coordonner le travail et les nœuds Worker pour effectuer des tâches de traitement.

> **Conseil** : si vous disposez déjà d’un cluster avec une version du runtime 13.3 LTS dans votre espace de travail Azure Databricks, vous pouvez l’utiliser pour effectuer cet exercice et ignorer cette procédure.

1. Dans le portail Azure, accédez au groupe de ressources **dp203-*xxxxxxx*** créé par le script (ou le groupe de ressources contenant votre espace de travail Azure Databricks existant)
1. Sélectionnez votre ressource de service Azure Databricks (nommée **databricks*xxxxxxx*** si vous avez utilisé le script d’installation pour la créer).
1. Dans la page **Vue d’ensemble** de votre espace de travail, utilisez le bouton **Lancer l’espace de travail** pour ouvrir votre espace de travail Azure Databricks dans un nouvel onglet de navigateur et connectez-vous si vous y êtes invité.

    > **Conseil** : lorsque vous utilisez le portail de l’espace de travail Databricks, plusieurs conseils et notifications peuvent s’afficher. Ignorez-les et suivez les instructions fournies pour effectuer les tâches de cet exercice.

1. Affichez le portail de l’espace de travail Azure Databricks et notez que la barre latérale gauche contient des icônes indiquant les différentes tâches que vous pouvez effectuer.

1. Sélectionnez la tâche **(+) Nouveau**, puis **Cluster**.
1. Dans la page **Nouveau cluster**, créez un cluster avec les paramètres suivants :
    - **Nom du cluster** : cluster de *nom d’utilisateur* (nom de cluster par défaut)
    - **Mode cluster** : nœud unique
    - **Mode d’accès ** : utilisateur unique (*avec votre compte d’utilisateur sélectionné*)
    - **Version du runtime Databricks** : 13.3 LTS (Spark 3.4.1, Scala 2.12)
    - **Utiliser l’accélération photon** : sélectionné
    - **Type de nœud** : Standard_DS3_v2
    - **Terminer après** *30* **minutes d’inactivité**

1. Attendez que le cluster soit créé. Cette opération peut prendre une à deux minutes.

> **Remarque** : si votre cluster ne démarre pas, le quota de votre abonnement est peut-être insuffisant dans la région où votre espace de travail Azure Databricks est approvisionné. Pour plus d’informations, consultez l’article [La limite de cœurs du processeur empêche la création du cluster](https://docs.microsoft.com/azure/databricks/kb/clusters/azure-core-limit). Si cela se produit, vous pouvez essayer de supprimer votre espace de travail et d’en créer un dans une autre région. Vous pouvez spécifier une région comme paramètre pour le script d’installation comme suit : `./setup.ps1 eastus`

## Explorer les données à l’aide d’un notebook

Comme dans de nombreux environnements Spark, Databricks prend en charge l’utilisation de notebooks pour combiner des notes et des cellules de code interactives que vous pouvez utiliser pour explorer les données.

1. Dans le portail de l’espace de travail Azure Databricks pour votre espace de travail, dans la barre latérale gauche, sélectionnez **Espace de travail**. Sélectionnez ensuite le dossier **⌂ Accueil**.
1. En haut de la page, dans le menu **⋮** en regard de votre nom d’utilisateur, sélectionnez **Importer**. Ensuite, dans la boîte de dialogue **Importer**, sélectionnez **URL** et importez le notebook à partir de `https://github.com/MicrosoftLearning/dp-203-azure-data-engineer/raw/master/Allfiles/labs/24/Databricks-Spark.ipynb`
1. Connectez le notebook à votre cluster et suivez les instructions qu’il contient ; en exécutant les cellules qu’il contient pour explorer les données dans les fichiers.

## Supprimer les ressources d'Azure Databricks

Maintenant que vous avez terminé d’explorer Azure Databricks, vous devez supprimer les ressources que vous avez créées pour éviter les coûts Azure inutiles et libérer de la capacité dans votre abonnement.

1. Fermez l’onglet du navigateur de l’espace de travail Azure Databricks et retournez au portail Azure.
2. Dans le portail Azure, dans la page **Accueil**, sélectionnez **Groupes de ressources**.
3. Sélectionnez le groupe de ressources **dp203-*xxxxxxx*** (et non le groupe de ressources managées) et vérifiez qu’il contient votre espace de travail Azure Databricks.
4. Au sommet de la page **Vue d’ensemble** de votre groupe de ressources, sélectionnez **Supprimer le groupe de ressources**.
5. Saisissez le nom du groupe de ressources **dp203-*xxxxxxx*** pour confirmer que vous souhaitez le supprimer, puis sélectionnez **Supprimer**.

    Après quelques minutes, votre groupe de ressources et les groupes de ressources de l’espace de travail managées qui lui sont associés seront supprimés.
