
---

# 🗺️ Project Phases & Guide

## 🏗️ Phase1 - Project Initialization

<aside>

**Goal**: Préparation des étapes de développement à travers des **Backlogs**, **Users stories** et **Sprints** en utilisant **notion** ou **jira**.

</aside>

- [ ]  **Environment setup**
    - [ ]  **Setup Azure**
        - [ ]  Use new email get 30 days free with $200 credit
        - [ ]  Create Resource Group
        - [ ]  Create Storage Account
          - [ ]  Choisir un nom unique
         -  [ ]  Choisir "Big Data Analytics" pour Primary workload
         -  [ ]  Choisir LRS pour Redondancy
         -  [ ]  Bien vérifier que la coche **Enable Hierarchical namespace** est bien activée
         -  [ ]  vérifier dans Resource group que le storage account est bien créé et disponible(update de azure)
            
        - [ ]  Create Databricks workspace
           - [ ]  Pour Pricing tier en entreprise on choisi "Premium" et "Trial" en mode gratuit
           - [ ]  La configuration du Managed resource group est très important ici car elle définit les ressources de calcul à utiliser par Databricks(Disk, virtual network, Ram); il faut lui donné un nom explicite(man_databricks);On donne juste le nom mais les compute resources seront gérées par Databricks contrairement aux projets traditionnels ou azure fournissait les ressources de calcul(compute) pour Databricks
           - [ ]  On vérifie dans Resource group si le workspace Databricks est bien créé
           - [ ]  Lancer le Databricks workspace pour accéder au UI(Interface Utilisateur)
        - [ ]  Création de Unity Catalog metastore
             - [ ]  On ne peut pas créer Unity Catalog metastore directement dans l'onglet Catalog mais on passe par **Account**
             - [ ]  Cliquez sur le coin en haut à droite puis sur **Manage account** qui est un console pour gérer Databricks.
             - [ ]  Ce console(**Manage account**) n'est disponible que pour les **ADMIN**
             - [ ]  Par défaut l'email disponible dans Azure databricks workspace ne nous offre pas le rôle admin; l'admin est votre compte user disponible dans Entra ID (active directory d'azure) qu'il faut récupérer depuis Azure Entra ID ---> Users ---> copiez l'identifiant EXT azure pour se connecter au databricks account(azure databricks console login) via le lien **https://accounts.azuredatabricks.net/login?tuuid=d35690e2-fe1b-40ab-a8d6-437611c15183**
             - [ ]  Une fois reconnecter avec le l'identifiant EXT qui est admin par défaut on attribut à l'identifiant présent sur Databricks le role d'ADMIN
             - [ ]  Aller dans User Management cliquer sur user referent à mon email normal(diffèrent du EXT) et lui conférer le role d'admin
             - [ ]  On retourne dans Azure databricks workspace pour retrouver **Manage account**
             - [ ]  Metastore doit être lié à ADLS pour lire ou écrire les données sur ADSL depuis le Matastore de Databricks
             - [ ]  avant de créer le metastore depuis **Manage account** on configure un connector(access connector)
         - [ ]  Créer Access connector for azure databricks depuis Azure
         - [ ]  Permettre Access connector for azure databricks d'accéder à ADLS depuis IAM storage account
              - [ ]  Clique sur Storage account
              - [ ]  Go to Access Control(IAM)
              - [ ]  Add role assignment & choose Storage blob data contributor
              - [ ]  Choice managed identity & ajouter comme membre à ce rôle le Access connector qu'on avait créé
              - [ ]  Le Access connector dispose désormais un rôle de contributeur pour faire ce qu'on veut sur ADLS
         - [ ]  Créer un container nommé metastore dans ADLS
         - [ ]  On retourne maintenant dans **Manage account** d'azure databricks pour **créer le metastore** de unity catalog
              - [ ]   Il faut toujours préciser le ADLS GEN2 Path lors de la création du métastore sinon on sera obligé de le renseigner à chaque fois qu'on créera un catalog  (**metastore@adls_account.dfs.core.windows.net/**)
              - [ ]   Il faut aussi renseigner le access connector id  disponible dans Access connector lors de la création du metastore
              - [ ]   Assigner le ou les workspaces à ce metastore
              - [ ]   Le compte email avec EXT est toujours ADMIN du metastore il faut le changer pour le rôle d'admin au compte email normal(créer un groupe admin et attribué le role d'dmin)
                  


      
        - [ ]  File system name just make it something relevant and meaningful
        - [ ]  Create Key Vault
    - [ ]  **Setup SQL On-Prem**
        - [ ]  Download sql server
        - [ ]  download ssms(if no work, sql server config mgr and then run the service)
        - [ ]  download adventureworks
**move to C:\Program Files\Microsoft SQL Server\MSSQL16.SQLEXPRESS\MSSQL\Backup**
        - [ ]  restore db
[[ config mgr if you shut down ]]
        - [ ]  create login sql script to get username and password
        - [ ]  execute in correct db (might need to load again)
        - [ ]  give user permissions via role on LHS
        - [ ]  create key vault -> create secret username, password
**(issue: The operation is not allowed by RBAC. If role assignments were recently changed, please wait several minutes for role assignments to become effective.
solution:**
            - [ ]  step1: select the Resource group where creating Azure Key Vault -> select "Access Control(IAM) ->Add "Add role assignment" and for Role search for "Key Vault Administrator" -> select the member by searching name or email.
            - [ ]  step2: back to same Resource group -> "Access Control(IAM).
            - [ ]  step3: select "view my access" you will find role created.
            - [ ]  step4: try creating Azure secret done.
        - [ ]  **Setup PowerBI**
download in Microsoft store and make works email by creating 365 account (or something) 
If you don’t have windows, y.ou could try vm but nightmare – just use powerbi in synapse.

- [ ]   **Data Ingestion with  ADF (phase 1)**
    - [ ]  Launch Data Factory
    - [ ]  Install self host integration runtime to our machine (since we are running the sql server)
        - [ ]  Go to manager -> integration runtimes
        - [ ]  One already exists to let cloud resources integrate
        - [ ]  New -> azure -> self hosted -> create
        - [ ]  Manual downloads an app with key used later to run, instead do express
(if it fails do manual…)
        - [ ]  Open integration runtime config mgr to confirm
    - [ ]  Step 1 – connect to on prem db and copy using data factory
        - [ ]  Create new pipeline in author
        - [ ]  New copy data activity
        - [ ]  Create new source dataset -> sql -> linked service (needed to connect to any data source) 
        - [ ]  Linkedservice: name, runtime, server name and db name (from ssms), 
sql auth (password from kv – linkedservice, test connection)
        - [ ]  **fails cos of authentication to read**
        - [ ]  go to kv -> iam  -> role assignment -> key vault secret user -> member (used manage services) 
            - [ ]  go back and select password, test connection, then create
won’t work cos you need to right click on your server in ssms and properties and security and change server authentication to SQL
        - [ ]  restart server in config mgr then go back test connection and create (oh make sure encrypt is optional!! Or get https cert error)
        - [ ]  then create new sink dataset, new linkedservice, your storage account may get error cos of soft delete – so go to storage account -> data protection -> uncheck enable soft delete for blobs
        - [ ]  **if this doesn’t work you can check by previewing and then run the following**

        - [ ]  USE AdventureWorksLT2019;
        - [ ]  GRANT SELECT ON SalesLT.Address TO mrk;
        - [ ]  **If still doesn’t work (JreNotFound) it may be that you need java installed (via choco or brew ideally)**

- [ ]  **Data Ingestion with ADF (phase 2)**
    - [ ]  Delete the file, as now creating pipeline for all tables
    - [ ]  Create new pipeline
    - [ ]  Create new SQL script in SSMS that lists all tables under SalesLT schema
**SELECT
s.name AS SchemaName,
t.name AS TableName
FROM sys.tables t
INNER JOIN sys.schemas s
ON t.schema_id = s.schema_id
WHERE s.name = 'SalesLT'**

So on pipeline create lookup activity, settings make a new source dataset and don’t select a specific table and use query option and copy the script (and uncheck first row only)
    - [ ]  Run debug and look at inputs outputs on output – see its in json
    - [ ]  Create forecah activity and connect on success
    - [ ]  On settings click items -> dynamic -> activity outputs for look for all tables -> add .values (which is the json list output)
    - [ ]  Update activities -> click pencil -> in foreach place copydata -> use SqlDBTables but select query and add dynamic content and insert:
**@{concat('SELECT * FROM ', item().SchemaName, '.', item().TableName )} // remember the space after from!!**
    - [ ]  Sink select the same parquet
We want it in format bronze/Schema/Tablename/Tablename.parquet so we make a new Parquet sink and select parameters where we can leverage the item() we used for the source. Now go back to the sink and update value to dynamic content and put in the relevant item() – make sure to use @
Now go back to parquet and under connection -> file path, update directory to @{concat( <<schema>>, ‘/’, <<table>>)}
And for file concat the tablename and .parquet
Validate and publish, go back to outer pipeline
We can either debug or trigger, so lets add trigger to trigger now
Click on link and can go monitor pipeline, each foreach is running concurrently as seen on gantt (if you need to make any changes, ensure you publish before triggering)
NEED TO UPDATE SSMS QUERY FOR mrk PRIVILEGES:
USE AdventureWorksLT2019;
GRANT SELECT ON SCHEMA::SalesLT TO mrk;
Since we made a change not in azure, we can click rerun pipeline in top left
Now you can see the files and directories in storage account
•	If you get an empty file:
“Azure blob storage does not support having empty folders. Thus, when you try to create folders (or empty folders), there will be a duplicate empty file. 
•	It is a blob storage with hierrachial namespace disabled-is that the cause? Yes, enabling hierarchical workspace will enable azure data lake which supports file and directory semantics and therefore which wouldn't create that additional file.”
•	E.g. https://stackoverflow.com/questions/76074718/additional-empty-blob-created-with-folder-names-in-azure-storage-container-not-a 

- [ ]  **Design the architecture**
    - [ ]  Read Databricks reference for the project → **LINK**
    - [ ]  Draw the data lakehouse architecture using draw.io or similar → **LINK**
- [ ]  **Create GitHub repository** → **LINK**
- [ ]  **Connect GitHub to Databricks using URL (**Workspace → Create → Git Folder)
- [ ]  **Create Lakehouse schemas (Unity Catalog) using**UI or SQL**:** `bronze` `silver` `gold`
- [ ]  **Create a volume inside bronze schema** `raw_sources`
- [ ]  Upload the 6 CSV files from engineering folder into the Bronze volume → **LINK**

<aside>

**Result:** Project is ready to start building Bronze, Silver, and Gold layers.
