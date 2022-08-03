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
comprend l’étape : STP\_SANITY\_CHECK\_SIP (Contrôle sanitaire du SIP). Les autres étapes font partie du processus d’entrée interne.

Toutes les étapes, tâches et traitements sont journalisés dans le journal des opérations et décrivent le processus (clé et description de la clé associée dans le journal des opérations) tel qu’implémenté dans la version actuelle de la solution logicielle Vitam.

#### Processus des contrôles préalables à l’entrée (STP\_SANITY\_CHECK\_SIP)

##### Contrôle sanitaire du SIP SANITY\_CHECK\_SIP (IngestExternalImpl.java)

-   **Règle** : tâche consistant à vérifier l’absence de virus dans le SIP

-   **Type** : bloquant

-   **Statuts** :
