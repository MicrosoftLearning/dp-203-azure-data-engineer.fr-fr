---
lab:
  title: Bien démarrer avec Azure Stream Analytics
  ilt-use: Suggested demo
---

# Bien démarrer avec Azure Stream Analytics

Dans cet exercice, vous allez approvisionner un travail Azure Stream Analytics dans votre abonnement Azure, puis l’utiliser pour interroger et résumer un flux de données d’événement en temps réel et stocker les résultats dans Stockage Azure.

Vous devriez terminer cet exercice en **15** minutes environ.

## Avant de commencer

Vous avez besoin d’un [abonnement Azure](https://azure.microsoft.com/free) dans lequel vous avez un accès administratif.

## Approvisionner des ressources Azure

Dans cet exercice, vous allez capturer le flux des données d’une transaction de vente simulée, le traiter et stocker les résultats dans un conteneur d’objet blob dans Stockage Azure. Vous avez besoin d’un espace de noms Azure Event Hubs vers lequel vous pouvez envoyer les données de diffusion en continu et d’un compte Stockage Azure dans lequel vous allez stocker les résultats du traitement.

Vous allez utiliser la combinaison d’un script PowerShell et d’un modèle ARM pour approvisionner ces ressources.

1. Connectez-vous au [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com`.
2. Utilisez le bouton **[\>_]** à droite de la barre de recherche, en haut de la page, pour créer un environnement Cloud Shell dans le portail Azure, puis sélectionnez un environnement ***PowerShell*** et créez le stockage si vous y êtes invité. Cloud Shell fournit une interface de ligne de commande dans un volet situé en bas du portail Azure, comme illustré ici :

    ![Portail Azure avec un volet Cloud Shell](./images/cloud-shell.png)

    > **Remarque** : si vous avez créé un shell cloud qui utilise un environnement *Bash*, utilisez le menu déroulant en haut à gauche du volet Cloud Shell pour le remplacer par ***PowerShell***.

3. Notez que vous pouvez redimensionner le volet Cloud Shell en faisant glisser la barre de séparation en haut du volet. Vous pouvez aussi utiliser les icônes **&#8212;** , **&#9723;** et **X** situées en haut à droite du volet pour réduire, agrandir et fermer le volet. Pour plus d’informations sur l’utilisation d’Azure Cloud Shell, consultez la [documentation Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. Dans le volet PowerShell, entrez les commandes suivantes pour cloner le référentiel contenant cet exercice :

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Une fois que le référentiel a été cloné, entrez les commandes suivantes pour accéder au dossier de cet exercice et exécutez le script **setup.ps1** qu’il contient :

    ```
    cd dp-203/Allfiles/labs/17
    ./setup.ps1
    ```

6. Si vous y êtes invité, choisissez l’abonnement à utiliser (uniquement si vous avez accès à plusieurs abonnements Azure).
7. Attendez que le script se termine. Cela prend généralement environ 5 minutes, mais dans certains cas, cela peut prendre plus de temps. Pendant que vous attendez, consultez l’article [Bienvenue dans Azure Stream Analytics](https://learn.microsoft.com/azure/stream-analytics/stream-analytics-introduction) dans la documentation Azure Stream Analytics.

## Afficher la source de données de diffusion en continu

Avant de créer un travail Azure Stream Analytics pour traiter des données en temps réel, examinons le flux de données qu’il doit interroger.

1. Une fois l’exécution du script d’installation terminée, redimensionnez ou réduisez le volet Cloud Shell afin de pouvoir voir le Portail Azure (vous reviendrez au Cloud Shell plus tard). Puis dans le Portail Azure, accédez au groupe de ressources **dp203-*xxxxxxx*** qu’il a créé et notez que ce groupe de ressources contient un compte Stockage Azure et un espace de noms Event Hubs.

    Notez l’**Emplacement** d’approvisionnement des ressources. Vous créerez plus tard un travail Azure Stream Analytics dans le même emplacement.

2. Rouvrez le volet Cloud Shell et entrez les commandes suivantes pour exécuter une application cliente qui envoie 100 commandes simulées à Azure Event Hubs :

    ```
    node ~/dp-203/Allfiles/labs/17/orderclient
    ```

3. Observez les données de la commande client au moment de leur envoi, chaque envoi comporte un ID de produit et une quantité. L’application s’arrête après l’envoi de 1 000 commandes, ce qui prend une minute environ.

## Créer une tâche Azure Stream Analytics

Vous êtes maintenant prêt à créer un travail Azure Stream Analytics pour traiter les données de la transaction de vente quand elles arrivent dans l’Event Hub.

1. Dans le Portail Azure, dans la page **dp203-*xxxxxxx***, sélectionnez **+ Créer** et recherchez `Stream Analytics job`. Créez ensuite une **Tâche Stream Analytics** avec les propriétés suivantes :
    - **Paramètres de base**:
        - **Abonnement** : votre abonnement Azure.
        - **Groupe de ressources** : sélectionnez le groupe de ressources **dp203-*xxxxxxx*** existant.
        - **Nom :** `process-orders`
        - **Région** : Sélectionnez la région où vos autres ressources sont approvisionnées.
        - **Environnement d’hébergement** : cloud.
        - **Unités de diffusion en continu** : 1.
    - **Stockage** :
        - **Ajout d’un compte de stockage** : non sélectionné
    - **Balises :**
        - *Aucun*
2. Attendez la fin du déploiement, puis accédez à la ressource de tâche Stream Analytics déployée.

## Créez une entrée pour le flux d’événement

Votre travail Azure Stream Analytics doit obtenir des données d’entrée de l’Event Hub où les commandes client sont enregistrées.

1. Dans la page de vue d’ensemble **process-orders**, sélectionnez **Ajouter une entrée**. Puis dans la page **Entrées**, utilisez le menu **Ajouter une entrée de flux** pour ajouter une entrée **Event Hub** avec les propriétés suivantes :
    - **Alias d’entrée** : `orders`.
    - **Sélectionner un Event Hub dans vos abonnements** : sélectionné.
    - **Abonnement** : votre abonnement Azure.
    - **Espace de noms Event Hub** : sélectionnez l’espace de noms Event Hub **events*xxxxxxx***.
    - **Nom de l’Event Hub** : sélectionnez l’Event Hub **eventhub*xxxxxxx*** existant.
    - **Groupe de consommateurs Event Hub** : sélectionner le groupe de consommateurs **$Default** existant.
    - **Mode d’authentification** : créer une identité managée affectée par le système.
    - **Clé de partition** : *laisser vide*.
    - **Format de sérialisation de l’événement** : JSON.
    - **Encodage** : UTF-8
2. Enregistrez l’entrée et attendez qu’elle soit créée. Plusieurs notifications vont s’afficher. Attendez qu’une notification **Test de connexion réussi** s’affiche.

## Création d’une sortie pour le magasin d’objets blob

Vous allez stocker les données agrégées de commande client au format JSON dans un conteneur d’objets blob Stockage Azure.

1. Affichez la page **Sorties** pour le travail Stream Analytics **process-orders**. Utilisez ensuite le menu **Ajouter** pour ajouter une sortie **Stockage d’objets blob/ADLS Gen2** avec les propriétés suivantes :
    - **Alias de sortie** : `blobstore`.
    - **Sélectionner un stockage blob/ADLS Gen2 dans vos abonnements** : sélectionné.
    - **Abonnement** : votre abonnement Azure.
    - **Compte de stockage** : Sélectionner le compte de stockage **store*xxxxxxx***
    - **Conteneur** : Sélectionner le conteneur **données** existant
    - **Mode d'authentification** : Identité managée : Attribuée par le système
    - **Format de sérialisation de l’événement** : JSON.
    - **Format** : Ligne séparée
    - **Encodage** : UTF-8
    - **Mode écriture** : Ajout à l’arrivée des résultats
    - **Modèle de chemin d’accès** : `{date}`.
    - **Format de date** : AAAA/MM/JJ.
    - **Format d’heure** : *non applicable*.
    - **Nombre minimal de lignes** : 20.
    - **Durée maximale** : 0 heures, 1 minutes, 0 secondes
2. Enregistrez la sortie et attendez qu’elle soit créée. Plusieurs notifications vont s’afficher. Attendez qu’une notification **Test de connexion réussi** s’affiche.

## Créer une requête

Maintenant que vous avez défini une entrée et une sortie pour votre travail Azure Stream Analytics, vous pouvez utiliser une requête afin de sélectionner, filtrer et agréger des données à partir de l’entrée et envoyer les résultats à la sortie.

1. Affichez la page **Requête** pour le travail Stream Analytics **process-orders**. Patientez ensuite quelques instants jusqu’à ce que l’aperçu d’entrée s’affiche (en fonction des événements de commandes précédemment capturés dans l’Event Hub).
2. Notez que les données d’entrée incluent les champs **ProductID** et **Quantity** dans les messages envoyés par l’application cliente, ainsi que d’autres champs Event Hubs, notamment le champ **EventProcessedUtcTime** qui indique quand l’événement a été ajouté à l’Event Hub.
3. Modifiez la requête par défaut de la manière suivante :

    ```
    SELECT
        DateAdd(second,-10,System.TimeStamp) AS StartTime,
        System.TimeStamp AS EndTime,
        ProductID,
        SUM(Quantity) AS Orders
    INTO
        [blobstore]
    FROM
        [orders] TIMESTAMP BY EventProcessedUtcTime
    GROUP BY ProductID, TumblingWindow(second, 10)
    HAVING COUNT(*) > 1
    ```

    Observez que cette requête utilise **System-Timestamp** (basé sur le champ **EventProcessedUtcTime**) pour définir le début et la fin de chaque fenêtre *bascule* de 10 secondes (séquentielle sans chevauchement) dans laquelle la quantité totale pour chaque ID de produit est calculée.

4. Utilisez le bouton **&#9655; Tester la requête** pour valider la requête, puis vérifiez que l’état **Résultats des tests** indique **Réussite** (même si aucune ligne n’est retournée).
5. Enregistrez la requête.

## Exécuter le travail de diffusion en continu

OK. Vous êtes maintenant prêt à exécuter le travail et à traiter les données de commandes client en temps réel.

1. Consultez la page **Vue d’ensemble** pour le travail Stream Analytics **process-orders**, puis sous l’onglet **Propriétés**, passez en revue les champs **Entrées**, **Requête**, **Sorties** et **Fonctions** pour le travail. Si le nombre d’**Entrées** et de **Sorties** est 0, utilisez le bouton **&#8635; Actualiser** sur la page **Vue d’ensemble** pour afficher l’entrée **commandes** et la sortie **blobstore**.
2. Sélectionnez le bouton **▷ Démarrer** et démarrez la tâche de diffusion en continu maintenant. Attendez de recevoir la notification indiquant que la tâche de diffusion en continu a démarré avec succès.
3. Rouvrez le volet Cloud Shell, en le reconnectant si nécessaire, puis réexécutez la commande suivante pour envoyer une autre série de 1 000 commandes.

    ```
    node ~/dp-203/Allfiles/labs/17/orderclient
    ```

4. Pendant que l’application s’exécute, dans le Portail Azure, retournez à la page du groupe de ressources **dp203-*xxxxxxx***, puis sélectionnez le compte de stockage **store*xxxxxxxxxxxx***.
6. Dans le volet situé à gauche du volet du compte de stockage, sélectionnez l’onglet **Conteneurs**.
7. Ouvrez le conteneur **données**, puis utilisez le bouton **&#8635; Actualiser** pour actualiser la vue jusqu’à ce qu’un dossier s’affiche avec le nom de l’année en cours.
8. Dans le conteneur **données**, parcourez l’arborescence des dossiers qui comprend le dossier pour l’année en cours avec des sous-dossiers pour le mois et le jour.
9. Dans le dossier de l’heure, notez le fichier qui a été créé et qui devrait avoir un nom similaire à **0_xxxxxxxxxxxxxxxx.json**.
10. Dans le menu **...** du fichier (à droite des détails du fichier), sélectionnez **Afficher/modifier**, puis examinez le contenu du fichier qui doit se composer d’un enregistrement JSON pour chaque période de 10 secondes qui indique le nombre de commandes traitées par ID de produit, comme suit :

    ```
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":6,"Orders":13.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":8,"Orders":15.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":5,"Orders":15.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":1,"Orders":16.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":3,"Orders":10.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":2,"Orders":25.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":7,"Orders":13.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":4,"Orders":12.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":10,"Orders":19.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":9,"Orders":8.0}
    {"StartTime":"2022-11-23T18:16:35.0000000Z","EndTime":"2022-11-23T18:16:45.0000000Z","ProductID":6,"Orders":41.0}
    {"StartTime":"2022-11-23T18:16:35.0000000Z","EndTime":"2022-11-23T18:16:45.0000000Z","ProductID":8,"Orders":29.0}
    ...
    ```

11. Dans le volet Azure Cloud Shell, attendez la fin de l’application de la commande client.
12. Dans Portail Azure, actualisez à nouveau le fichier pour voir les résultats complets produits.
13. Revenez au groupe de ressources **dp203-*xxxxxxx***, puis rouvrez le travail **process-orders** Stream Analytics.
14. En haut de la page de la tâche Stream Analytics, utilisez le bouton **&#11036; Arrêter** pour arrêter la tâche, en confirmant lorsque vous y êtes invité.

## Supprimer les ressources Azure

Si vous avez fini d’explorer Azure Stream Analytics, vous devriez supprimer les ressources que vous avez créées afin d’éviter des coûts Azure inutiles.

1. Dans le portail Azure, dans la page **Accueil**, sélectionnez **Groupes de ressources**.
2. Sélectionnez le groupe de ressources **dp203-*xxxxxxx*** contenant vos ressources Stockage Azure, Event Hubs et Stream Analytics.
3. Au sommet de la page **Vue d’ensemble** de votre groupe de ressources, sélectionnez **Supprimer le groupe de ressources**.
4. Entrez le nom du groupe de ressources **dp203-*xxxxxxx*** pour confirmer que vous souhaitez le supprimer, puis sélectionnez **Supprimer**.

    Après quelques minutes, les ressources créées dans cet exercice seront supprimées.
