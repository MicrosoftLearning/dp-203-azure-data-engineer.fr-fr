---
lab:
  title: Ingérer des données en temps réel à l’aide d’Azure Stream Analytics et Azure Synapse Analytics
  ilt-use: Lab
---

# Ingérer des données en temps réel à l’aide d’Azure Stream Analytics et Azure Synapse Analytics

Les solutions d’analytique des données incluent souvent une exigence d’ingestion et de traitement *des flux* de données. Le traitement des flux diffère du traitement par lots en ce sens que les flux sont généralement *sans limite*, en d’autres termes, il s’agit de sources continues de données qui doivent être traitées perpétuellement plutôt qu’à intervalles fixes.

Azure Stream Analytics fournit un service cloud que vous pouvez utiliser pour définir une *requête* qui fonctionne sur un flux de données à partir d’une source de streaming, comme Azure Event Hubs ou Azure IoT Hub. Vous pouvez utiliser une requête Azure Stream Analytics pour ingérer le flux de données directement dans un magasin de données pour une analyse plus poussée, ou pour filtrer, agréger et synthétiser les données en fonction des fenêtres temporelles.

Dans cet exercice, vous allez utiliser Azure Stream Analytics pour traiter un flux de données de commandes, qui peut être généré à partir d’une application de vente au détail en ligne, par exemple. Les données de commande sont envoyées à Azure Event Hubs, d’où vos travaux Azure Stream Analytics lisent les données et les ingèrent dans Azure Synapse Analytics.

Cet exercice devrait prendre environ **45** minutes.

## Avant de commencer

Vous avez besoin d’un [abonnement Azure](https://azure.microsoft.com/free) dans lequel vous avez un accès administratif.

## Approvisionner des ressources Azure

Dans cet exercice, vous aurez besoin d’un espace de travail Azure Synapse Analytics avec accès au stockage Data Lake et d’un pool SQL dédié. Vous aurez également besoin d’un espace de noms Azure Event Hubs dans lequel les données de diffusion de commandes en continu peuvent être envoyées.

Vous allez utiliser la combinaison d’un script PowerShell et d’un modèle ARM pour approvisionner ces ressources.

1. Connectez-vous au [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com`.
2. Utilisez le bouton **[\>_]** à droite de la barre de recherche, en haut de la page, pour créer un environnement Cloud Shell dans le portail Azure, en sélectionnant un environnement ***PowerShell*** et en créant le stockage si vous y êtes invité. Cloud Shell fournit une interface de ligne de commande dans un volet situé en bas du portail Azure, comme illustré ici :

    ![Portail Azure avec un volet Cloud Shell](./images/cloud-shell.png)

    > **Remarque** : si vous avez déjà créé un interpréteur de commandes cloud qui utilise un environnement *Bash*, utilisez le menu déroulant en haut à gauche du volet de l’interpréteur de commandes cloud pour le remplacer par ***PowerShell***.

3. Notez que vous pouvez redimensionner le volet Cloud Shell en faisant glisser la barre de séparation en haut du volet. Vous pouvez aussi utiliser les icônes **&#8212;** , **&#9723;** et **X** situées en haut à droite du volet pour réduire, agrandir et fermer le volet. Pour plus d’informations sur l’utilisation d’Azure Cloud Shell, consultez la [documentation Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

4. Dans le volet PowerShell, entrez les commandes suivantes pour cloner le référentiel contenant cet exercice :

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Une fois que le référentiel a été cloné, entrez les commandes suivantes pour accéder au dossier de cet exercice et exécutez le script **setup.ps1** qu’il contient :

    ```
    cd dp-203/Allfiles/labs/18
    ./setup.ps1
    ```

6. Si vous y êtes invité, choisissez l’abonnement à utiliser (uniquement si vous avez accès à plusieurs abonnements Azure).
7. Quand vous y êtes invité, entrez un mot de passe approprié à définir pour votre pool SQL Azure Synapse.

    > **Remarque** : veillez à mémoriser ce mot de passe.

8. Attendez que le script se termine. Cette opération prend généralement environ 15 minutes, mais dans certains cas, elle peut être plus longue. Pendant que vous attendez, consultez l’article [Bienvenue dans Azure Stream Analytics](https://learn.microsoft.com/azure/stream-analytics/stream-analytics-introduction) dans la documentation Azure Stream Analytics.

## Ingérer des données de streaming dans un pool SQL dédié

Commençons par ingérer un flux de données directement dans une table dans un pool SQL dédié Azure Synapse Analytics.

### Afficher la source de streaming et la table de base de données

1. Une fois l’exécution du script d’installation terminée, réduisez le volet Cloud Shell (vous y retournerez ultérieurement). Ensuite, dans le portail Azure, accédez au groupe de ressources **dp203-*xxxxxxx*** qu’il a créé et notez que ce groupe de ressources contient un espace de travail Azure Synapse, un compte de stockage pour votre lac de données, un pool SQL dédié et un espace de noms Event Hubs.
2. Sélectionnez votre espace de travail Synapse et, dans sa page **Vue d’ensemble**, dans la carte **Ouvrir Synapse Studio**, sélectionnez **Ouvrir** pour ouvrir Synapse Studio dans un nouvel onglet de navigateur. Synapse Studio est une interface web que vous pouvez utiliser pour travailler avec votre espace de travail Synapse Analytics.
3. Sur le côté gauche de Synapse Studio, utilisez l’icône **&rsaquo;&rsaquo;** pour développer le menu. Cela permet d’afficher les différentes pages de Synapse Studio qui vous permettront de gérer les ressources et d’effectuer des tâches d’analytique de données.
4. Dans la page **Gérer**, dans la section **Pools SQL**, sélectionnez la ligne du pool SQL dédié **sql*xxxxxxx***, puis utilisez l’icône **▷** correspondante pour le reprendre.
5. Pendant que vous attendez que le pool SQL démarre, revenez à l’onglet du navigateur contenant le portail Azure et rouvrez le volet Cloud Shell.
6. Dans le volet Cloud Shell, entrez la commande suivante pour exécuter une application cliente qui envoie 100 commandes simulées à Azure Event Hubs :

    ```
    node ~/dp-203/Allfiles/labs/18/orderclient
    ```

7. Observez les données de commande envoyées ; chaque commande se compose d’un ID de produit et d’une quantité.
8. Une fois l’application cliente de commande terminée, réduisez le volet Cloud Shell et revenez à l’onglet du navigateur Synapse Studio.
9. Dans Synapse Studio, dans la page **Gérer**, vérifiez que l’état de votre pool SQL dédié est **En ligne**, puis basculez vers la page **Données** et, dans le volet **Espace de travail**, développez **Base de données SQL**, votre pool SQL **sql*xxxxxxx*** et **Tables** pour afficher la table **dbo.FactOrder**.
10. Dans le menu **...** de la table **dbo.FactOrder**, sélectionnez **Nouveau script SQL** > **Sélectionner les 100 premières lignes**, puis passez en revue les résultats. Notez que la table inclut des colonnes pour **OrderDateTime**, **ProductID** et **Quantity** , mais il n’y a actuellement aucune ligne de données.

### Créer un travail Azure Stream Analytics pour ingérer des données de commande

1. Revenez à l’onglet du navigateur contenant le portail Azure et notez la région où votre groupe de ressources **dp203-*xxxxxxx*** a été approvisionné. Vous allez créer votre travail Stream Analytics dans la <u>même région</u>.
2. Dans la page **Accueil**, sélectionnez **+ Créer une ressource** et recherchez `Stream Analytics job`. Créez ensuite une **Tâche Stream Analytics** avec les propriétés suivantes :
    - **Paramètres de base**:
        - **Abonnement** : votre abonnement Azure.
        - **Groupe de ressources** : sélectionnez le groupe de ressources **dp203-*xxxxxxx*** existant.
        - **Nom :** `ingest-orders`
        - **Région** : sélectionnez la <u>même région</u> que celle où votre espace de travail Synapse Analytics est approvisionné.
        - **Environnement d’hébergement** : cloud.
        - **Unités de diffusion en continu** : 1.
    - **Stockage** :
        - **Ajouter un compte de stockage** : sélectionné.
        - **Abonnement** : votre abonnement Azure.
        - **Comptes de stockage** : sélectionnez le compte de stockage **datalake*xxxxxxx***.
        - **Mode d’authentification** : chaîne de connexion.
        - **Sécuriser les données privées dans le compte de stockage** : sélectionné.
    - **Étiquettes** :
        - *Aucun*
3. Attendez la fin du déploiement, puis accédez à la ressource de tâche Stream Analytics déployée.

### Créer une entrée pour le flux de données d’événement

1. Dans la page de vue d’ensemble **ingest-orders**, sélectionnez la page **Entrées**. Utilisez le menu **Ajouter une entrée** pour ajouter une entrée **Event Hub** avec les propriétés suivantes :
    - **Alias d’entrée** : `orders`.
    - **Sélectionner un Event Hub dans vos abonnements** : sélectionné.
    - **Abonnement** : votre abonnement Azure.
    - **Espace de noms Event Hub** : sélectionnez l’espace de noms Event Hub **events*xxxxxxx***.
    - **Nom de l’Event Hub** : sélectionnez l’Event Hub **eventhub*xxxxxxx*** existant.
    - **Groupe de consommateurs Event Hub** : sélectionnez **Utiliser existant**, puis sélectionnez le groupe de consommateurs **$Default**.
    - **Mode d’authentification** : créer une identité managée affectée par le système.
    - **Clé de partition** : *laisser vide*.
    - **Format de sérialisation de l’événement** : JSON.
    - **Encodage** : UTF-8
2. Enregistrez l’entrée et attendez qu’elle soit créée. Plusieurs notifications vont s’afficher. Attendez qu’une notification **Test de connexion réussi** s’affiche.

### Créer une sortie pour la table SQL

1. Affichez la page **Sorties** de la tâche Stream Analytics **ingest-orders**. Utilisez ensuite le menu **Ajouter une sortie** pour ajouter une sortie **Azure Synapse Analytics** avec les propriétés suivantes :
    - **Alias de sortie** : `FactOrder`.
    - **Sélectionner Azure Synapse Analytics dans vos abonnements :** sélectionné.
    - **Abonnement** : votre abonnement Azure.
    - **Base de données** : sélectionnez la base de données **sql*xxxxxxx* (synapse*xxxxxxx *)**.
    - **Mode d’authentification** : authentification SQL Server.
    - **Nom d’utilisateur** : SQLUser.
    - **Mot de passe** : *mot de passe que vous avez spécifié pour votre pool SQL lors de l’exécution du script d’installation*.
    - **Table** : `FactOrder`
2. Enregistrez la sortie et attendez qu’elle soit créée. Plusieurs notifications vont s’afficher. Attendez qu’une notification **Test de connexion réussi** s’affiche.

### Créer une requête pour ingérer le flux d’événements

1. Affichez la page **Requête** de la tâche Stream Analytics **ingest-orders**. Patientez ensuite quelques instants jusqu’à ce que l’aperçu d’entrée s’affiche (en fonction des événements de commandes précédemment capturés dans l’Event Hub).
2. Notez que les données d’entrée incluent les champs **ProductID** et **Quantity** dans les messages envoyés par l’application cliente, ainsi que d’autres champs Event Hubs, notamment le champ **EventProcessedUtcTime** qui indique quand l’événement a été ajouté à l’Event Hub.
3. Modifiez la requête par défaut de la manière suivante :

    ```
    SELECT
        EventProcessedUtcTime AS OrderDateTime,
        ProductID,
        Quantity
    INTO
        [FactOrder]
    FROM
        [orders]
    ```

    Notez que cette requête prend les champs de l’entrée (Event Hub) et les écrit directement dans la sortie (table SQL).

4. Enregistrez la requête.

### Exécuter la tâche de diffusion en continu pour ingérer des données de commandes

1. Affichez la page **Vue d’ensemble** de la tâche Stream Analytics **ingest-orders** puis, sous l’onglet **Propriétés**, passez en revue les **Entrées**, **Requête**, **Sorties** et **Fonctions** de la tâche. Si le nombre d’**Entrées** et de **Sorties** est de 0, utilisez le bouton **↻ Actualiser** sur la page **Vue d’ensemble** pour afficher l’entrée **orders** et la sortie **FactTable**.
2. Sélectionnez le bouton **▷ Démarrer** et démarrez la tâche de diffusion en continu maintenant. Attendez de recevoir la notification indiquant que la tâche de diffusion en continu a démarré avec succès.
3. Rouvrez le volet de l’interpréteur de commandes cloud et réexécutez la commande suivante pour envoyer 100 commandes supplémentaires.

    ```
    node ~/dp-203/Allfiles/labs/18/orderclient
    ```

4. Pendant que l’application cliente de commande est en cours d’exécution, basculez vers l’onglet du navigateur Synapse Studio et affichez la requête que vous avez précédemment exécutée pour sélectionner les 100 premières lignes de la table **dbo.FactOrder**.
5. Utiliser le bouton **▷ Exécuter** pour réexécuter la requête et vérifier que la table contient désormais des données de commandes du flux d’événements (si ce n’est pas le cas, attendez une minute et réexécutez la requête). La tâche Stream Analytics envoie toutes les nouvelles données d’événements dans la table tant que la tâche est en cours d’exécution et que des événements de commandes sont envoyés à l’Event Hub.
6. Dans la page **Gérer**, suspendez le pool SQL dédié **sql*xxxxxxx*** (pour éviter des frais Azure inutiles).
7. Revenez à l’onglet du navigateur contenant le portail Azure et réduisez le volet de l’interpréteur de commandes cloud. Utilisez ensuite le bouton ** Arrêter** pour arrêter la tâche Stream Analytics et attendre la notification indiquant que la tâche Stream Analytics s’est arrêtée avec succès.

## Résumer les données de diffusion en continu dans un lac de données

Jusqu’à présent, vous avez vu comment utiliser une tâche Stream Analytics pour ingérer des messages d’une source de diffusion en continu dans une table SQL. Découvrons maintenant comment utiliser Azure Stream Analytics pour agréger des données sur des fenêtres temporelles ; ici, pour calculer la quantité totale de chaque produit vendu toutes les 5 secondes. Nous allons également découvrir comment utiliser un type de sortie différent pour la tâche en écrivant les résultats au format CSV dans un magasin d’objets blob de lac de données.

### Créer une tâche Azure Stream Analytics pour agréger les données de commandes

1. Dans le portail Azure, dans la page **Accueil**, sélectionnez **+ Créer une ressource** et recherchez `Stream Analytics job`. Créez ensuite une **Tâche Stream Analytics** avec les propriétés suivantes :
    - **Paramètres de base**:
        - **Abonnement** : votre abonnement Azure.
        - **Groupe de ressources** : sélectionnez le groupe de ressources **dp203-*xxxxxxx*** existant.
        - **Nom :** `aggregate-orders`
        - **Région** : sélectionnez la <u>même région</u> que celle où votre espace de travail Synapse Analytics est approvisionné.
        - **Environnement d’hébergement** : cloud.
        - **Unités de diffusion en continu** : 1.
    - **Stockage** :
        - **Ajouter un compte de stockage** : sélectionné.
        - **Abonnement** : votre abonnement Azure.
        - **Comptes de stockage** : sélectionnez le compte de stockage **datalake*xxxxxxx***.
        - **Mode d’authentification** : chaîne de connexion.
        - **Sécuriser les données privées dans le compte de stockage** : sélectionné.
    - **Étiquettes** :
        - *Aucun*

2. Attendez la fin du déploiement, puis accédez à la ressource de tâche Stream Analytics déployée.

### Créer une entrée pour les données de commandes brutes

1. Dans la page de vue d’ensemble **aggregate-orders**, sélectionnez la page **Entrées**. Utilisez le menu **Ajouter une entrée** pour ajouter une entrée **Event Hub** avec les propriétés suivantes :
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

### Créer une sortie pour le Data Lake Store

1. Affichez la page **Sorties** de la tâche Stream Analytics **aggregate-orders**. Utilisez ensuite le menu **Ajouter une sortie** pour ajouter une sortie **Stockage Blob/ADLS Gen2** avec les propriétés suivantes :
    - **Alias de sortie** : `datalake`.
    - **Sélectionner un stockage blob/ADLS Gen2 dans vos abonnements** : sélectionné.
    - **Abonnement** : votre abonnement Azure.
    - **Compte de stockage** : sélectionnez le compte de stockage **datalake*xxxxxxx***.
    - **Conteneur** : sélectionnez **Utiliser existant** puis, dans la liste, sélectionnez le conteneur **files**.
    - **Mode d’authentification** : chaîne de connexion.
    - **Format de sérialisation de l’événement** : CSV - Virgule (,).
    - **Encodage** : UTF-8
    - **Mode écriture** : ajouter, à mesure que des résultats arrivent.
    - **Modèle de chemin d’accès** : `{date}`.
    - **Format de date** : AAAA/MM/JJ.
    - **Format d’heure** : *non applicable*.
    - **Nombre minimal de lignes** : 20.
    - **Durée maximale** : 0 heures, 1 minutes, 0 secondes
2. Enregistrez la sortie et attendez qu’elle soit créée. Plusieurs notifications vont s’afficher. Attendez qu’une notification **Test de connexion réussi** s’affiche.

### Créer une requête pour agréger les données d’événements

1. Affichez la page **Requête** de la tâche Stream Analytics **aggregate-orders**.
2. Modifiez la requête par défaut de la manière suivante :

    ```
    SELECT
        DateAdd(second,-5,System.TimeStamp) AS StartTime,
        System.TimeStamp AS EndTime,
        ProductID,
        SUM(Quantity) AS Orders
    INTO
        [datalake]
    FROM
        [orders] TIMESTAMP BY EventProcessedUtcTime
    GROUP BY ProductID, TumblingWindow(second, 5)
    HAVING COUNT(*) > 1
    ```

    Notez que cette requête utilise l’élément **System.Timestamp** (basé sur le champ **EventProcessedUtcTime**) pour définir la fenêtre de début et de fin de chaque *fenêtre de bascule* (séquentielle sans chevauchement) de 5 secondes dans laquelle la quantité totale de chaque ID de produit est calculée.

3. Enregistrez la requête.

### Exécuter la tâche de diffusion en continu pour agréger les données de commandes

1. Affichez la page **Vue d’ensemble** de la tâche Stream Analytics **aggregate-orders** puis, sous l’onglet **Propriétés**, passez en revue les **Entrées**, **Requête**, **Sorties** et **Fonctions** de la tâche. Si le nombre d’**Entrées** et de **Sorties** est de 0, utilisez le bouton **↻ Actualiser**sur la page **Vue d’ensemble** pour afficher l’entrée **orders** et la sortie **datalake**.
2. Sélectionnez le bouton **▷ Démarrer** et démarrez la tâche de diffusion en continu maintenant. Attendez de recevoir la notification indiquant que la tâche de diffusion en continu a démarré avec succès.
3. Rouvrez le volet de l’interpréteur de commandes cloud et réexécutez la commande suivante pour envoyer 100 commandes supplémentaires :

    ```
    node ~/dp-203/Allfiles/labs/18/orderclient
    ```

4. Une fois l’application de commande terminée, réduisez le volet de l’interpréteur de commandes cloud. Basculez ensuite vers l’onglet du navigateur Synapse Studio puis, dans la page **Données**, sous l’onglet **Lié**, développez **Azure Data Lake Storage Gen2** > **synapse*xxxxxxx* (primary - datalake*xxxxxxx *)** et sélectionnez le conteneur **files (Primary)**.
5. Si le conteneur **files** est vide, attendez une minute ou plus, puis utilisez le bouton **↻ Actualiser** pour actualiser la vue. Finalement, un dossier nommé pour l’année en cours doit s’afficher. Il contient lui-même des dossiers pour le mois et le jour.
6. Sélectionnez le dossier de l’année et, dans le menu **Nouveau script SQL**, sélectionnez **Sélectionner les 100 premières lignes**. Définissez ensuite le **Type de fichier** sur **Format texte** et appliquez les paramètres.
7. Dans le volet de requête qui s’ouvre, modifiez la requête pour ajouter un paramètre `HEADER_ROW = TRUE`, comme illustré ici :

    ```sql
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/2023/**',
            FORMAT = 'CSV',
            PARSER_VERSION = '2.0',
            HEADER_ROW = TRUE
        ) AS [result]
    ```

8. Utiliser le bouton **▷ Exécuter** pour exécuter la requête SQL et afficher les résultats, qui affichent la quantité de chaque produit commandé en cinq secondes.
9. Revenez à l’onglet du navigateur contenant le portail Azure et utilisez le bouton ** Arrêter** pour arrêter la tâche Stream Analytics et attendre la notification indiquant que la tâche Stream Analytics s’est arrêtée avec succès.

## Supprimer les ressources Azure

Si vous avez fini d’explorer Azure Stream Analytics, vous devriez supprimer les ressources que vous avez créées afin d’éviter des coûts Azure inutiles.

1. Fermez l’onglet du navigateur Azure Synapse Studio et revenez dans le portail Azure.
2. Dans le portail Azure, dans la page **Accueil**, sélectionnez **Groupes de ressources**.
3. Sélectionnez le groupe de ressources **dp203-*xxxxxxx*** contenant vos ressources Azure Synapse, Event Hubs et Stream Analytics (et non le groupe de ressources managé).
4. Au sommet de la page **Vue d’ensemble** de votre groupe de ressources, sélectionnez **Supprimer le groupe de ressources**.
5. Entrez le nom du groupe de ressources **dp203-*xxxxxxx*** pour confirmer que vous souhaitez le supprimer, puis sélectionnez **Supprimer**.

    Après quelques minutes, les ressources créées dans cet exercice seront supprimées.
