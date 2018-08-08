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

* **Les schémas** :

4 types de préfixes de dénomination de schémas sont présents dans la base de données :

|Préfixe|Nom du schéma|Exemple|Définition|
|:-:|:-:|:-:|:-:|
|m_|nom de la thématique|m_habitat|contient des données métiers gérés par l'Agglomération ou utilisées pour les besoins d'un service|
