---
lab:
  title: "Automatiser un notebook Azure\_Databricks avec Azure\_Data\_Factory"
  ilt-use: Suggested demo
---

# Automatiser un notebook Azure Databricks avec Azure Data Factory

Vous pouvez utiliser des notebooks dans Azure Databricks pour effectuer des tâches d’ingénierie des données, telles que le traitement des fichiers de données et le chargement de données dans des tables. Lorsque vous devez orchestrer ces tâches dans le cadre d’un pipeline d’ingénierie des données, vous pouvez utiliser Azure Data Factory.

Cet exercice devrait prendre environ **40** minutes.

## Avant de commencer

Vous avez besoin d’un [abonnement Azure](https://azure.microsoft.com/free) dans lequel vous avez un accès administratif.

## Approvisionner des ressources Azure

Dans cet exercice, vous allez utiliser un script pour approvisionner un nouvel espace de travail Azure Databricks et une ressource Azure Data Factory dans votre abonnement Azure.

1. Dans un navigateur web, connectez-vous au [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com`.
2. Utilisez le bouton **[\>_]** à droite de la barre de recherche, en haut de la page, pour créer un environnement Cloud Shell dans le portail Azure, en sélectionnant un environnement ***PowerShell*** et en créant le stockage si vous y êtes invité. Cloud Shell fournit une interface de ligne de commande dans un volet situé en bas du portail Azure, comme illustré ici :

    ![Portail Azure avec un volet Cloud Shell](./images/cloud-shell.png)

    > **Remarque** : si vous avez déjà créé un interpréteur de commandes cloud qui utilise un environnement *Bash*, utilisez le menu déroulant en haut à gauche du volet de l’interpréteur de commandes cloud pour le remplacer par ***PowerShell***.

3. Notez que vous pouvez redimensionner le volet Cloud Shell en faisant glisser la barre de séparation en haut du volet. Vous pouvez aussi utiliser les icônes **&#8212;** , **&#9723;** et **X** situées en haut à droite du volet pour réduire, agrandir et fermer le volet. Pour plus d’informations sur l’utilisation d’Azure Cloud Shell, consultez la [documentation Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. Dans le volet PowerShell, entrez les commandes suivantes pour cloner ce référentiel :

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Une fois que le référentiel a été cloné, entrez les commandes suivantes pour accéder au dossier de ce labo et exécutez le script **setup.ps1** qu’il contient :

    ```
    cd dp-203/Allfiles/labs/27
    ./setup.ps1
    ```

6. Si vous y êtes invité, choisissez l’abonnement à utiliser (uniquement si vous avez accès à plusieurs abonnements Azure).

7. Attendez que le script se termine. Cela prend généralement environ 5 minutes, mais dans certains cas, cela peut prendre plus de temps. Pendant que vous attendez, consultez [Présentation d’Azure Data Factory](https://docs.microsoft.com/azure/data-factory/introduction).
8. Lorsque le script est terminé, fermez le volet de l’interpréteur de commandes cloud et accédez au groupe de ressources **dp203-*xxxxxxx*** créé par le script pour vérifier qu’il contient un espace de travail Azure Databricks et une ressource Azure Data Factory (V2) (vous devrez peut-être actualiser la vue du groupe de ressources).

## Importer un notebook

Vous pouvez créer des notebooks dans votre espace de travail Azure Databricks pour exécuter du code écrit dans divers langages de programmation. Dans cet exercice, vous allez importer un notebook existant qui contient du code Python.

1. Dans le portail Azure, accédez au groupe de ressources **dp203-*xxxxxxx*** créé par le script que vous avez exécuté.
2. Sélectionnez la ressource Azure Databricks Service **databricks*xxxxxxx***.
3. Dans la page **Vue d’ensemble** de **databricks*xxxxxxx***, utilisez le bouton **Lancer l’espace de travail** pour ouvrir votre espace de travail Azure Databricks dans un nouvel onglet de navigateur. Connectez-vous si vous y êtes invité.
4. Si un message **Quel est votre projet de données actuel ?** s’affiche, sélectionnez **Terminer** pour le fermer. Affichez ensuite le portail de l’espace de travail Azure Databricks et notez que la barre latérale à gauche contient des icônes pour les différentes tâches que vous pouvez effectuer.

    >**Conseil** : lorsque vous utilisez le portail de l’espace de travail Databricks, différents conseils et notifications peuvent être affichés. Ignorez-les et suivez les instructions fournies pour effectuer les tâches de cet exercice.

1. Dans la barre latérale à gauche, sélectionnez **Espaces de travail**. Sélectionnez ensuite le dossier **⌂ Accueil**.
1. En haut de la page, dans le menu **⋮** en regard de votre nom d’utilisateur, sélectionnez **Importer**. Ensuite, dans la boîte de dialogue **Importer**, sélectionnez **URL** et importez le notebook à partir de `https://github.com/MicrosoftLearning/dp-203-azure-data-engineer/raw/master/Allfiles/labs/27/Process-Data.ipynb`
1. Passez en revue le contenu du notebook, qui inclut des cellules de code Python pour :
    - Récupérer un paramètre nommé **folder** s’il a été transmis (sinon, utilisez la valeur par défaut *data*).
    - Télécharger des données de GitHub et les enregistrer dans le dossier spécifié dans le Système de fichiers Databricks (DBFS).
    - Quitter le notebook, pour revenir au chemin d’accès où les données ont été enregistrées en sortie.

    > **Conseil** : le notebook peut contenir quasiment toute logique de traitement des données dont vous avez besoin. Cet exemple simple est conçu pour illustrer les principes fondamentaux.

## Activer l’intégration d’Azure Databricks avec Azure Data Factory

Pour utiliser Azure Databricks à partir d’un pipeline Azure Data Factory, vous devez créer un service lié dans Azure Data Factory qui permet d’accéder à votre espace de travail Azure Databricks.

### Générer un jeton d’accès

1. Dans le portail Azure Databricks, dans la barre de menus en haut à droite, sélectionnez le nom d’utilisateur, puis **Paramètres utilisateurs** dans la liste déroulante.
1. Dans la page **Paramètres utilisateurs**, sélectionnez **Développeur**. Ensuite, en regard de **Jetons d’accès**, sélectionnez **Gérer**.
1. Sélectionnez **Générer un nouveau jeton** et générez un nouveau jeton avec le commentaire *Data Factory* et une durée de vie vide (le jeton n’expire donc pas). Veillez à **copier le jeton lorsqu’il est affiché <u>avant</u> de sélectionner *Terminé***.
1. Collez le jeton copié dans un fichier texte afin de pouvoir l’utiliser ultérieurement dans cet exercice.

### Créer un service lié dans Azure Data Factory

1. Revenez au portail Azure et, dans le groupe de ressources **dp203-*xxxxxxx***, sélectionnez la ressource Azure Data Factory **adf*xxxxxxx***.
2. Dans la page **Vue d’ensemble**, sélectionnez **Lancer Studio** pour ouvrir Azure Data Factory Studio. Connectez-vous si vous y êtes invité.
3. Dans Azure Data Factory Studio, utilisez l’icône **>>** pour développer le volet de navigation à gauche. Sélectionnez ensuite la page **Gérer**.
4. Dans la page **Gérer**, sous l’onglet **Services liés**, sélectionnez **+ Nouveau** pour ajouter un nouveau service lié.
5. Dans le volet **Nouveau service lié**, sélectionnez l’onglet **Calcul** dans la partie supérieure. Sélectionnez ensuite **Azure Databricks**.
6. Créez ensuite le service lié avec les paramètres suivants :
    - **Nom** : AzureDatabricks
    - **Description** : espace de travail Azure Databricks
    - **Se connecter via un runtime d’intégration** : AutoResolveIntegrationRuntime
    - **Méthode de sélection du compte** : à partir d’un abonnement Azure
    - **Abonnement Azure** : *sélectionnez votre abonnement*
    - **Espace de travail Databricks** : *sélectionnez votre espace de travail **databricksxxxxxxx***
    - **Sélectionner un cluster** : nouveau cluster de travail
    - **URL de l’espace de travail Databricks** : *définie automatiquement sur l’URL de votre espace de travail Databricks*
    - **Type d’authentification** : jeton d’accès
    - **Jeton d’accès** : *collez votre jeton d’accès*
    - **Version du cluster** : 12.2 LTS (Scala 2.12, Spark 3.2.2)
    - **Type de nœud de cluster ** : Standard_DS3_v2
    - **Version de Python** : 3
    - **Options de Worker** : fixe
    - **Workers** : 1

## Utiliser un pipeline pour exécuter le notebook Azure Databricks

Maintenant que vous avez créé un service lié, vous pouvez l’utiliser dans un pipeline pour exécuter le notebook que vous avez consulté précédemment.

### Créer un pipeline

1. Dans Azure Data Factory Studio, sélectionnez **Créer** dans le volet de navigation.
2. Dans la page **Créer**, dans le volet **Ressources Factory**, utilisez l’icône **+** pour ajouter un **pipeline**.
3. Dans le volet **Propriétés** du nouveau pipeline, remplacez son nom par **Traiter les données avec Databricks**. Utilisez ensuite le bouton **Propriétés** (qui ressemble à **<sub>*</sub>**) à droite de la barre d’outils pour masquer le volet **Propriétés**.
4. Dans le volet **Activités**, développez **Databricks**, puis faites glisser une activité **Notebook** vers la surface du concepteur de pipeline.
5. Une fois la nouvelle activité **Notebook1** sélectionnée, définissez les propriétés suivantes dans le volet du bas :
    - **Général :**
        - **Nom** : traiter les données
    - **Azure Databricks** :
        - **Service lié Databricks** : *sélectionnez le service lié **AzureDatabricks** que vous avez créé précédemment*
    - **Paramètres**:
        - **Chemin d’accès du notebook** : *accédez au dossier **Users/votre_nom_utilisateur** et sélectionnez le notebook **Process-Data***
        - **Paramètres de base** : *ajoutez un nouveau paramètre nommé **folder** avec la valeur **product_data***
6. Utilisez le bouton **Valider** au-dessus de la surface du concepteur de pipeline pour valider le pipeline. Utilisez ensuite le bouton **Publier tout** pour le publier (l’enregistrer).

### Exécuter le pipeline

1. Au-dessus de la surface du concepteur de pipeline, sélectionnez **Ajouter un déclencheur**, puis sélectionnez **Déclencher maintenant**.
2. Dans le volet **Exécution de pipeline**, sélectionnez **OK** pour exécuter le pipeline.
3. Dans le volet de navigation de gauche, sélectionnez **Superviser** et observez le pipeline **Traiter les données avec Databricks** sous l’onglet **Exécutions de pipeline**. L’exécution peut prendre un certain temps en raison de la création dynamique d’un cluster Spark et de l’exécution du notebook. Vous pouvez utiliser le bouton **↻ Actualiser** sur la page **Exécutions de pipeline** pour actualiser l’état.

    > **Remarque** : si votre pipeline échoue, il se peut que le quota de votre abonnement soit insuffisant dans la région où votre espace de travail Azure Databricks est approvisionné pour créer un cluster de travail. Pour plus d’informations, consultez l’article [La limite de cœurs du processeur empêche la création du cluster](https://docs.microsoft.com/azure/databricks/kb/clusters/azure-core-limit). Si cela se produit, vous pouvez essayer de supprimer votre espace de travail et d’en créer un dans une autre région. Vous pouvez spécifier une région comme paramètre pour le script d’installation comme suit : `./setup.ps1 eastus`

4. Une fois l’exécution réussie, sélectionnez son nom pour afficher les détails de l’exécution. Ensuite, dans la page **Traiter les données avec Databricks**, dans la section **Exécutions d’activité**, sélectionnez l’activité **Traiter les données** et utilisez son icône de ***sortie*** pour afficher le code JSON de sortie de l’activité, qui doit ressembler à ceci :
    ```json
    {
        "runPageUrl": "https://adb-..../run/...",
        "runOutput": "dbfs:/product_data/products.csv",
        "effectiveIntegrationRuntime": "AutoResolveIntegrationRuntime (East US)",
        "executionDuration": 61,
        "durationInQueue": {
            "integrationRuntimeQueue": 0
        },
        "billingReference": {
            "activityType": "ExternalActivity",
            "billableDuration": [
                {
                    "meterType": "AzureIR",
                    "duration": 0.03333333333333333,
                    "unit": "Hours"
                }
            ]
        }
    }
    ```

5. Notez la valeur **runOutput**, qui est la variable de *chemin d’accès* à laquelle le notebook a enregistré les données.

## Supprimer les ressources Azure Databricks

Maintenant que vous avez terminé d’explorer l’intégration d’Azure Data Factory à Azure Databricks, vous devez supprimer les ressources que vous avez créées pour éviter les coûts Azure inutiles et libérer de la capacité dans votre abonnement.

1. Fermez l’espace de travail Azure Databricks et les onglets du navigateur Azure Data Factory Studio, puis revenez au portail Azure.
2. Dans le portail Azure, dans la page **Accueil**, sélectionnez **Groupes de ressources**.
3. Sélectionnez le groupe de ressources **dp203-*xxxxxxx*** contenant votre espace de travail Azure Databricks et Azure Data Factory (et non le groupe de ressources managé).
4. Au sommet de la page **Vue d’ensemble** de votre groupe de ressources, sélectionnez **Supprimer le groupe de ressources**.
5. Entrez le nom du groupe de ressources pour confirmer que vous souhaitez le supprimer, puis sélectionnez **Supprimer**.

    Après quelques minutes, votre groupe de ressources et le groupe de ressources de l’espace de travail managé qui lui est associé seront supprimés.
