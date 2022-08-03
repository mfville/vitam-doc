Description des workflows et des opérations
===========================================

Objectif du document
--------------------

Ce document a pour objectif de présenter les différents processus employés par la solution logicielle Vitam. Il est destiné aux administrateurs aussi bien techniques que fonctionnels, aux archivistes souhaitant avoir une connaissance plus avancée du logiciel ainsi qu’aux développeurs.

Il explicite chaque processus (appelé également « workflow »), et pour chacun leurs tâches, traitements et actions.

Ce document comprend du matériel additionnel pour faciliter la compréhension des processus comme des fiches récapitulatives et des schémas. Il explique également la manière dont est formée la structure des fichiers de workflow.


gestion du cycle de vie des archives
------------------------------------

Cette seconde partie du document détail les workflows suivants :

-   Entrée
-   Mise à jour unitaire
-   Mise à jour de masse
-   Mise à jour unitaire de masse (bulk update)
-   Export DIP
-   Élimination
-   Modification d’arborescence
-   Transfert
-   Restauration de métadonnées essentielles

### Ingest

Cette section décrit le processus (workflow) d’entrée, utilisé lors du transfert d’un Submission Information Package (SIP) dans la solution logicielle Vitam. Ce workflow se décompose en deux grandes catégories : le processus d’entrée externe dit « ingest externe » et le processus d’entrée interne dit « ingest interne ». Le premier prend en charge le SIP et effectue des contrôles techniques préalables, tandis que le second débute dès le premier traitement métier. Ex: Le processus d’entrée externe
comprend l’étape : STP_SANITY_CHECK_SIP (Contrôle sanitaire du SIP). Les autres étapes font partie du processus d’entrée interne.

Toutes les étapes, tâches et traitements sont journalisés dans le journal des opérations et décrivent le processus (clé et description de la clé associée dans le journal des opérations) tel qu’implémenté dans la version actuelle de la solution logicielle Vitam.


#### Processus des contrôles préalables à l’entrée (STP_SANITY_CHECK_SIP)

##### Contrôle sanitaire du SIP SANITY_CHECK_SIP (IngestExternalImpl.java)

-   **Règle** : tâche consistant à vérifier l’absence de virus dans le SIP

-   **Type** : bloquant

-   **Statuts** :

    -   OK : aucun virus n’a été détecté dans le SIP (SANITY\_CHECK\_SIP.OK = Succès du processus des contrôles préalables à l’entrée)

    -   KO : un ou plusieurs virus ont été détectés dans le SIP (SANITY\_CHECK\_SIP.KO = Échec du processus des contrôles préalables à l’entrée)

    -   FATAL : une erreur technique est survenue lors de la vérification de la présence de virus dans le SIP (SANITY\_CHECK\_SIP.FATAL = Erreur technique lors du processus des contrôles préalables à l’entrée)

##### Contrôle du format du conteneur du SIP CHECK_CONTAINER (IngestExternalImpl.java)

-   **Règle** : tâche consistant à vérifier le format du SIP via un outil d’identification de format qui se base sur le référentiel des formats qu’il intègre

    **Formats acceptés** : .zip, .tar, .tar.gz, .tar.bz2 et tar.gz2

-   **Type** : bloquant

-   **Statuts** :

    -   OK : le conteneur du SIP est au bon format (CHECK\_CONTAINER.OK = Succès du contrôle du format du conteneur du SIP)

    -   KO : le conteneur du SIP n’est pas au bon format (CHECK\_CONTAINER.KO = Échec du contrôle du format du conteneur du SIP)

    -   FATAL : une erreur technique est survenue lors de la vérification du format du conteneur du SIP, liée à l’outil d’identification des formats (CHECK\_CONTAINER.FATAL = Erreur technique lors du contrôle du format du conteneur du SIP)

##### Contrôle du nom du bordereau de transfert MANIFEST_FILE_NAME_CHECK (IngestExternalImpl.java)

-   **Règle** : tâche consistant à vérifier le nom du bordereau de transfert. Le nom du bordereau doit être conforme avec l’expression régulière suivante :

    \^((\[a-zA-Z0-9\]{1,56}\[\_-\]{1}){0,1}|\_{0,1})(manifest.xml)\\b »} A savoir : une chaîne de caractères débutant par des caractères alphanumériques sans accent, jusqu’à 56 caractères, suivi d’un « - » ou d’un « \_ » puis suivi de « manifest.xml ». Exemples valides : « MonNouveau-manifest.xml », « UnAutreBordereau\_manifest.xml ». « manifest.xml » tout court est également valide. Un SIP ne possédant pas du tout de bordereau de transfert verra également son entrée terminer en KO à cette
    tâche.

-   **Type** : bloquant

-   **Statuts** :

    -   OK : le nom du bordereau de transfert est conforme (MANIFEST\_FILE\_NAME\_CHECK.OK = Succès du contrôle du nom du bordereau de transfert : nom du fichier conforme)

    -   KO : le nom du bordereau de transfert n’est pas conforme (MANIFEST\_FILE\_NAME\_CHECK.KO = Échec du contrôle du nom du bordereau de transfert : nom du fichier non conforme)

    -   FATAL : une erreur technique est survenue lors de la vérification du nom du bordereau de transfert (MANIFEST\_FILE\_NAME\_CHECK.FATAL = Erreur technique lors du contrôle du nom du bordereau de transfert)

##### Vérification de l’intégrité du bordereau de transfert MANIFEST\_DIGEST\_CHECK (IngestExternalImpl.java)

-   **Règle** : tâche consistant à vérifier l’empreinte du bordereau de transfert calculée par la solution logicielle Vitam et celle déclarée dans le bordereau. Si l’empreinte déclarée dans le bordereau de transfert n’a pas été calculée avec l’algorithme SHA-512, alors l’empreinte est recalculée avec cet algorithme. Elle sera alors enregistrée dans la solution logicielle Vitam.

    **Algorithmes autorisés en entrée** : MD5, SHA-1, SHA-256, SHA-512

-   **Type** : bloquant

-   **Statuts** :

    -   KO : L’empreinte du bordereau de transfert n’est pas conforme (MANIFEST\_DIGEST\_CHECK.KO=Échec du contrôle de l''empreinte du bordereau de transfert : fichier non conforme)

    -   FATAL : Une erreur technique est survenue lors du contrôle de l’empreinte (MANIFEST\_DIGEST\_CHECK.FATAL=Erreur technique lors du contrôle de l''empreinte du bordereau de transfert)

#### Processus de réception du SIP dans Vitam STP\_UPLOAD\_SIP (IngestInternalResource.java)

-   **Règle** : étape consistant à vérifier la bonne réception du SIP sur l’espace de travail interne (« workspace »)

-   **Type** : bloquant

-   **Statuts** :

    -   OK : le SIP a été réceptionné sur l’espace de travail interne (STP\_UPLOAD\_SIP.OK = Succès du processus de réception du SIP)

    -   KO : le SIP n’a pas été réceptionné sur l’espace de travail interne (STP\_UPLOAD\_SIP.KO = Échec du processus de réception du SIP)

    -   FATAL : une erreur technique est survenue lors de la réception du SIP dans la solution logicielle Vitam, par exemple suite à une indisponibilité du serveur (STP\_UPLOAD\_SIP.FATAL = Erreur technique lors du processus de réception du SIP)

#### Processus de contrôle du SIP (STP\_INGEST\_CONTROL\_SIP)

##### Vérification globale du CHECK\_SEDA (CheckSedaActionHandler.java)

-   **Règle** : tâche consistant à vérifier la cohérence physique du SIP reçu par rapport au modèle de SIP accepté

    **Type de SIP accepté** : le bordereau de transfert, obligatoire dans le SIP, doit être conforme au schéma xsd par défaut fourni avec le standard SEDA v. 2.1, le SIP doit satisfaire les exigences du document « Structuration des SIP » et doit posséder un répertoire unique nommé « Content ».

-   **Type** : bloquant

-   **Statuts** :

    -   OK : le SIP est présent et conforme au schéma xsd par défaut fourni avec le standard SEDA v.2.1. il satisfait aux exigences de « structuration des SIP » et possède un repertoire unique « Content » (CHECK\_SEDA.OK = Succès de la vérification globale du SIP)

    -   KO :

        -   Cas 1 : le bordereau de transfert est introuvable dans le SIP ou n’est pas au format XML (CHECK\_SEDA.NO\_FILE.KO = Absence du bordereau de transfert ou bordereau de transfert au mauvais format)

        -   Cas 2 : le bordereau de transfert n’est pas au format XML (CHECK\_SEDA.NOT\_XML\_FILE.KO = Échec de la vérification globale du SIP : bordereau de transfert non conforme aux caractéristiques d’un fichier xml)

        -   Cas 3 : le bordereau de transfert ne respecte pas le schéma par défaut fourni avec le standard SEDA 2.1 (CHECK\_SEDA.NOT\_XSD\_VALID.KO = Échec de la vérification globale du SIP : bordereau de transfert non conforme au schéma SEDA 2.1)

        -   Cas 4 : le SIP contient plus d’un dossier « Content » (CHECK\_SEDA.CONTAINER\_FORMAT.DIRECTORY.KO = Échec de la vérification globale du SIP : le SIP contient plus d’un dossier ou un dossier dont le nommage est invalide)

        -   Cas 5 : le SIP contient plus d’un seul fichier à la racine (CHECK\_SEDA.CONTAINER\_FORMAT.FILE.KO = Échec de la vérification globale du SIP : le SIP contient plus d’un fichier à sa racine)

        -   Cas 6 : l’action est déjà exécutée CHECK\_SEDA.ALREADY\_EXECUTED = Action déjà exécutée : Pas de vérification globale du SIP

    -   FATAL :

        -   Cas 1 : une erreur technique est survenue lors de la vérification globale du SIP (CHECK\_SEDA.FATAL = Erreur technique lors de la vérification globale du SIP)

        -   Cas 2 : une erreur technique est survenue lors de la vérification globale du SIP (CHECK\_SEDA.NOT\_XML\_FILE.FATAL=Erreur technique lors de la vérification globale du SIP)

        -   Cas 3 : une erreur technique est survenue lors de la vérification globale du SIP (CHECK\_SEDA.NOT\_XSD\_VALID.FATAL=Erreur technique lors de la vérification globale du SIP)

##### Vérification de l’en-tête du bordereau de transfert CHECK\_HEADER (CheckHeaderActionHandler.java)

-   **Règles** : tâche permettant de vérifier les informations générales du bordereau de transfert (nommées « header » dans le fichier « manifest.xml ») et de l’existence du service producteur (OriginatingAgencyIdentifier)

-   **Type** : bloquant

-   **Statuts** :

    -   OK : les informations du bordereau de transfert sont conformes et le service producteur est déclaré (CHECK\_HEADER.OK = Succès de la vérification générale du bordereau de transfert)

    -   KO :

        -   Cas 1 : les informations du bordereau de transfert ne sont pas conformes ou il n’y a pas de service producteur déclaré (CHECK\_HEADER.KO = Échec de la vérification générale du bordereau de transfert)

        -   Cas 2 : les données référentielles sont inactives (CHECK\_HEADER.INACTIVE.KO = Échec de la vérification générale du bordereau de transfert : donnée référentielle inactive)

        -   Cas 3 : les données référentielles sont inconnues (CHECK\_HEADER.UNKNOWN.KO = Échec de la vérification générale du bordereau de transfert : donnée référentielle inconnue)

        -   Cas 4 : il y a une différence entre le profil déclaré dans le bordereau de transfert et celui déclaré dans le contrat (CHECK\_HEADER.DIFF.KO = Échec de la vérification générale du bordereau de transfert : différence entre le profil déclaré dans le bordereau de transfert et celui déclaré dans le contrat)

        -   Cas 5 : un des champs obligatoires n’est pas remplie (CHECK\_HEADER.EMPTY\_REQUIRED\_FIELD.KO = Vérification générale du bordereau de transfert : champ obligatoire vide)

        -   Cas 6 : la vérification a déjà été effectuée (CHECK\_HEADER.ALREADY\_EXECUTED = Action déjà exécutée : pas de vérification générale du bordereau de transfert)

    -   FATAL : une erreur technique est survenue lors des contrôles sur les informations générales du bordereau de transfert (CHECK\_HEADER.FATAL = Erreur technique lors de la vérification générale du bordereau de transfert)

**La tâche check\_header contient les traitements suivants :**

##### Vérification de la présence et contrôle des services agents CHECK\_HEADER.CHECK\_AGENT

Ce traitement n’est exécuté que si la valeur IN de *checkOriginatingAgency* est « true ».

-   **Règle** : traitement consistant à vérifier le service producteur ainsi que le service versant déclarés dans le SIP par rapport au référentiel des services agents présent dans la solution logicielle Vitam

-   **Type** : bloquant

-   **Statuts** :

    -   OK : le service producteur et, le cas échéant, le service versant déclarés dans le SIP est valide (service agent existant dans le référentiel des services agents)(CHECK\_HEADER.CHECK\_AGENT.OK = Succès de la vérification de la présence et du contrôle des services agents)

    -   KO :

        -   Cas 1 : aucun service producteur n’est déclaré dans la balise dédiée dans le bordereau de transfert (CHECK\_HEADER.CHECK\_AGENT.EMPTY\_REQUIRED\_FIELD.KO = Échec de la vérification de la présence et du contrôle des services agents : champ obligatoire vide)

        -   Cas 2 : le service producteur et, le cas échéant, le service versant déclarés dans le SIP n’est pas connu du référentiel des services agents (CHECK\_HEADER.CHECK\_AGENT.UNKNOWN.KO = Échec de la vérification de la présence et du contrôle des services agents : services agents inconnus du référentiel des services agents)

        -   Cas 3 : la balise permettant de déclarer un service producteur est absente du bordereau de tranfert (CHECK\_HEADER.CHECK\_AGENT.KO = Échec de la vérification de la présence et du contrôle des services agents)

    -   FATAL : une erreur technique est survenue lors de la vérification de la présence et du contrôle des services agents (CHECK\_HEADER.CHECK\_AGENT.FATAL = Erreur technique lors de la vérification de la présence et du contrôle des services agents)

##### Vérification de la présence et contrôle du contrat d’entrée CHECK\_HEADER.CHECK\_CONTRACT\_INGEST

Ce traitement n’est exécuté que si la valeur IN de *checkContract* est « true ».

-   **Règle** : traitement consistant à vérifier le contrat d’entrée déclaré dans le SIP par rapport au référentiel des contrats d’entrée présent dans la solution logicielle Vitam

-   **Type** : bloquant

-   **Statuts** :

    -   OK : le contrat déclaré dans le SIP existant dans le référentiel des contrats d’entrée de la solution logicielle Vitam et est actif (CHECK\_HEADER.CHECK\_CONTRACT\_INGEST.OK = Succès de la vérification de la présence et du contrôle du contrat d’entrée)

    -   KO :

        -   Cas 1 : le contrat d’entrée déclaré dans le SIP est inexistant (CHECK\_HEADER.CHECK\_CONTRACT\_INGEST.CONTRACT\_UNKNOWN.KO = Échec de la vérification de la présence du contrat d’entrée : contrat d’entrée inconnu du référentiel des contrats d’entrée)

        -   Cas 2 : le contrat d’entrée déclaré dans le SIP est inactif (CHECK\_HEADER.CHECK\_CONTRACT\_INGEST.CONTRACT\_INACTIVE.KO = Échec de la vérification du caractère actif du contrat d’entrée)

        -   Cas 3 : aucun contrat d’entrée n’a été trouvé dans le manifeste (CHECK\_HEADER.CHECK\_CONTRACT\_INGEST.CONTRACT\_NOT\_IN\_MANIFEST.KO = Échec de la vérification de la présence du contrat d’entrée : le champ ArchivalAgreement est absent du bordereau de transfert)

        -   Cas 4 : le contrat d’entrée déclaré dans le SIP n’existe pas dans le contexte applicatif (CHECK\_HEADER.CHECK\_CONTRACT\_INGEST.CONTRACT\_NOT\_IN\_CONTEXT.KO = Échec du contrôle de la présence du contrat d’entrée dans le contexte applicatif)

        -   Cas 5 : le contexte applicatif est inexistant (CHECK\_HEADER.CHECK\_CONTRACT\_INGEST.CONTEXT\_UNKNOWN.KO = Échec du contrôle de la présence du contexte applicatif : contexte inconnu du référentiel des contextes)

        -   Cas 6 : le contexte applicatif est inactif (CHECK\_HEADER.CHECK\_CONTRACT\_INGEST.CONTEXT\_INACTIVE.KO = Échec du contrôle du caractère actif du contexte applicatif)

        -   Cas 7 : erreur lors de la récupération du contexte applicatif (CHECK\_HEADER.CHECK\_CONTRACT\_INGEST.CONTEXT\_CHECK\_ERROR.KO = Échec de la vérification de la présence et du contrôle du contexte applicatif)

        -   Cas 8 : le contrat de gestion déclaré est inexistant CHECK\_HEADER.CHECK\_CONTRACT\_INGEST.MANAGEMENT\_CONTRACT\_UNKNOWN.KO = Échec de la vérification de la présence du contrat de gestion déclaré dans le contrat d’entrée: contrat de gestion connu dans le référentiel des contrats de gestion

        -   Cas 9 : le contrat de gestion déclaré est inactif CHECK\_HEADER.CHECK\_CONTRACT\_INGEST.MANAGEMENT\_CONTRACT\_INACTIVE.KO = Échec de la vérification de la présence du contrat de gestion déclaré dans le contrat d’entrée: contrat de gestion au statut inactif dans le référentiel des contrats de gestion

        -   Cas 10 : le contrat de gestion déclaré est invalide CHECK\_HEADER.CHECK\_CONTRACT\_INGEST.MANAGEMENT\_CONTRACT\_INVALID.KO = Échec de la vérification de la présence du contrat de gestion déclaré dans le contrat d’entrée: échec de validation des stratégies de stockage déclarées dans le contrat de gestion

    -   FATAL : une erreur technique est survenue lors de la vérification de la présence et du contrôle du contrat d’entrée ou du contexte applicatif (CHECK\_HEADER.CHECK\_CONTRACT\_INGEST.FATAL = Erreur technique lors de la vérification de la présence et du contrôle du contrat d’entrée ou du contexte applicatif)

##### Vérification de la relation entre le contrat d’entrée et le profil d’archivage CHECK\_HEADER.CHECK\_IC\_AP\_RELATION

Ce traitement n’est exécuté que si la valeur IN de *checkProfile* est « true ».

-   **Règle** : traitement consistant à vérifier que le profil d’archivage déclaré dans le contrat d’entrée du SIP est le même que celui déclaré dans le bordereau de transfert.

-   **Type** : bloquant

-   **Statuts** :

    -   OK : le profil d’archivage déclaré dans le contrat d’entrée et celui déclaré dans le bordereau de transfert sont les mêmes (CHECK\_HEADER.CHECK\_IC\_AP\_RELATION.OK = Succès de la vérification de la relation entre le contrat d’entrée et le profil)

    -   KO :

        -   Cas 1 : le profil d’archivage déclaré dans le SIP est inexistant (CHECK\_HEADER.CHECK\_IC\_AP\_RELATION.UNKNOWN.KO = Échec du contrôle de la présence du profil d’archivage dans le référentiel des profils d’archivage)

        -   Cas 2 : le profil d’archivage déclaré dans le SIP est inactif (CHECK\_HEADER.CHECK\_IC\_AP\_RELATION.INACTIVE.KO = Échec du contrôle du caractère actif du profil d’archivage)

        -   Cas 3 : le profil d’archivage déclaré dans le contrat d’entrée et celui déclaré dans le bordereau de transfert ne sont pas les mêmes (CHECK\_HEADER.CHECK\_IC\_AP\_RELATION.DIFF.KO = Échec du contrôle de cohérence entre le profil d’archivage déclaré dans le bordereau de transfert et celui déclaré dans le contrat d’entrée)

    -   FATAL : une erreur technique est survenue lors de la vérification de la relation entre le contrat d’entrée et le profil d’archivage (CHECK\_HEADER.CHECK\_IC\_AP\_RELATION.FATAL = Erreur technique lors de la vérification de la relation entre le contrat d’entrée et le profil d’archivage)

##### Vérification de la conformité du bordereau de transfert par le profil d’archivage CHECK\_HEADER.CHECK\_ARCHIVEPROFILE

-   **Règle** : traitement consistant à vérifier que le bordereau de transfert du SIP est conforme aux exigences du profil d’archivage. Si aucun profil d’archivage ne s’applique au SIP, ce traitement est ignoré

-   **Type** : bloquant

-   **Statuts** :

    -   OK : le bordereau de transfert est conforme aux exigences du profil d’archivage (CHECK\_HEADER.CHECK\_ARCHIVEPROFILE.OK = Succès de la vérification de la conformité au profil d’archivage)

    -   KO : le bordereau de transfert n’est pas conforme aux exigences du profil d’archivage (CHECK\_HEADER.CHECK\_ARCHIVEPROFILE.KO = Échec de la vérification de la conformité au profil d’archivage)

    -   FATAL : une erreur technique est survenue lors de la vérification du bordereau de transfert par rapport au profil d’archivage (CHECK\_HEADER.CHECK\_ARCHIVEPROFILE.FATAL = Erreur technique lors de la vérification de la conformité au profil d’archivage)

##### Préparation des informations de stockage PREPARE\_STORAGE\_INFO (PrepareStorageInfoActionHandler.java)

-   **Règle** : tâche consistant à récupérer les informations liées aux offres de stockage à partir des stratégies

-   **Type** : bloquant

-   **Statuts** :

    -   OK : la récupération des informations de stockage a bien été effectuée (PREPARE\_STORAGE\_INFO.OK = Succès de la préparation des informations de stockage)

    -   KO : la récupération des informations de stockage n’a pas pu être effectuée (PREPARE\_STORAGE\_INFO.KO = Échec de la préparation des informations de stockage)

    -   FATAL : une erreur technique est survenue lors de la récupération des informations de stockage (PREPARE\_STORAGE\_INFO.FATAL = Erreur technique lors de la récupération des informations de stockage)

##### Vérification des objets et groupes d’objets CHECK\_DATAOBJECTPACKAGE

-   **Règle** : tâche consistant à vérifier les objets et groupes d’objets

-   **Type** : bloquant

-   **Statuts** :

    -   OK : la vérification des objets et groupes d’objets à été effectué avec succès (CHECK\_DATAOBJECTPACKAGE.OK=Succès de la vérification des objets et groupes d’objets)

    -   KO : la vérification des objets et groupes d’objets n’a pu être effectué (CHECK\_DATAOBJECTPACKAGE.KO=Échec de la vérification des objets et groupes d’objets)

    -   FATAL : une erreur technique est servenue lors de la vérification des objets et groupes d’objets (CHECK\_DATAOBJECTPACKAGE.FATAL=Erreur technique lors de la vérification des objets et groupes d’objets)

        La tâche Check\_DataObjectPackage contient les traitements suivants :

##### Vérification des usages des groupes d’objets CHECK\_DATAOBJECTPACKAGE.CHECK\_MANIFEST\_DATAOBJECT\_VERSION (CheckVersionActionHandler.java)

-   **Règle** : traitement consistant à vérifier que tous les objets décrits dans le bordereau de transfert du SIP déclarent un usage conforme à la liste des usages acceptés dans la solution logicielle Vitam ainsi qu’un numéro de version respectant la norme de ce champ

-   **Types d’usages acceptés : **

    -   original papier (PhysicalMaster),

    -   original numérique (BinaryMaster),

    -   diffusion (Dissemination),

    -   vignette (Thumbnail),

    -   contenu brut (TextContent).

        Les numéros de versions sont optionnels, il s'agit d'un entier positif ou nul (0, 1, 2…).

        La grammaire est : « usage\_version ». Exemples : « BinaryMaster\_2 », « TextContent\_10 » ou sans numéro de versions « PhysicalMaster ».

-   **Type** : bloquant

-   **Statuts** :

    -   OK : les objets contenus dans le SIP déclarent tous dans le bordereau de transfert un usage cohérent avec ceux acceptés et optionnellement un numéro de version respectant la norme de ce champ usage, par exemple « BinaryMaster\_2 » (CHECK\_DATAOBJECTPACKAGE.CHECK\_MANIFEST.DATAOBJECT.VERSION.OK = Succès de la vérification des usages des objets)

    -   KO :

        -   Cas 1 : un ou plusieurs BinaryMaster sont déclarés dans un ou plusieurs objets physiques (CHECK\_DATAOBJECTPACKAGE.CHECK\_MANIFEST\_DATAOBJECT\_VERSION.PDO\_DATAOBJECTIONVERSION\_BINARYMASTER.KO = L’objet physique déclare un usage « BinaryMaster ». Cet usage n’est pas autorisé pour les objets physiques

        -   Cas 2 : un ou plusieurs PhysicalMaster sont déclarés dans un ou plusieurs objets binaires (CHECK\_DATAOBJECTPACKAGE.BDO\_DATAOBJECTIONVERSION\_PHYSICALMASTER.KO = Au moins un objet binaire déclare un usage « PhysicalMaster ». Cet usage n’est pas autorisé pour les objets binaires)

        -   Cas 3 : un ou plusieurs objets contenus dans le SIP déclarent dans le bordereau de transfert un usage ou un numéro de version incohérent avec ceux acceptés (CHECK\_DATAOBJECTPACKAGE.CHECK\_MANIFEST\_DATAOBJECT\_VERSION.INVALID\_DATAOBJECTVERSION.KO = Cet objet déclare un usage incorrect. L’usage doit s’écrire sous la forme \[usage\] ou \[usage\]\_\[version\]. « Usage » doit être parmi l’énumération DataObjectVersion définie pour Vitam, « version » doit être un entier positif)

        -   Cas 4 : une ou plusieurs URI sont vides (CHECK\_DATAOBJECTPACKAGE.CHECK\_MANIFEST\_DATAOBJECT\_VERSION.EMPTY\_REQUIRED\_FIELD.KO = Il existe au moins un champ non renseigné dont la valeur est obligatoire)

    -   FATAL : une erreur technique est survenue lors du contrôle des usages déclarés dans le bordereau de transfert pour les objets contenus dans le SIP (CHECK\_MANIFEST\_DATAOBJECT\_VERSION.FATAL = Erreur technique lors de la vérification des usages des objets)

##### Vérification du nombre d’objets CHECK\_DATAOBJECTPACKAGE.CHECK\_MANIFEST\_OBJECTNUMBER (CheckObjectsNumberActionHandler.java)

-   **Règle** : traitement consistant à vérifier que le nombre d’objets binaires reçus dans la solution logicielle Vitam et stocké dans l’espace de travail interne (« workspace ») est strictement égal au nombre d’objets binaires déclaré dans le manifeste du SIP

-   **Type** : bloquant

-   **Statuts** :

    -   OK : le nombre d’objets reçus dans la solution logicielle Vitam est strictement égal au nombre d’objets déclarés dans le bordereau de transfert du SIP (CHECK\_DATAOBJECTPACKAGE.CHECK\_MANIFEST\_OBJECTNUMBER.OK = Succès de la vérification du nombre d’objets)

    -   KO :

        -   Cas 1 : le nombre d’objets reçus dans la solution logicielle Vitam est supérieur au nombre d’objets déclaré dans le bordereau de transfert du SIP (CHECK\_DATAOBJECTPACKAGE.CHECK\_MANIFEST\_OBJECTNUMBER.MANIFEST\_INFERIOR\_BDO.KO = Le bordereau de transfert déclare moins d’objets binaires qu’il n’en existe dans le répertoire Content du SIP)

        -   Cas 2 : le nombre d’objets reçus dans la solution logicielle Vitam est inférieur au nombre d’objets déclaré dans le bordereau de transfert du SIP (CHECK\_DATAOBJECTPACKAGE.CHECK\_MANIFEST\_OBJECTNUMBER.MANIFEST\_SUPERIOR\_BDO.KO = Le bordereau de transfert déclare plus d’objets binaires qu’il n’en existe dans le répertoire Content du SIP)

        -   Cas 3 : une ou plusieurs balises URI déclarent un chemin invalide (CHECK\_DATAOBJECTPACKAGE.CHECK\_MANIFEST\_OBJECTNUMBER.INVALID\_URI.KO = Au moins un objet déclare une URI à laquelle ne correspond pas de fichier ou déclare une URI déjà utilisée par un autre objet)

    -   FATAL : une erreur technique est survenue lors de la vérification du nombre d’objets (CHECK\_DATAOBJECTPACKAGE.CHECK\_MANIFEST\_OBJECTNUMBER.FATAL = Erreur technique lors de la vérification du nombre d’objets)

##### Vérification de la cohérence du bordereau de transfert CHECK\_DATAOBJECTPACKAGE.CHECK\_MANIFEST (ExtractSedaActionHandler.java)

-   **Règle** : traitement consistant à :

    -   créer les journaux du cycle de vie des unités archivistiques et des groupes d’objets,

    -   extraire les unités archivistiques, objets binaires et objets physiques du bordereau de transfert,

    -   vérifier la présence de récursivité dans les arborescences des unités archivistiques et à créer l’arbre d’ordre d’indexation,

    -   extraire les métadonnées contenues dans la balise ManagementMetadata du bordereau de transfert pour le calcul des règles de gestion,

    -   vérifier la validité des rattachements des unités du SIP aux unités présentes dans la solution logicielle Vitam si demandés,

    -   détecter des problèmes d’encodage dans le bordereau de transfert et vérifier que les objets ne font pas référence directement à des unités si ces objets possèdent des groupes d’objets,

    -   vérifier la présence obligatoire d’un objet de type Master pour une entrée, et vérifier les usages d’objets autorisés pour les rattachements.

-   **Type** : bloquant

-   **Statuts** :

    -   OK : les journaux du cycle de vie des unités archivistiques et des groupes d’objets ont été créés avec succès, aucune récursivité n’a été détectée dans l’arborescence des unités archivistiques, la structure de rattachement déclarée existe, le type de structure de rattachement est autorisé, (par exemple, un SIP peut être rattaché à un plan de classement, mais pas l’inverse) aucun problème d’encodage n’a été détecté et les objets avec groupe d’objets ne référencent pas directement les
        > unités. L’extraction des unités archivistiques, objets binaires et physiques, la création de l’arbre d’indexation et l’extraction des métadonnées des règles de gestion ont été effectuées avec succès, les vérifications au niveau des types d’usages autorisés ont bien été effectués. (CHECK\_DATAOBJECTPACKAGE.CHECK\_MANIFEST.OK = Succès du contrôle de cohérence du bordereau de transfert).

    -   KO :

        -   Cas 1 : au moins une demande de rattachement à des unités archivistiques existantes dans le système a échoué, car le nœud de rattachement déclaré dans le contrat d’entrées a pour valeur « null » (CHECK\_DATAOBJECTPACKAGE.CHECK\_MANIFEST.NULL\_LINK\_PARENT\_ID\_ATTACHMENT.KO = Le rattachement n’a pas été effectué : le contrat d’entrée ne déclare pas de nœud de rattachement)

        -   Cas 2 : au moins une demande de rattachement à des unités archivistiques existantes dans le système a échoué, car le contrat d’entrée requiert un rattachement à au moins une unité archivistique (CHECK\_DATAOBJECTPACKAGE.CHECK\_MANIFEST.ATTACHMENT\_REQUIRED.KO = Le contrat d’entrée requiert un rattachement à au moins une unité archivistique)

        -   Cas 3 : au moins une demande de rattachement à des unités archivistiques existantes dans le système a échoué, car le contrat d’entrée n’autorise pas les rattachements (CHECK\_DATAOBJECTPACKAGE.CHECK\_MANIFEST.UNAUTHORIZED\_ATTACHMENT\_BY\_CONTRACT.KO = Le rattachement n’a pas été effectué : le contrat d’entrée n’autorise pas les rattachements)

        -   Cas 4 : au moins une demande de rattachement à des unités archivistiques existantes dans le système a échoué en raison d’un nombre d’unités archivistiques répondant à la requête effectuée supérieur à 1 (CHECK\_DATAOBJECTPACKAGE.CHECK\_MANIFEST.TOO\_MANY\_FOUND\_ATTACHMENT.KO = Le rattachement n’a pas été effectué : l’élément de rattachement n’est pas unique dans le système)

        -   Cas 5 : au moins un objet binaire dans le bordereau de transfert déclare plusieurs version d’un même usage (CHECK\_DATAOBJECTPACKAGE.CHECK\_MANIFEST.TOO\_MANY\_VERSION\_BY\_USAGE.KO = Le transfert de plusieurs versions d’un même usage dans un même versement est interdit)

        -   Cas 6 : au moins une demande de rattachement à des unités archivistiques existantes dans le système a échoué en raison d’un nombre d’unités archivistiques répondant à la requête, égal à 0 (CHECK\_DATAOBJECTPACKAGE.CHECK\_MANIFEST.NOT\_FOUND\_ATTACHMENT.KO = Le rattachement n’a pas été effectué : l’élément de rattachement n’existe pas dans le système)

        -   Cas 7 : au moins une demande de rattachement à des unités archivistiques existantes dans le système a échoué, car le rattachement demandé n’est pas autorisé (CHECK\_DATAOBJECTPACKAGE.CHECK\_MANIFEST.UNAUTHORIZED\_ATTACHMENT.KO = Le rattachement n’a pas été effectué : le rattachement n’est pas situé dans le périmètre autorisé)

        -   Cas 8 : au moins une demande de rattachement à des unités archivistiques existantes dans le système a échoué, car le GUID déclaré n’est pas valide (CHECK\_DATAOBJECTPACKAGE.CHECK\_MANIFEST.INVALID\_GUID\_ATTACHMENT.KO = Le rattachement n’a pas été effectué : l’élément de rattachement est incorrect)

        -   Cas 9 : au moins une demande de rattachement à des unités archivistiques existantes dans le système a échoué, car elle provoquerait une récursivité de l’arborescence (CHECK\_DATAOBJECTPACKAGE.CHECK\_MANIFEST.MODIFY\_PARENT\_EXISTING\_UNIT\_UNAUTHORIZED.KO = Le rattachement n’a pas été effectué : impossibilité de rattacher une unité archivistique existante à une unité archivistique parente)

        -   Cas 10 : une ou plusieurs balises de rattachement vers un groupe d'objets techniques existant déclarent autre chose que le GUID d’un groupe d'objets techniques existant (CHECK\_DATAOBJECTPACKAGE.CHECK\_MANIFEST.EXISTING\_OG\_NOT\_DECLARED.KO = Une unité archivistique déclare un objet à la place du groupe d’objets correspondant)

        -   Cas 11 : une récursivité a été détectée dans l’arborescence des unités archivistiques (CHECK\_DATAOBJECTPACKAGE.CHECK\_MANIFEST.CHECK\_MANIFEST\_LOOP.KO = Le bordereau de transfert présente une récursivité dans l’arborescence de ses unités archivistiques)

        -   Cas 12 : il y a un problème d’encodage ou des objets référencent directement des unités archivistiques (CHECK\_DATAOBJECTPACKAGE.CHECK\_MANIFEST.KO = Échec du contrôle de cohérence du bordereau de transfert)

        -   Cas 13 : présence attendue d’un objet de type Master: Binary ou physical CHECK\_DATAOBJECTPACKAGE.CHECK\_MANIFEST.MASTER\_MANDATORY\_REQUIRED.KO = Absence d’un BinaryMaster ou PhysicalMaster dans le groupe d’objet

        -   Cas 14 : le contrat d’entrée n’autorise pas un ou plusieurs usages d’objets (CHECK\_DATAOBJECTPACKAGE.CHECK\_MANIFEST.ATTACHMENT\_OBJECTGROUP.KO = Le contrat d’entrée n’autorise pas le rattachement d’un objet à un groupe d’objets existant)

        -   Cas 15 : il y a une donnée malformatée (CHECK\_DATAOBJECTPACKAGE.CHECK\_MANIFEST\_MALFORMED\_DATA.KO = Le bordereau de transfert possède une donnée malformée)

    -   FATAL : une erreur technique est survenue lors de la vérification de la cohérence du bordereau, par exemple les journaux du cycle de vie n’ont pu être créés (CHECK\_MANIFEST.FATAL = Erreur technique lors du contrôle de cohérence du bordereau de transfert)

##### Vérification de la cohérence entre objets, groupes d’objets et unités archivistiques CHECK\_DATAOBJECTPACKAGE.CHECK\_CONSISTENCY (CheckObjectUnitConsistencyActionHandler.java)

-   **Règle** : traitement consistant à vérifier que chaque objet ou groupe d’objets est référencé par une unité archivistique, à rattacher à un groupe d’objets les objets sans groupe d’objets mais référencés par une unité archivistique, à créer la table de concordance (MAP) entre les identifiants des objets et des unités archivistiques du SIP et à générer leurs identifiants pérennes dans la solution logicielle Vitam (GUID)

-   **Type** : bloquant

-   **Statuts** :

    -   OK : aucun objet ou groupe d’objets n’est orphelin (c’est-à-dire non référencé par une unité archivistique) et tous les objets sont rattachés à un groupe d’objets. La table de concordance est créée et les identifiants des objets et unités archivistiques ont été générés (CHECK\_DATAOBJECTPACKAGE.CHECK\_CONSISTENCY.OK = Succès de la vérification de la cohérence entre objets, groupes d’objets et unités archivistiques)

    -   KO : au moins un objet ou groupe d’objets est orphelin (c’est-à-dire non référencé par une unité archivistique) (CHECK\_DATAOBJECTPACKAGE.CHECK\_CONSISTENCY.KO = Échec de la vérification de la cohérence entre objets, groupes d’objets et unités archivistiques)

    -   FATAL : une erreur technique est survenue lors de la vérification de la cohérence entre objets, groupes d’objets et unités archivistiques (CHECK\_DATAOBJECTPACKAGE.CHECK\_CONSISTENCY.FATAL = Erreur technique lors de la vérification de la cohérence entre objets, groupes d’objets et unités archivistiques)

##### Vérification du rattachement à un groupe d’objets ou une unité archivistique entrés sans erreur CHECK\_ATTACHEMENT (CheckAttachementActionHandler.java)

-   **Règle** : tâche consistant à vérifier le rattachement à des groupes d’objets techniques et des unités archivistiques entrés sans erreur dans le système.

-   **Type** : bloquant

-   **Statuts** :

    -   OK : la vérification des objets et groupes d’objets à été effectué avec succès (CHECK\_ATTACHEMENT.OK=Succès de la vérification du rattachement entre objets, groupes d'objets et unités archivistiques existantes et les nouveaux)

    -   KO : la vérification des objets et groupes d’objets n’a pu être effectuée, car le groupe d’objets techniques ou l’unité archivistique devant faire l’objet d’un rattachement dans le système sont entrés en erreur (CHECK\_ATTACHEMENT.KO=Échec de la vérification du rattachement entre objets, groupes d'objets et unités archivistiques existantes et les nouveaux)

    -   FATAL : une erreur technique est survenue lors de la vérification des objets et groupes d’objets (CHECK\_ATTACHEMENT.FATAL=Erreur technique lors de la vérification du rattachement entre objets, groupes d'objets et unités archivistiques existantes et les nouveaux)

#### Processus de contrôle et traitement des objets (STP\_OG\_CHECK\_AND\_TRANSFORME)
