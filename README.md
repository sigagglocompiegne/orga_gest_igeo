![picto](/doc/img/Logo_web-GeoCompiegnois.png)

# Principe organisationnel de la base de données igeo_compiegnois de l'Agglomération de la Région de Compiègne

## Les rôles de connexion et privilèges des groupes
  
  * **Généralité** :
  
L'ensemble des paramètres de connexion sont disponibles sur le wiki dédié "Serveur de base de données" sur le serveur RedMine.
  
La propriété des schémas, tables, ... a été transféré au groupe create_sig pour qu'il puisse modifier la structure. Les autres groupes ne pourront pas le faire. Le rôle postgres reste le superutilisateur et peut modifier la structure même si il n'est pas le propriétaire. 
**ATTENTION :** petite particularité importante, le rafraichissement des données d’une vue matérialisée peut se produire uniquement avec ce rôle de connexion.

Le groupe edit_sig ne peut que modifier les données. Il ne peut pas modifier la structure, faire de trigger, ni de clés, ni tronquer les tables.
Le groupe read_sig peut uniquement lire les données.

Des particularités ont été intégrés concernant les privilèges sur certains schémas. En effet, il n’est pas nécessaire que le rôle edit_sig puisse modifier les données sur les schémas contenant des référentiels extérieurs. Sur ceux-ci le rôle edit_sig a seulement un rôle de lecture.
  
  * **Tableaux de répartition** :

|Rôle de connexion|Superutilisateur|Propriétaire des objets|Appartient au groupe|Privilèges sur le structure|Privilèges sur les données|
|:-:|:-:|:-:|:-:|:-:|:-:|
|postgres|x|(par défaut)|-|all|all|
|sig_create||x|create_sig|all|all|
|sig_edit|||edit_sig|aucun|select,insert,update,delete|
|sig_read|||read_sig|aucun|select|

## Règles de dénomination des objets de la base de données

L'ensemble des libellés (schéma, table, champ, vue, ...) doit être écrit en minuscule ce qui permet d'éviter l’utilisation des "" dans les requêtes sql).

### Les schémas

  * **Généralité** :
  
Un schéma doit contenir uniquement de la donnée brute qui peut être modifiée soit manuellement ou avec l'aide de déclencheur (ou trigger) mis en place pour automatiser certaines tâches.
Seules les vues pour la gestion ou de filtrage simplifié de la donnée peuvent être contenues dans les schémas de gestion. Les autres vues ayant des usages décisionnels, d'analyses ou d'OpenData sont stockées dans le schémas d'exploitation.

   * **Tableaux de nomage** :

4 types de préfixes de dénomination de schémas sont présents dans la base de données :

|Préfixe|Nom du schéma|Type|Exemple|Définition|
|:-:|:-:|:-:|:-:|:-:|
|m_|nom de la thématique|gestion|m_urbanisme_doc, m_habitat, ...|contient des données métiers gérés par l'Agglomération ou utilisées pour les besoins d'un service|
|r_|nom du référentiel|gestion|r_bdtopo, r_pcrs, r_objet,...|contient des données issues de référentiel ou étant concédéré comme des référenties gérées par l'Agglomération ou provenant de producteurs tiers |
|s_|nom de la base de données|gestion|s_sirene, s_rpls, ...|contient des données attributaires de référence provenant de producteurs tiers|
|x_|nom de l'usage|exploitation||contient des données pré-traitées pour les applications WebSIG métiers, Grands Publics, pour des exports OpenData ou des traitements particuliers liés à des projets|
||||x_apps|schéma contenant des tables ou vues pré-traitées et utilisées dans les applicatifs WebSIG métiers|
||||x_apps_public|schéma contenant des tables ou vues pré-traitées et utilisées dans les applicatifs Grands Publics|
||||x_opendata|schéma contenant des tables ou vues pré-traitées et utilisées pour les exports OpenData|
||||x_projet|schéma contenant des tables ou vues pré-traitées pour répondre à une demande dans le cadre d'un projet|

### Les tables

  * **Généralité** :
  
La dénomination des tables doit être cohérente entre tous les schémas afin d'assurer une meilleure visibilité des données.
Néanmoins, on peut considérer 2 cas :
- les données de référence : elles sont issues de producteurs extérieurs (comme l'IGN, l'Insee, ...) et dans ces cas particuliers, le nom des tables est conservé afin d'assurer un meilleur suivi,
- les données "dites" métiers sont gérées (pour la plupart) en interne (mais peuvent être d'origine extérieur) et ne sont donc pas soumises à des contraintes de modèle externe. Dans le cas de l'existence d'un format d'échange standard de données, le nom des tables est alors généré à l'export des données.

  * **Tableaux de nomage** :

