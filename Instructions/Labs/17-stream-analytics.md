---
lab:
  title: Bien démarrer avec Azure Stream Analytics
  ilt-use: Suggested demo
---

# Bien démarrer avec Azure Stream Analytics

Dans cet exercice, vous allez provisionner un espace de travail Azure Stream Analytics dans votre abonnement Azure et l’utiliser pour traiter un flux de données en temps réel.

Cet exercice devrait prendre environ **30** minutes.

## Avant de commencer

Vous avez besoin d’un [abonnement Azure](https://azure.microsoft.com/free) dans lequel vous avez un accès administratif.

## Approvisionner des ressources Azure

Dans cet exercice, vous allez capturer un flux de données de transaction de ventes simulées, les traiter et stocker les résultats dans un conteneur d’objets blob dans Stockage Azure. Vous aurez besoin d’un espace de noms Azure Event Hubs auquel les données de diffusion en continu peuvent être envoyées et d’un compte Stockage Azure dans lequel les résultats du traitement de flux seront stockés.

Vous allez utiliser une combinaison d’un script PowerShell et d’un modèle ARM pour approvisionner ces ressources.

1. Connectez-vous au [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com`.
2. Utilisez le bouton **[\>_]** à droite de la barre de recherche, en haut de la page, pour créer un environnement Cloud Shell dans le portail Azure, en sélectionnant un environnement ***Bash*** et en créant le stockage si vous y êtes invité. Cloud Shell fournit une interface de ligne de commande dans un volet situé en bas du portail Azure, comme illustré ici :

    ![Portail Azure avec un volet Cloud Shell](./images/cloud-shell.png)

    > **Remarque** : Si vous avez créé un interpréteur de commandes cloud qui utilise un *environnement Bash* , utilisez le menu déroulant en haut à gauche du volet Cloud Shell pour le remplacer par ***PowerShell***.

3. Notez que vous pouvez redimensionner le volet Cloud Shell en faisant glisser la barre de séparation en haut du volet. Vous pouvez aussi utiliser les icônes **&#8212;** , **&#9723;** et **X** situées en haut à droite du volet pour réduire, agrandir et fermer le volet. Pour plus d’informations sur l’utilisation d’Azure Cloud Shell, consultez la [documentation Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. Dans le volet PowerShell, entrez les commandes suivantes pour cloner le dépôt contenant cet exercice :

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Une fois que le dépôt a été cloné, entrez les commandes suivantes pour accéder au dossier de ce labo et exécutez le script **setup.sh** qu’il contient :

    ```
    cd dp-203/Allfiles/labs/17
    ./setup.ps1
    ```

6. Si vous y êtes invité, choisissez l’abonnement que vous souhaitez utiliser (cela se produit uniquement si vous avez accès à plusieurs abonnements Azure).
7. Attendez que le script se termine, ce qui prend généralement entre 5 et 10 minutes. Pendant que vous attendez, consultez l’article [Bienvenue dans Azure Stream Analytics](https://learn.microsoft.com/azure/stream-analytics/stream-analytics-introduction) dans la documentation Azure Stream Analytics.

## Afficher la source de données de streaming

Avant de créer un travail Azure Stream Analytics pour traiter les données en temps réel, examinons le flux de données qu’il devra interroger.

1. Une fois le script d’installation terminé, redimensionnez ou réduisez le volet Cloud Shell afin de voir les Portail Azure (vous revenez ultérieurement à Cloud Shell). Ensuite, dans le Portail Azure, accédez au **groupe de ressources dp203-*xxxxxxx*** qu’il a créé et notez que ce groupe de ressources contient un compte Stockage Azure et un espace de noms Event Hubs.

    Notez l’emplacement **** où les ressources ont été approvisionnées . Plus tard, vous allez créer un travail Azure Stream Analytics dans le même emplacement.

2. Ouvrez à nouveau le volet Cloud Shell et entrez la commande suivante pour exécuter une application cliente qui envoie 100 commandes simulées à Azure Event Hubs :

    ```
    node ~/dp-203/Allfiles/labs/17/orderclient
    ```

3. Observez les données des commandes commerciales telles qu’elles sont envoyées : chaque commande se compose d’un ID de produit et d’une quantité. L’application se termine après l’envoi de 1 000 commandes, ce qui prend une minute.

## Créer une tâche Azure Stream Analytics

Vous êtes maintenant prêt à créer un travail Azure Stream Analytics pour traiter les données de transaction de vente à mesure qu’elles arrivent dans le hub d’événements.

1. Dans le Portail Azure, dans la **page dp203-*xxxxxxx***, sélectionnez **+ Créer** et rechercher `Stream Analytics job`. Créez ensuite un **travail** Stream Analytics avec les propriétés suivantes :
    - **Paramètres de base**:
        - **Abonnement** : votre abonnement Azure.
        - Sélectionnez le groupe de ressources existant [nom du groupe de ressources de bac à sable].
        - **Nom :** `process-orders`
        - **Région** : sélectionnez la région où vos autres ressources Azure sont approvisionnées.
        - Environnement d’hébergement cloud
        - Unités de streaming TJ
    - **Stockage** :
        - **Ajouter un compte** de stockage : non sélectionné
    - **Étiquettes** :
        - *Aucun*
2. Attendez la fin du déploiement, puis accédez à la ressource déployée.

## Créer un Event Hub pour l’entrée de diffusion en continu

Votre travail Azure Stream Analytics doit obtenir des données d’entrée à partir du hub d’événements où les commandes sont enregistrées.

1. Dans la page vue d’ensemble **des commandes** de processus, sélectionnez **Ajouter une entrée**. Ensuite, dans la **page Entrées** , utilisez le **menu Ajouter une entrée** de flux pour ajouter une **entrée Event Hub** avec les propriétés suivantes :
    - Alias d’entrée :
    - **Sélectionner un Event Hub dans vos abonnements :** sélectionné.
    - **Abonnement** : votre abonnement Azure.
    - **Espace de noms** Event Hub : sélectionnez l’espace **de noms Event Hubs events*xxxxxxx***
    - **Nom** du hub d’événements : sélectionnez le hub d’événements eventhub*xxxxxxx *** existant**.
    - **Groupe** de consommateurs Event Hub : sélectionner le groupe de consommateurs $Default** existant **
    - **Type d’authentification** : Identité managée affectée par le système
    - **Clé** de partition : *laissez vide*
    - **Format de sérialisation de l’événement** :
    - **Encodage** : UTF-8
2. Enregistrez l’entrée et attendez qu’elle soit créée. Vous verrez plusieurs notifications. Attendez une notification de test** de **connexion réussie.

## Créer une sortie pour le magasin d’objets blob

Vous allez stocker les données de commande commerciale agrégées au format JSON dans un conteneur d’objets blob Stockage Azure.

1. Affichez la **page Sorties du **travail Stream Analytics des commandes**** de processus. Utilisez ensuite le **menu Ajouter** pour ajouter une **sortie Stockage Blob/ADLS Gen2** avec les propriétés suivantes :
    - Alias de sortie :
    - **Sélectionnez Sélectionner le stockage d’objets blob/ADLS Gen2 dans vos abonnements à partir de vos abonnements** : sélectionné
    - **Abonnement** : votre abonnement Azure.
    - **** compte Stockage : sélectionnez le **compte de stockage store*xxxxxxx***
    - **Conteneur** : sélectionner le conteneur de données** existant **
    - **Type d’authentification** : Identité managée affectée par le système
    - **Format de sérialisation de l’événement** :
    - **FORMAT**: séparé par une ligne
    - **Encodage** : UTF-8
    - **Mode** d’écriture : Ajouter à mesure que les résultats arrivent
    - Modèle de chemin d’accès
    - **Format** de date : AAAA/MM/DD
    - **Format** d’heure : *non applicable*
    - Nombre minimal de lignes
    - **Durée** maximale : 0 heures, 1 minutes, 0 secondes
2. Enregistrez la sortie et attendez qu’elle soit créée. Vous verrez plusieurs notifications. Attendez une notification de test** de **connexion réussie.

## Créer une requête

Maintenant que vous avez défini une entrée et une sortie pour votre travail Azure Stream Analytics, vous pouvez utiliser une requête pour sélectionner, filtrer et agréger des données à partir de l’entrée et envoyer les résultats à la sortie.

1. Affichez la **page Requête** pour le **travail Stream Analytics des commandes** de processus. Patientez ensuite quelques instants jusqu’à ce que l’aperçu d’entrée s’affiche (en fonction des événements de commandes précédemment capturés dans le hub d’événements).
2. Notez que les données d’entrée incluent les **champs ProductID** et **Quantity** dans les messages envoyés par l’application cliente, ainsi que d’autres champs Event Hubs, y compris le **champ EventProcessedUtcTime** qui indique quand l’événement a été ajouté au hub d’événements.
3. Modifiez la requête de la manière suivante :

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

    Notez que cette requête utilise l’élément **System.Timestamp** (basé sur le **champ EventProcessedUtcTime**) pour définir la fenêtre de début et de fin de chaque fenêtre de bascule* de 10 secondes *(séquentiel sans chevauchement) dans laquelle la quantité totale de chaque ID de produit est calculée.

4. Utiliser le **&#9655 ; Bouton Tester la requête** pour valider la requête et vérifier que l’état résultats** du **test indique **Réussite** (même si aucune ligne n’est retournée).
5. Enregistrez la requête.

##   Démarrer le travail de streaming

OK, vous êtes maintenant prêt à exécuter le travail et à traiter des données de commande en temps réel.

1. Affichez la **page Vue d’ensemble** du **travail Stream Analytics des commandes** de processus et, sous l’onglet **Propriétés** , passez en revue les **entrées**, **les requêtes**, **les sorties** et **les fonctions** du travail. Si le nombre d’entrées et de **sorties** est 0, utilisez le **&#8635 **;** Bouton Actualiser** sur la **page Vue d’ensemble** pour afficher les commandes** d’entrée et **la **sortie du magasin** d’objets blob.
2. Sélectionnez le **&#9655 ; Bouton Démarrer** et démarrer la tâche de diffusion en continu maintenant. Attendez la notification indiquant que la tâche de diffusion en continu a démarré avec succès.
3. Ouvrez à nouveau le volet Cloud Shell, reconnectez-vous si nécessaire, puis réexécutez la commande suivante pour envoyer 1 000 commandes.

    ```
    node ~/dp-203/Allfiles/labs/17/orderclient
    ```

4. Pendant que la simulation est en cours, retournez dans le Portail Azure. Revenez à la page pour le groupe de ressources **learn-*xxxxxxxxxxxxxxxxx...***, et sélectionnez le compte de stockage **store*xxxxxxxxxxxx***.
6. Dans le volet situé à gauche du volet du compte de stockage, sélectionnez l’onglet **Conteneurs**.
7. Ouvrez le **conteneur de données** et utilisez le **fichier &#8635 ; Bouton Actualiser** pour actualiser l’affichage jusqu’à ce que vous voyiez un dossier portant le nom de l’année en cours.
8. Dans le conteneur de **données**, parcourez l’arborescence des dossiers, qui comprend un dossier pour l’année en cours, avec des sous-dossiers pour le mois, le jour et l’heure.
9. Dans le dossier de l’heure, notez le fichier qui a été créé et qui devrait avoir un nom similaire à **0_xxxxxxxxxxxxxxxx.json**.
10. Dans le menu **...** du fichier (à droite des détails du fichier), sélectionnez **Afficher/modifier**, puis examinez le contenu du fichier qui devrait se composer d’un enregistrement JSON pour chaque période de 10 secondes, indiquant le nombre de messages reçus des appareils IoT, comme suit :

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

11. Dans le volet Azure Cloud Shell, attendez que l’application cliente de commande se termine.
12. De retour sur le Portail Azure, actualisez le fichier une fois de plus pour voir l’ensemble complet des résultats qui ont été produits pendant la simulation.
13. Retournez dans le groupe de ressources **learn-*xxxxxxxxxxxxx....***, et rouvrez la tâche Stream Analytics **stream*xxxxxxxxxxxxx***.
14. En haut de la page de la tâche Stream Analytics, utilisez le bouton **&#11036; Arrêter** pour arrêter la tâche, en confirmant lorsque vous y êtes invité.

## Supprimer les ressources Azure

Si vous avez fini d’explorer Azure Synapse Analytics, vous devriez supprimer les ressources que vous avez créées afin d’éviter des coûts Azure inutiles.

1. Dans le portail Azure, dans la page **Accueil**, sélectionnez **Groupes de ressources**.
2. Sélectionnez le **groupe de ressources dp203-*xxxxxxx*** contenant vos ressources Stockage Azure, Event Hubs et Stream Analytics.
3. Au sommet de la page **Vue d’ensemble** de votre groupe de ressources, sélectionnez **Supprimer le groupe de ressources**.
4. Entrez le nom du groupe de ressources pour confirmer que vous souhaitez le supprimer, puis sélectionnez Supprimer.

    Après quelques minutes, les ressources créées dans cet exercice seront supprimées.
