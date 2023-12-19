---
lab:
  title: Créer un rapport en temps réel avec Azure Stream Analytics et Microsoft Power BI
  ilt-use: Suggested demo
---

# Créer un rapport en temps réel avec Azure Stream Analytics et Microsoft Power BI

Les solutions d’analytique des données incluent souvent une exigence d’ingestion et de traitement *des flux* de données. Le traitement des flux diffère du traitement par lots dans lequel les flux sont généralement *sans* limite , en d’autres termes, il s’agit de sources continues de données qui doivent être traitées perpétuellement plutôt qu’à intervalles fixes.

Azure Stream Analytics fournit un service cloud que vous pouvez utiliser pour définir une *requête* qui fonctionne sur un flux de données à partir d’une source de streaming, comme Azure Event Hubs ou Azure IoT Hub. Vous pouvez utiliser une requête Azure Stream Analytics pour traiter un flux de données et envoyer les résultats directement à Microsoft Power BI pour une visualisation en temps réel.

Dans cet exercice, vous allez utiliser Azure Stream Analytics pour traiter un flux de données de commandes, comme peut être généré à partir d’une application de vente au détail en ligne. Les données de commande sont envoyées à Azure Event Hubs, à partir de laquelle votre travail Azure Stream Analytics lit et récapitule les données avant de les envoyer à Power BI, où vous visualiserez les données dans un rapport.

Cet exercice devrait prendre environ **45** minutes

## Avant de commencer

Vous avez besoin d’un [abonnement Azure](https://azure.microsoft.com/free) dans lequel vous avez un accès administratif.

Vous aurez également besoin d’un accès au service Microsoft Power BI. Votre établissement scolaire ou votre organisation peut déjà fournir cet accès, ou vous pouvez vous [inscrire au service Power BI en tant qu’individu](https://learn.microsoft.com/power-bi/fundamentals/service-self-service-signup-for-power-bi).

## Approvisionner des ressources Azure

Dans cet exercice, vous aurez besoin d’un espace de travail Azure Synapse Analytics avec accès au stockage data lake et à un pool SQL dédié. Vous aurez également besoin d’un espace de noms Azure Event Hubs auquel les données de commande de streaming peuvent être envoyées.

Vous allez utiliser une combinaison d’un script PowerShell et d’un modèle ARM pour approvisionner ces ressources.

1. Connectez-vous au [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com`.
2. Utilisez le bouton **[\>_]** à droite de la barre de recherche, en haut de la page, pour créer un environnement Cloud Shell dans le portail Azure, en sélectionnant un environnement ***Bash*** et en créant le stockage si vous y êtes invité. Cloud Shell fournit une interface de ligne de commande dans un volet situé en bas du portail Azure, comme illustré ici :

    ![Capture d’écran du Portail Azure avec un volet Cloud Shell.](./images/cloud-shell.png)

    > **Remarque** : Si vous avez créé un interpréteur de commandes cloud qui utilise un *environnement Bash* , utilisez le menu déroulant en haut à gauche du volet Cloud Shell pour le remplacer par ***PowerShell***.

3. Notez que vous pouvez redimensionner le volet Cloud Shell en faisant glisser la barre de séparation en haut du volet. Vous pouvez aussi utiliser les icônes **&#8212;** , **&#9723;** et **X** situées en haut à droite du volet pour réduire, agrandir et fermer le volet. Pour plus d’informations sur l’utilisation d’Azure Cloud Shell, consultez la [documentation Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. Dans le volet PowerShell, entrez les commandes suivantes pour cloner le dépôt contenant cet exercice :

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Une fois que le dépôt a été cloné, entrez les commandes suivantes pour accéder au dossier de ce labo et exécutez le script **setup.sh** qu’il contient :

    ```
    cd dp-203/Allfiles/labs/19
    ./setup.ps1
    ```

6. Si vous y êtes invité, choisissez l’abonnement que vous souhaitez utiliser (cela se produit uniquement si vous avez accès à plusieurs abonnements Azure).

7. Pendant que vous attendez que le script se termine, passez à la tâche suivante.

## Création d’un espace de travail Power BI

Dans le service Power BI, vous organisez des jeux de données, des rapports et d’autres ressources dans *les* espaces de travail. Chaque utilisateur Power BI a un espace de travail par défaut nommé **Mon espace de** travail, que vous pouvez utiliser dans cet exercice, mais il est généralement recommandé de créer un espace de travail pour chaque solution de création de rapports discrète que vous souhaitez gérer.

1. Connectez-vous au service Power BI à [https://app.powerbi.com/](https://app.powerbi.com/) l’aide de vos informations d’identification de service Power BI.
2. Dans la barre de menus à gauche, sélectionnez **Espaces de travail** (l’icône ressemble à &#128455;).
3. Créez un espace de travail avec un nom explicite (par exemple, *mslearn-streaming*), en sélectionnant le **mode de licence Pro** .

    > **Remarque** : Si vous utilisez un compte d’évaluation, vous devrez peut-être activer d’autres fonctionnalités d’évaluation.

4. Lorsque vous affichez votre espace de travail, notez son identificateur global unique (GUID) dans l’URL de la page (qui doit être similaire à `https://app.powerbi.com/groups/<GUID>/list`). Vous aurez besoin de ce GUID ultérieurement.

## Traité des données de streaming avec Azure Stream Analytics

Un travail Azure Stream Analytics définit une requête perpétuelle qui fonctionne sur les données de streaming à partir d’une ou plusieurs entrées et envoie les résultats à une ou plusieurs sorties.

### Création d’un travail Stream Analytics

1. Revenez à l’onglet du navigateur contenant le Portail Azure et, lorsque le script est terminé, notez la région où votre **groupe de ressources dp203-*xxxxxxx*** a été provisionné.
2. Connectez-vous au **portail Azure** et cliquez sur la page **Accueil**, puis sur Créer une ressource. Créez ensuite un **travail** Stream Analytics avec les propriétés suivantes :
    - **Abonnement** : votre abonnement Azure.
    - Sélectionnez le groupe de ressources existant [nom du groupe de ressources de bac à sable].
    - **Nom :** `stream-orders`
    - **Région** : sélectionnez la région où votre espace de travail Synapse Analytics est provisionné.
    - Environnement d’hébergement cloud
    - Unités de streaming TJ
3. Attendez la fin du déploiement, puis accédez à la ressource déployée.

### Créer un Event Hub pour l’entrée de diffusion en continu

1. Dans la **page vue d’ensemble des commandes** de flux, sélectionnez la **page Entrées** et utilisez le **menu Ajouter une **entrée** Event Hub** avec les propriétés suivantes :
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

### Créer une sortie pour Power BI

1. Affichez la **page Sorties** pour le **travail Stream Analytics stream-orders** . Utilisez ensuite le **menu Ajouter une sortie** pour ajouter une **sortie Power BI** avec les propriétés suivantes :
    - Alias de sortie :
    - **Sélectionner manuellement** les paramètres Power BI : Sélectionné
    - **Espace de travail** de groupe : *GUID de votre espace de travail*
    - **Mode** d’authentification : *sélectionnez *** le jeton** *utilisateur, puis utilisez le *** bouton Autoriser** *en bas pour vous connecter à votre compte Power BI.*
    - Nom du jeu de données :
    - Nom de la table :

2. Enregistrez la sortie et attendez qu’elle soit créée. Vous verrez plusieurs notifications. Attendez une notification de test** de **connexion réussie.

### Créer une requête pour résumer le flux d’événements

1. Affichez la **page Requête** pour le **travail Stream Analytics stream-orders** .
2. Modifiez la requête de la manière suivante :

    ```
    SELECT
        DateAdd(second,-5,System.TimeStamp) AS StartTime,
        System.TimeStamp AS EndTime,
        ProductID,
        SUM(Quantity) AS Orders
    INTO
        [powerbi-dataset]
    FROM
        [orders] TIMESTAMP BY EventEnqueuedUtcTime
    GROUP BY ProductID, TumblingWindow(second, 5)
    HAVING COUNT(*) > 1
    ```

    Notez que cette requête utilise l’élément **System.Timestamp** (basé sur le **champ EventEnqueuedUtcTime** ) pour définir la fenêtre de début et de fin de chaque *fenêtre de bascule* (séquentiel sans chevauchement) dans laquelle la quantité totale de chaque ID de produit est calculée.

3. Enregistrez la requête.

### Exécuter le travail de diffusion en continu pour traiter les données de commande

1. Affichez la **page Vue d’ensemble** du **travail Stream Analytics de flux de commandes** et, sous l’onglet **Propriétés** , passez en revue les **entrées**, **les requêtes**, **les sorties** et **les fonctions** du travail. Si le nombre d’entrées et de **sorties** est 0, utilisez le **&#8635 **;** Bouton Actualiser** sur la **page Vue d’ensemble** pour afficher l’entrée des commandes** et **la **sortie powerbi-dataset**.
2. Sélectionnez le **&#9655 ; Bouton Démarrer** et démarrer la tâche de diffusion en continu maintenant. Attendez la notification indiquant que la tâche de diffusion en continu a démarré avec succès.
3. Ouvrez à nouveau le volet Cloud Shell et exécutez la commande suivante pour envoyer 100 commandes.

    ```
    node ~/dp-203/Allfiles/labs/19/orderclient
    ```

4. Pendant que l’application cliente de commande est en cours d’exécution, basculez vers l’onglet navigateur de l’application Power BI et affichez votre espace de travail.
5. Actualisez la page de l’application Power BI jusqu’à ce que vous voyiez le **jeu de données** en temps réel dans votre espace de travail. Ce jeu de données est généré par le travail Azure Stream Analytics.

## Visualiser les données dans un rapport Power BI

Maintenant que vous disposez d’un jeu de données pour les données d’ordre de diffusion en continu, vous pouvez créer un tableau de bord Power BI qui le représente visuellement.

1. Revenez à votre onglet de navigateur PowerBI.

2. Dans le **menu déroulant + Nouveau** pour votre espace de travail, sélectionnez **Tableau de bord** et créez un tableau de bord nommé **Order Tracking**.

3. Dans le **tableau de bord Suivi** des commandes, sélectionnez le **&#9999 ; &#65039; Menu Modifier** , puis sélectionner **+ Ajouter une vignette**. Dans la page **Ajouter une vignette**, sélectionnez **Données de streaming personnalisées**, puis **Suivant**.

4. Dans le **volet Ajouter une vignette** de données de streaming personnalisée, sous **Vos jeux de données, sélectionnez le **jeu de** données** en temps réel, puis sélectionnez **Suivant**.

5. Remplacez le type **de visualisation par défaut par graphique en courbes**. Sélectionnez les propriétés suivantes, puis sélectionnez Enregistrer :
    - **Axe** : EndTime
    - **Valeur** : Commandes
    - Fenêtre de temps à afficher

6. Dans le volet Détails** de la **vignette, définissez le **titre** sur **Nombre de commandes** en temps réel, puis sélectionnez **Appliquer**.

7. Revenez à l’onglet du navigateur contenant le Portail Azure et, si nécessaire, rouvrez le volet Cloud Shell. Réexécutez ensuite la commande suivante pour envoyer 100 commandes.

    ```
    node ~/dp-203/Allfiles/labs/19/orderclient
    ```

8. Pendant que le script de soumission de commandes est en cours d’exécution, revenez à l’onglet du navigateur contenant le **tableau de bord Power BI Suivi** des commandes et observez que la visualisation est mise à jour pour refléter les nouvelles données d’ordre telles qu’elles sont traitées par le travail Stream Analytics (qui doit toujours être en cours d’exécution).

    ![Capture d’écran d’un rapport Power BI montrant un flux de données de commande en temps réel.](./images/powerbi-line-chart.png)

    Vous pouvez réexécuter le **script orderclient** et observer les données capturées dans le tableau de bord en temps réel.

## Supprimer des ressources

Si vous avez fini d’explorer Azure Synapse Analytics, vous devriez supprimer les ressources que vous avez créées afin d’éviter des coûts Azure inutiles.

1. Fermez l’onglet du navigateur contenant le rapport Power BI. Ensuite, dans le **volet Espaces de** travail, dans le **menu &#8942 ;** de votre espace de travail, sélectionnez **Paramètres** de l’espace de travail et supprimez l’espace de travail.
2. Revenez à l’onglet du navigateur contenant le portail Azure, fermez le volet Cloud Shell et utilisez le **fichier &#128454 ; Bouton Arrêter** pour arrêter le travail Stream Analytics. Attendez la notification indiquant que la tâche de diffusion en continu a démarré avec succès.
3. Dans le portail Azure, dans la page **Accueil**, sélectionnez **Groupes de ressources**.
4. Sélectionnez le **groupe de ressources dp203-*xxxxxxx*** contenant vos ressources Azure Event Hub et Stream Analytics.
5. Au sommet de la page **Vue d’ensemble** de votre groupe de ressources, sélectionnez **Supprimer le groupe de ressources**.
6. Entrez le nom du groupe de ressources pour confirmer que vous souhaitez le supprimer, puis sélectionnez Supprimer.

    Après quelques minutes, les ressources créées dans cet exercice seront supprimées.
