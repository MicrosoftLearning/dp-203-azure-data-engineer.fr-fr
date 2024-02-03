---
lab:
  title: "Utiliser un notebook Apache\_Spark dans un pipeline"
  ilt-use: Lab
---

# Utiliser un notebook Apache Spark dans un pipeline

Dans cet exercice, nous allons créer un pipeline Azure Synapse Analytics qui inclut une activité pour exécuter un notebook Apache Spark.

Cet exercice devrait prendre environ **30** minutes.

## Avant de commencer

Vous avez besoin d’un [abonnement Azure](https://azure.microsoft.com/free) dans lequel vous avez un accès administratif.

## Provisionner un espace de travail Azure Synapse Analytics

Vous aurez besoin d’un espace de travail Azure Synapse Analytics avec accès au Data Lake Storage et d’un pool Spark.

Dans cet exercice, vous allez utiliser la combinaison d’un script PowerShell et d’un modèle ARM pour approvisionner un espace de travail Azure Synapse Analytics.

1. Connectez-vous au [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com`.
2. Utilisez le bouton **[\>_]** à droite de la barre de recherche, en haut de la page, pour créer un environnement Cloud Shell dans le portail Azure, puis sélectionnez un environnement ***PowerShell*** et créez le stockage si vous y êtes invité. Cloud Shell fournit une interface de ligne de commande dans un volet situé en bas du portail Azure, comme illustré ici :

    ![Portail Azure avec un volet Cloud Shell](./images/cloud-shell.png)

    > **Remarque** : si vous avez déjà créé un interpréteur de commandes cloud qui utilise un environnement *Bash*, utilisez le menu déroulant en haut à gauche du volet de l’interpréteur de commandes Cloud Shell pour le remplacer par ***PowerShell***.

3. Notez que vous pouvez redimensionner Cloud Shell en faisant glisser la barre de séparation en haut du volet. Vous pouvez aussi utiliser les icônes —, **◻** et **X** situées en haut à droite du volet pour réduire, agrandir et fermer le volet. Pour plus d’informations sur l’utilisation d’Azure Cloud Shell, consultez la [documentation Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. Dans le volet PowerShell, entrez les commandes suivantes pour cloner ce référentiel :

    ```powershell
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Une fois que le référentiel a été cloné, entrez les commandes suivantes pour accéder au dossier de cet exercice et exécutez le script **setup.ps1** qu’il contient :

    ```powershell
    cd dp-203/Allfiles/labs/11
    ./setup.ps1
    ```
    
6. Si vous y êtes invité, choisissez l’abonnement à utiliser (uniquement si vous avez accès à plusieurs abonnements Azure).
7. Quand vous y êtes invité, entrez un mot de passe approprié à définir pour votre pool Azure Synapse SQL.

    > **Remarque** : veillez à mémoriser ce mot de passe.

8. Attendez que le script se termine. Cela prend généralement environ 10 minutes, mais dans certains cas, cela peut prendre plus de temps. Pendant que vous attendez, consultez l’article [Pipelines Azure Synapse](https://learn.microsoft.com/en-us/azure/data-factory/concepts-data-flow-performance-pipelines) dans la documentation Azure Synapse Analytics.

## Exécuter un notebook Spark de manière interactive

Avant d’automatiser un processus de transformation de données avec un notebook, il peut être utile d’exécuter le notebook de manière interactive pour mieux comprendre le processus que vous automatiserez ultérieurement.

1. Une fois le script terminé, dans le portail Azure, accédez au groupe de ressources dp203-xxxxxxx qu’il a créé, puis sélectionnez votre espace de travail Synapse.
2. Dans la page **Vue d’ensemble** de votre espace de travail Synapse, dans la carte **Ouvrir Synapse Studio**, sélectionnez **Ouvrir** pour ouvrir Synapse Studio dans un nouvel onglet de navigateur. Connectez-vous si vous y êtes invité.
3. Sur le côté gauche de Synapse Studio, utilisez l’icône ›› pour développer le menu et afficher les différentes pages de Synapse Studio.
4. Dans la page **Données**, affichez l’onglet Lié et vérifiez que votre espace de travail inclut un lien vers votre compte de stockage Azure Data Lake Storage Gen2, qui doit avoir un nom similaire à **synapsexxx (Primary - datalakexxxxxxxxx)**.
5. Développez votre compte de stockage et vérifiez qu’il contient un conteneur de système de fichiers nommé **files (primary)**.
6. Sélectionnez le conteneur de fichiers. Notez qu’il contient un dossier nommé **data** avec les fichiers de données que vous allez transformer.
7. Ouvrez le dossier **data**** et affichez les fichiers CSV qu’il contient. Cliquez avec le bouton droit sur l’un des fichiers, puis sélectionnez **Aperçu** pour afficher un exemple de données. Fermez l’aperçu lorsque vous avez terminé.
8. Dans Synapse Studio, dans la page **Développer**, développez **Notebooks** et ouvrez le notebook **Spark Transform**.

    > **Remarque** : si vous constatez que le notebook n’est pas chargé pendant le script d’exécution, vous devez télécharger le fichier nommé Spark Transform.ipynb à partir de GitHub [Allfiles/labs/11/notebooks](https://github.com/MicrosoftLearning/dp-203-azure-data-engineer/tree/master/Allfiles/labs/11/notebooks) et le charger sur Synapse.

9. Passez en revue le code que contient le notebook et notez qu’il :
    - Définit une variable pour définir un nom de dossier unique.
    - Charge les données de commandes CSV à partir du dossier **/data**.
    - Transforme les données en fractionnant le nom du client en plusieurs champs.
    - Enregistre les données transformées au format Parquet dans le dossier au nom unique.
10. Dans la barre d’outils du notebook, attachez le notebook à votre pool Spark **spark*xxxxxxx***, puis utilisez le bouton **▷ Exécuter tout** pour exécuter toutes les cellules de code dans le notebook.
  
    Le démarrage de l’exécution des cellules de code de la session Spark peut prendre quelques minutes.

11. Lorsque toutes les cellules du notebook ont été exécutées, notez le nom du dossier dans lequel les données transformées ont été enregistrées.
12. Passez à l’onglet **Fichiers** (qui doit toujours être ouvert) et affichez le dossier racine **files**. Si nécessaire, dans le menu **Plus**, sélectionnez **Actualiser** pour afficher le nouveau dossier. Ouvrez-le ensuite pour vérifier qu’il contient des fichiers Parquet.
13. Revenez au dossier racine **files** et sélectionnez le dossier au nom unique généré par le notebook puis, dans le menu **Nouveau script SQL**, sélectionnez **Sélectionner les 100 premières lignes**.
14. Dans le volet **Sélectionner les 100 premières lignes**, définissez le type de fichier sur **Format Parquet** et appliquez la modification.
15. Dans le volet de nouveau script SQL qui s’ouvre, utilisez le bouton **▷ Exécuter** pour exécuter le code SQL et vérifier qu’il retourne les données de commandes transformées.

## Exécuter le notebook dans un pipeline

Maintenant que vous comprenez le processus de transformation, vous êtes prêt à l’automatiser en encapsulant le notebook dans un pipeline.

### Créer une cellule de paramètres

1. Dans Synapse Studio, revenez à l’onglet **Transformation Spark** qui contient le notebook et, dans la barre d’outils, dans le menu **...** à droite, sélectionnez **Effacer la sortie**.
2. Sélectionnez la première cellule de code (qui contient le code pour définir la variable **folderName**).
3. Dans la barre d’outils contextuelle située en haut à droite de la cellule de code, dans le menu **...**, sélectionnez **\[@] Activer/désactiver la cellule de paramètre**. Vérifiez que le mot **paramètres** apparaît en bas à droite de la cellule.
4. Dans la barre d’outils, utilisez le bouton **Publier** pour enregistrer les modifications.

### Créer un pipeline

1. Dans Synapse Studio, sélectionnez la page **Intégrer**. Ensuite, dans le menu **+**, sélectionnez **Pipeline** pour créer un pipeline.
2. Dans le volet des **Propriétés** de votre nouveau pipeline, changez son nom en remplaçant **Pipeline1** par **Transform Sales Data**. Utilisez ensuite le bouton **Propriétés** au-dessus du volet des **Propriétés** pour le masquer.
3. Dans le volet **Activités**, développez **Synapse**, puis faites glisser une activité de **Notebook** sur l’aire de conception du pipeline, comme illustré ici :

    ![Capture d’écran d’un pipeline avec une activité de notebook.](images/notebook-pipeline.png)

4. Sous l’onglet **Général** de l’activité de notebook, remplacez son nom par **Exécuter la transformation Spark**.
5. Sous l’onglet **Paramètres** de l’activité de notebook, définissez les propriétés suivantes :
    - **Notebook** : sélectionnez le notebook **Transformation Spark**.
    - **Paramètres de base** : développez cette section et définissez un paramètre avec les paramètres suivants :
        - **Nom** : NomDossier
        - **Type** : Chaîne
        - **Valeur** : sélectionnez **Ajouter du contenu dynamique** et définissez la valeur du paramètre sur la variable système *ID d’exécution du pipeline* (`@pipeline().RunId`)
    - **Pool Spark** : sélectionnez le pool **spark*xxxxxxx***.
    - **Taille de l’exécuteur** : sélectionnez **Petite (4 vCores, 28 Go de mémoire)**.

    Votre volet Pipeline doit ressembler à ceci :

    ![Capture d’écran d’un pipeline avec une activité de notebook et des paramètres.](images/notebook-pipeline-settings.png)

### Publier et exécuter le pipeline

1. Utilisez le bouton **Publier tout** pour publier le pipeline (et toutes les autres ressources non enregistrées).
2. En haut du volet Concepteur de pipeline, dans le menu **Ajouter un déclencheur**, sélectionnez **Déclencher maintenant**. Sélectionnez ensuite **OK** pour confirmer que vous souhaitez exécuter le pipeline.

    **Remarque** : vous pouvez également créer un déclencheur pour exécuter le pipeline à un moment planifié ou en réponse à un événement spécifique.

3. Une fois le pipeline en cours d’exécution, dans la page **Moniteur**, affichez l’onglet **Exécutions de pipeline** et passez en revue l’état du pipeline **Transformer les données de ventes**.
4. Sélectionnez le pipeline **Transformer les données de ventes** pour afficher ses détails, puis notez l’ID d’exécution du pipeline dans le volet **Exécutions d’activité**.

    Le pipeline peut prendre cinq minutes ou plus pour s’achever. Vous pouvez utiliser le bouton **↻ Actualiser** dans la barre d’outils pour vérifier son état.

5. Lorsque l’exécution du pipeline a réussi, dans la page **Données**, accédez au conteneur de stockage **files** et vérifiez qu’un nouveau dossier nommé pour l’ID d’exécution du pipeline a été créé et qu’il contient des fichiers Parquet pour les données de ventes transformées.
   
## Supprimer les ressources Azure

Si vous avez fini d’explorer Azure Synapse Analytics, vous devriez supprimer les ressources que vous avez créées afin d’éviter des coûts Azure inutiles.

1. Fermez l’onglet du navigateur Synapse Studio et revenez dans le portail Azure.
2. Dans le portail Azure, dans la page **Accueil**, sélectionnez **Groupes de ressources**.
3. Sélectionnez le groupe de ressources **dp203-*xxxxxxx*** de votre espace de travail Synapse Analytics (et non le groupe de ressources managé) et vérifiez qu’il contient l’espace de travail Synapse, le compte de stockage et le pool Spark de votre espace de travail.
4. Au sommet de la page **Vue d’ensemble** de votre groupe de ressources, sélectionnez **Supprimer le groupe de ressources**.
5. Entrez le nom du groupe de ressources **dp203-*xxxxxxx*** pour confirmer que vous souhaitez le supprimer, puis sélectionnez **Supprimer**.

    Après quelques minutes, le groupe de ressources de l’espace de travail Azure Synapse et le groupe de ressources managé de l’espace de travail qui lui est associé seront supprimés.
