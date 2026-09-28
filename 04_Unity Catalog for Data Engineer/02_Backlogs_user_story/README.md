
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
         - [ ]  Créer des containers bronze, silver, gold dans ADLS
         - [ ]  On retourne maintenant dans **Manage account** d'azure databricks pour **créer le metastore** de unity catalog
              - [ ]   Il faut toujours préciser le ADLS GEN2 Path lors de la création du métastore sinon on sera obligé de le renseigner à chaque fois qu'on créera un catalog  (**metastore@adls_account.dfs.core.windows.net/**)
              - [ ]   Il faut aussi renseigner le access connector id  disponible dans Access connector lors de la création du metastore
              - [ ]   Assigner le ou les workspaces à ce metastore
              - [ ]   Le compte email avec EXT est toujours ADMIN du metastore il faut le changer pour le rôle d'admin au compte email normal(créer un groupe admin et attribué le role d'dmin)
              - [ ]   GRANT ON metastore(Donner toutes les autorisation au compte admin) si ce n'est pas fait il y'aura erreur 
       
        - [ ]   Création de storage credential( de type azure managed identity) nommé **access_conn** et utiliser le **access connector ID**
        - [ ]   Création les External locations(Créer une external location pour chaque container de adls):Il faut s'assurer que les données brutes sont déja disponible dans le bronze container!
             - [ ]   **Bronze_external**
             - [ ]   **Silver_external**
             - [ ]   **Gold_external**
             - [ ]   Tester chacune des connexions des external locations
                   
                - [ ]   <img width="1100" height="466" alt="image" src="https://github.com/user-attachments/assets/0c043f22-bc4c-4f9e-892f-e5b02d548926" />

        - [ ]   Créer workspace (new project folder ou databricks unity catalog project) pour contenir les notebooks
        - [ ]   Créer un notebook nommé config level(pour définir catalogs,schemas)
        - [ ]   Attach notebook to cluster
        - [ ]   Création du catalog( cars_catalog ou bien créer 3 catalogues(bronze, silver, gold ou architecture Dev, Test & Prod)
        - [ ]   Création des schemas
              
             -  [ ]    <img width="1398" height="804" alt="image" src="https://github.com/user-attachments/assets/17314a6b-6160-45fa-bd19-b244d0e26405" />

  **Lors de la création ds objets unity catalog(catalog, tables,...)si on ne précise pas sur le code external location on aura des managed catalog & tables c'est-à-dire que la table ou catalog créé sera enregistré directement par defaut dans le container metastore de adls qu'on avait renseigné lors de la création du metastore:Databricks gère à la la fois les metadata et les fichiers**

**Managed_tables:** **Tables pour lesquelles lors de leur création on a pas utiliser une external location pointant sur un container specifique(bronze, silver, gold); Databricks s'occupe de gestion des tables managées(optimisation et performance)
les fichiers et metadata sont managés par Databricks: si on drop la managed table elle restera pendant 7jours dans ADLS et avec un undrop on peut la recréer**;
**L'external location auto créé lors de la création du metastore (metastore_root_location) doit etre configuré sur GRANT PRIVILEGES le email normal; il represente la managed_location**

**External_tables:** **Tables pour lesquelles lors de leur création on a utiliser une external location pointant sur un container specifique(bronze, silver, gold);
Les metadata sont dans databricks(catalog, schema, nom de table et infos) et les fichiers stockées en externe** 
  
Après la création du notebook ayant servi à créer des requetes pour créer catalog & schemas on crée une deuxième notebook pour lecture et transformation des données
   - [ ] Création du notebook silver(ou Bronze_to_silver)
   - [ ] **Reading Data in df**
         
        - [ ]    <img width="2000" height="1001" alt="image" src="https://github.com/user-attachments/assets/581566c7-7e53-4671-9f54-7e9024c76654" />
   - [ ] **Appliquer des transformations de données**
         
       - [ ]   <img width="1378" height="792" alt="image" src="https://github.com/user-attachments/assets/780ff6b8-2b45-419b-96df-d1668fa37b5d" />
   - [ ] Then we created an additional column to calculate the revenue per unit this can be useful for the analytics.
         
        - [ ]    <img width="1100" height="601" alt="image" src="https://github.com/user-attachments/assets/4dbcc1ea-ad44-4349-8b4d-583be15f9971" />
   - [ ] **Aggregations**
         
        - [ ]   <img width="1100" height="650" alt="image" src="https://github.com/user-attachments/assets/ab3d3045-d8a6-4b78-8100- d132a28aed19" />
   - [ ] On peut créer un visuel sur Databricks avec le bouton (+) à coté de la table
         
       - [ ]    <img width="939" height="743" alt="image" src="https://github.com/user-attachments/assets/e8670554-b21c-4b28-a49d-6d20ca99782e" />
   - [ ] **Writing the transformed data to the silver storage container**

       - [ ]  <img width="1100" height="305" alt="image" src="https://github.com/user-attachments/assets/c58a673d-190c-47a8-868d-c2be01993b28" />
- [ ]  On cheque dans ADSL pour voir si les fichiers transformées sont le silver container
     - [ ]  <img width="1100" height="423" alt="image" src="https://github.com/user-attachments/assets/c7adea56-bc6d-44ab-a016-e40a061f8a18" />
     
- [ ] On peut réqueter les fichiers
SELECT * FROM 'abfss://<container>@<storageaccount>.dfs.core.windows.net/<path>/<file>'
      



    
             
- [ ]  **Setup PowerBI**
download in Microsoft store and make works email by creating 365 account (or something) 
If you don’t have windows, y.ou could try vm but nightmare – just use powerbi in synapse.


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
