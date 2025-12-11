# Azure Pipeline

L’objectif était de construire une pipeline complète partant d’une base OLTP dans PostgreSQL, puis d’orchestrer son ingestion vers un Data Lake (ADLS2) via Azure Data Factory, de la transformer dans Azure Synapse Analytics, et enfin de la visualiser dans Power BI.
Avant d’entrer dans le pipeline, j’ai dû créer et configurer plusieurs services Azure.

### J'ai utilisé le service Azure Database for PostgreSQL – Flexible Server, donc créé un serveur PostgreSQL managé. Pourquoi ? Pas besoin d’une VM : Azure gère tout
Pour cela, j’ai autorisé mon adresse IP à accéder au serveur, puis installé psql et configuré les variables d’environnement afin de me connecter depuis mon terminal local.
Ainsi j’ai pu créer la base OLTP qui sert de point de départ au pipeline.

### Et pour accueillir cette nouvelle base, le service Azure Data Lake Storage Gen2 (ADLS2) était approprié car il supporte les formats optimisés pour Synapse et Power BI somme Parquet. Il est simple à gérer et peu coûteux.
Celui-ci m’a servi de :
zone d’atterrissage des données extraites depuis PostgreSQL (Bronze layer),
zone d’entrée pour Synapse.

### Une fois stockées, j’ai utilisé Azure Synapse Analytics (OLAP) pour transformer les données et les exploiter ensuite sous Power BI
Pour cela, il a fallu créer un SQL Dedicated Pool, qui est une base SQL indépendante avec ses propres paramètres.
Plusieurs étapes ont été nécessaires : création d’un login sur le serveur SQL du workspace (dans la base master) / création du user correspondant dans le dedicated pool /
attribution des permissions sur ce pool.
J’ai ensuite créé une table interne afin de stocker les données provenant d’ADLS, puis utilisées pour le reporting.

### Une simple connexion de l’endpoint du workspace, du login et du user dans Power BI suffit : nous pouvons alors visualiser les données comme souhaité !

### Enfin, pour orchestrer tous ces services j’ai utilisé Azure Data Factory
ADF offre des connecteurs natifs pour PostgreSQL et ADLS, un système de scheduling avec triggers, et une intégration simple avec Synapse.
J’ai configuré trois connexions sécurisées (Linked Services) permettant à ADF de communiquer avec : PostgreSQL, ADLS2, Synapse Analytics.

J’ai ensuite construit une pipeline contenant deux activités Copy Data :
une première activité avec pour source : PostgreSQL managé, et pour destination : ADLS Gen2 (Parquet),
une seconde activité avec pour source : ADLS2, et pour destination : Synapse Analytics afin de charger les données dans le SQL Dedicated Pool.

La pipeline peut être déclenchée manuellement, via un trigger planifié, ou depuis Airflow grâce à AzureDataFactoryRunPipelineOperator.

Ce projet m’a permis d’apprendre :
-comment manipuler un PostgreSQL managé sur Azure
-comment orchestrer une ingestion avec Data Factory
-comment utiliser un SQL Pool dédié dans Synapse
-comment connecter tous ces services entre eux via les Linked Services
-comment concevoir une architecture cloud complète
