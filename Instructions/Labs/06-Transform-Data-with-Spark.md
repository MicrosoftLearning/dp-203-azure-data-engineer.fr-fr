---
lab:
  title: Transformer des données en utilisant Spark dans Synapse Analytics
  ilt-use: Lab
---

# Transformer des données en utilisant Spark dans Synapse Analytics

Les *Ingénieurs* Données utilisent souvent des notebooks Spark comme l’un de leurs outils préférés pour effectuer des activités d’*extraction, de transformation et de chargement (ETL)* ou d’*extraction, de chargement et de transformation (ELT)* qui transforment des données d’un format ou d’une structure vers une autre.

Dans cet exercice, vous allez utiliser un notebook Spark dans Azure Synapse Analytics pour transformer des données dans des fichiers.

Cet exercice devrait prendre environ **30** minutes.

## Avant de commencer

Vous avez besoin d’un [abonnement Azure](https://azure.microsoft.com/free) dans lequel vous avez un accès administratif.

## Provisionner un espace de travail Azure Synapse Analytics

Vous aurez besoin d’un espace de travail Azure Synapse Analytics avec accès au Data Lake Storage et d’un pool Spark.

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
    cd dp-203/Allfiles/labs/06
    ./setup.ps1
    ```

6. Si vous y êtes invité, choisissez l’abonnement à utiliser (uniquement si vous avez accès à plusieurs abonnements Azure).
7. Une fois invité, entrez un mot de passe approprié à définir pour le pool SQL Azure Synapse.

    > **Remarque** : veillez à mémoriser ce mot de passe.

8. Attendez que le script se termine. Cela prend généralement environ 10 minutes, mais dans certains cas, cela peut prendre plus de temps. Pendant l’attente, passez en revue l’article [Apache Spark dans Azure Synapse Analytics – Concepts clés](https://learn.microsoft.com/azure/synapse-analytics/spark/apache-spark-concepts) dans la documentation Azure Synapse Analytics.

## Utiliser un notebook Spark pour transformer des données

1. Une fois le script de déploiement terminé, dans le portail Azure, accédez au groupe de ressources **dp203-*xxxxxxx*** qu’il a créé, et notez que ce groupe de ressources contient votre espace de travail Synapse, un compte de stockage pour votre lac de données et un pool Apache Spark.
2. Sélectionnez votre espace de travail Synapse puis, dans sa page **Vue d’ensemble**, dans la carte **Ouvrir Synapse Studio**, sélectionnez **Ouvrir** pour ouvrir Synapse Studio dans un nouvel onglet de navigateur. Connectez-vous si vous y êtes invité.
3. Sur le côté gauche de Synapse Studio, utilisez l’icône **&rsaquo;&rsaquo;** pour développer le menu. Cela permet d’afficher les différentes pages de Synapse Studio qui vous permettront de gérer les ressources et d’effectuer des tâches d’analytique de données.
4. Sur la page **Gérer**, sélectionnez l’onglet **Pools Apache Spark** et notez qu’un pool Spark portant un nom similaire à **spark*xxxxxxx*** a été provisionné dans l’espace de travail.
5. Dans la page **Données**, affichez l’onglet **Lié** et vérifiez que votre espace de travail inclut un lien vers votre compte de stockage Azure Data Lake Storage Gen2, qui doit avoir un nom similaire à **synapse*xxxxxxx* (Primary - datalake*xxxxxxx*)**.
6. Développez votre compte de stockage et vérifiez qu’il contient un conteneur de système de fichiers nommé **fichier (Primaire)**.
7. Sélectionnez le conteneur de **fichiers** et notez qu’il contient des dossiers nommés **données** et **synapse**. Le dossier synapse est utilisé par Azure Synapse et le dossier **données** contient les fichiers de données que vous allez interroger.
8. Ouvrez le dossier **données** et observez qu’il contient des fichiers .csv pour trois années de données de ventes.
9. Cliquez avec le bouton droit sur l’un des fichiers et sélectionnez **Aperçu** pour afficher les données qu’il contient. Notez que les fichiers contiennent une ligne d’en-tête. Vous pouvez donc sélectionner l’option permettant d’afficher des en-têtes de colonnes.
10. Fermez l’aperçu. Télécharger ensuite le fichier **Spark Transform.ipynb** à [partir de Allfiles/labs/06/notebooks](https://github.com/MicrosoftLearning/dp-203-azure-data-engineer/tree/master/Allfiles/labs/06/notebooks)

    > **Remarque** : il est préférable de copier ce texte en utilisant ***Ctrl+A***, ***Ctrl+C***, puis de le coller en utilisant ***Ctrl+V*** dans un outil tel que le Bloc-notes. Sélectionnez ensuite le fichier, puis enregistrez-le sous le nom **Spark Transform.ipynb** avec le type de fichier ***Tous les fichiers***. Vous pouvez également télécharger le fichier en cliquant dessus, puis sélectionnez les points de suspension, enfin téléchargez en vous souvenant de l’emplacement où vous l’avez enregistré.
    ![Télécharger un notebook Spark à partir de GitHub](./images/select-download-notebook.png)

11. Ensuite, sur la page **Développer**, développer **Notebooks** et cliquer sur l’option + Importer

    ![Importer un notebook Spark](./image/../images/spark-notebook-import.png)
        
12. Sélectionnez le fichier que vous venez de télécharger et d’enregistrer sous le nom **Spark Transfrom.ipynb**.
13. Attachez le notebook à votre pool Spark **spark*xxxxxxx***.
14. Passez en revue les notes du notebook et exécutez les cellules de code.

    > **Remarque** : l’exécution de la première cellule de code prend quelques minutes, car le pool Spark doit être démarré. Les cellules suivantes s’exécutent plus rapidement.

## Supprimer les ressources Azure

Si vous avez fini d’explorer Azure Synapse Analytics, vous devriez supprimer les ressources que vous avez créées afin d’éviter des coûts Azure inutiles.

1. Fermez l’onglet du navigateur Synapse Studio et revenez dans le portail Azure.
2. Dans le portail Azure, dans la page **Accueil**, sélectionnez **Groupes de ressources**.
3. Sélectionnez le groupe de ressources **dp203-*xxxxxxx*** de votre espace de travail Synapse Analytics (et non le groupe de ressources managé) et vérifiez qu’il contient l’espace de travail Synapse, le compte de stockage et le pool Spark de votre espace de travail.
4. Au sommet de la page **Vue d’ensemble** de votre groupe de ressources, sélectionnez **Supprimer le groupe de ressources**.
5. Entrez le nom du groupe de ressources **dp203-*xxxxxxx*** pour confirmer que vous souhaitez le supprimer, puis sélectionnez **Supprimer**.

    Après quelques minutes, le groupe de ressources de l’espace de travail Azure Synapse et le groupe de ressources managé de l’espace de travail qui lui est associé seront supprimés.
