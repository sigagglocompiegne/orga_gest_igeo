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

### Les objets d'un schéma

  * **Généralité** :
  
La dénomination des tables, vues, trigger, function, séquence, index, clé primaire et étrangère ... doit être cohérente entre tous les schémas afin d'assurer une meilleure visibilité des données.
Néanmoins, on peut considérer 2 cas :

. les données de référence : elles sont issues de producteurs extérieurs (comme l'IGN, l'Insee, ...) et dans ces cas particuliers, le nom des tables est conservé afin d'assurer un meilleur suivi,

. les données "dites" métiers sont gérées (pour la plupart) en interne (mais peuvent être d'origine extérieur) et ne sont donc pas soumises à des contraintes de modèle externe. Dans le cas de l'existence d'un format d'échange standard de données, le nom des tables est alors généré à l'export des données.

  * **Les tables** :

Le tableau ci-dessous indique les principes de dénomination des tables. 

|Contenus|Pré-préfixe|Préfixe|exemple|
|:-:|:-:|:-:|:-:|
|données attributaires et géométriques||**geo_**|`geo_p_zone_urba`|
|uniquement de la donnée attributaire ||**an_**|`an_doc_urba`|
|uniquement de la donnée attributaire servant de liens ou de correspondance||**lk_**|`lk_voirie_rurbain`|
|liste de valeur||**lt_**|`lt_typedoc`|
|log ou information de suivi||**log_**|`log_suivi_audit`|
|traitement applicatif grand public|**xappspublic_**|préfixe correspondant|`xappspublic_an_dec_pav_adr_proxi`|
|traitement applicatif pro|**xapps_**|préfixe correspondant|`xapps_an_fisc_geo_taxe_amgt`|
|export OpenData|**xopendata_**|préfixe correspondant||

**Rappel :**

Ces préfixes sont suivis de la dénomination classique des tables.

Cette dénomination peut-être liée à un modèle de données issus d'une norme (cas pour les données des pos-plu) ou laissé à la liberté de l'administrateur en respectant une syntaxe de bases :

**ex :** `geo_[theme]_[identification]`. Si on considère la création d'une table localisant les locaux d'activité, on pourrait la dénommer ainsi geo_eco_locaux.

Les tables doivent être commentées afin d'assurer la compréhension de la donnée (au minimum définir en quelques mots le contenu de la table, la source, l'échelle d'emprise, éventuellement une date de validité, de mise à jour, ...).

 * **Les vues** :
 
QGIS imposant une contrainte aux vues pour être affichée, à savoir qu'un identifiant doit être présent et de type entier (integer / serial), il faut penser à ajouter si nécessaire un compteur arbitraire au début de la requête SELECT (ROW_NUMBER() OVER())::integer AS gid.

Il est préférable de forcer le type de géométrie dans la vue pour être correctement intégrée dans geometry_column avec ce paramètre collé à l'attribut de géométrie (geom) `::geometry(polygon,2154)` par ex.

Le tableau ci-dessous indique les principes de dénomination des vues qui découlent de celui des tables. 

|cas|Contenus|Pré-préfixe|Préfixe|exemple|
|:-:|:-:|:-:|:-:|:-:|
|1|données attributaires et géométriques||**geo_v_**|`geo_v_docurba`|
|2|uniquement de la donnée attributaire ||**an_v_**|`an_v_docurba_arcba`|
|3|vue matérilaisée de données attributaires et géométriques||**geo_vmr_**|utilisée uniquement avec les cas 7,8 et 9|
|4|vue matérilaisée en table de données attributaires et géométriques||**geo_vm_**|utilisée uniquement avec les cas 7,8 et 9|
|5|vue matérilaisée de données attributaires||**an_vmr_**|utilisée uniquement avec les cas 7,8 et 9 |
|6|vue matérilaisée en table de données attributaires||**an_vm_**|utilisée uniquement avec les cas 7,8 et 9|
|7|traitement applicatif grand public|**xappspublic_** |préfixe correspondant|`xappspublic_an_vmr_fichegeo_ruplu0_gdpublic`|
|8|traitement applicatif pro|**xapps_** |préfixe correspondant|`xapps_an_v_troncon`|
|9|export OpenData|**xopendata_** |préfixe correspondant|`xopendata_an_v_bal`|

**Rappel :**

Ces préfixes sont suivis de la dénomination classique des tables.

Par défaut, les tables de liens ou de listes de valeur ne peuvent pas faire l'objet d'une vue.

La dénomination des vues doit intégrer l'aspect "emprise géographique" concernée par cette vue. Ex : `geo_v_zone_urba_compiegne` => vue géographique des zonages PLU sur la commune de Compiègne

ATTENTION : Les vues peuvent être commentées, mais l'action de relancer le code CREATE OR REPLACE VIEW sans intégrer la commande COMMENT ON VIEW supprimera le commentaire déjà intégré.



 * **Autres objets** :
 
 Le tableau ci-dessous indique les principes de dénomination des autres objets. 

|Objets|Préfixe|Libellé|suffixe|exemple|
|:-:|:-:|:-:|:-:|:-:|
|index||[nom_table]_[champ indexé]|_idx|`geo_p_zone_urba_geom_idx`|
|séquence||[nom_table]_[champ séquence]|_seq|`geo_a_zone_urba_gid_seq`|
|clé primaire||[nom_table]|_pkey|`geo_p_zone_urba_pkey`|
|clé étrangère||[nom_table]_[champ clé(si nécessaire)]|_fkey|`lt_destdomi_fkey`|
|trigger|t_t(+n° d'ordre d'éxécution)_(r pour trigger générique et m pour trigger spécifiques)|[nom_trigger]||`t_t1_r_l_surf_cal`|
|function trigger (générique)|r_|[nom]||`r_l_surf_cal_ha()`|
|function trigger (spécifique à une table ou vue cas 1)|m_|[nom table]|_[type d'éxécution]|`m_an_doc_urba_null()`|
|function trigger (spécifique à une table ou vue cas 2)|m_|[type d'éxécution]|_[nom table]|`m_geom1_information_surf()`|

