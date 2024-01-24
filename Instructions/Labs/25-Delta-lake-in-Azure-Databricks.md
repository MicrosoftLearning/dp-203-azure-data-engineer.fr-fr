---
lab:
  title: Utiliser Delta Lake dans Azure Databricks
  ilt-use: Optional demo
---

# Utiliser Delta Lake dans Azure Databricks

Delta Lake est un projet code source ouvert pour créer une couche de stockage de données transactionnelle pour Spark au-dessus d’un lac de données. Delta Lake ajoute la prise en charge de la sémantique relationnelle pour les opérations de données par lots et de streaming, et permet la création d’une architecture Lakehouse, dans laquelle Apache Spark peut être utilisé pour traiter et interroger des données dans des tables basées sur des fichiers sous-jacents dans un lac de données.

Cet exercice devrait prendre environ **40** minutes.

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
    cd dp-203/Allfiles/labs/25
    ./setup.ps1
    ```

6. Si vous y êtes invité, choisissez l’abonnement que vous souhaitez utiliser (cela se produit uniquement si vous avez accès à plusieurs abonnements Azure).

7. Attendez que le script se termine, ce qui prend généralement entre 5 et 10 minutes. Pendant que vous attendez, consultez l’article [Introduction à Delta Technologies](https://learn.microsoft.com/azure/databricks/introduction/delta-comparison) dans la documentation Azure Databricks.

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

## Explorer delta lake à l’aide d’un notebook

Dans cet exercice, vous allez utiliser du code dans un notebook pour explorer delta lake dans Azure Databricks.

1. Dans la barre de menus à gauche, sélectionnez **Espaces de travail**. Sélectionnez ensuite le **&#8962 ; Dossier d’accueil** .
1. En haut de la page, dans le **menu &#8942 ;** en regard de votre nom d’utilisateur, sélectionnez **Importer**. Ensuite, dans la **boîte de dialogue Importer** , sélectionnez **l’URL** et importez le bloc-notes à partir de `https://github.com/MicrosoftLearning/dp-203-azure-data-engineer/raw/master/Allfiles/labs/25/Delta-Lake.ipynb`
1. Connecter le bloc-notes de votre cluster et suivez les instructions qu’il contient ; en exécutant les cellules qu’il contient pour explorer les fonctionnalités delta lake.

## Ressources Azure Databricks

Maintenant que vous avez terminé d’explorer Delta Lake dans Azure Databricks, vous devez supprimer les ressources que vous avez créées pour éviter les coûts Azure inutiles et libérer de la capacité dans votre abonnement.

1. Fermez l’onglet du navigateur et retournez sous l’onglet Portail Azure.
2. Dans le portail Azure, dans la page **Accueil**, sélectionnez **Groupes de ressources**.
3. Sélectionnez le **groupe de ressources dp203-*xxxxxxx*** (et non le groupe de ressources managés) et vérifiez qu’il contient votre espace de travail Azure Databricks.
4. Au sommet de la page **Vue d’ensemble** de votre groupe de ressources, sélectionnez **Supprimer le groupe de ressources**.
5. Entrez le nom du groupe de ressources pour confirmer que vous souhaitez le supprimer, puis sélectionnez Supprimer.

    Après quelques minutes, votre espace de travail Azure Synapse et l’espace de travail managé qui lui est associé seront supprimés.
