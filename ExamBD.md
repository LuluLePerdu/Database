# Révision

Pour les aeliers : 
- 2 : Bd, schéma et rôles
- 3 : Héritage et partionnage
- 4 : PGSQL procedure / fonction
- 5 : Déclencheur / trigger
- 6 : Curseur et transactions
- 7 : Analyse





## Terminologie

 * BD: Base de données, collection organisée de données entreposé et accédés de manière électronique
 * SGBD: Système de gestion de base de données, permet de définir, créer, maintenir et contrôler l'accès aux bases de données
 * Table: Ensemble de données reliés entre elles sous un même concept
 * Colonne: Entité verticale représentant un ensemble de valeurs pour une donnée. Une colonne est défini avec un nom et un type précis.
 * Ligne: Aussi appelé entrée ou enregistrement. Entité horizontale représentant un ensemble de valeurs représentant la même entitée
 * Clé primaire: 1 ou plusieurs colonnes permettant d'identifier uniquement une ligne dans une table
 * Clé secondaire: Clé primaire copié dans une autre table, ce qui permet de lier les 2 lignes ensembles
 * Relation: Lien entre 2 tables défini avec les clés primaires et secondaires
 * SQL: *Structured Query Language*, langage informatique permettant de chercher, insérer, modifier et supprimer les données. Permet aussi de gérer la base de données et les tables.

## Formes normales

Une base de données en non-normalisée pourrait tout contenir dans une seule table. Une base de données normalisée au maximum n'aurait aucune donnée redondante.

 * 0FN: Non-normalisée
 * 1FN: Contient seulement des valeurs atomiques (non divisible), par exemple au lieu de "nom_complet" on a "nom" et "prenom"
 * 2FN: La table ne contient pas de dépendances partielles. Toutes les valeurs doivent être liés à la même clé primaire (ou aux mêmes clés).
 * 3FN: La table ne contient pas de dépendances transitives. Toutes les valeurs doivent uniquement être liés aux clés primaires, pas à une autre valeur de la table.
 * FNBC (Forme normale Boyce-Codd): Toutes les valeurs dépendent de la clé primaire (dépendance fonctionnelle)
 * 4FN: La table ne contient pas de dépendances multivaleurs. Si une colonne contient plusieurs fois la valeur X et que pour cette valeur une 2e colonne contient toujours les mêmes données, ce n'est pas en 4FN.
 * 5FN: Les valeurs sont séparés sémentiquement. Si une donnée peut être inféré par une autre, il faudrait faire une table de plus.

Une forme normale plus restrictive va:
 * Limiter la redondance
 * Limiter les inconhérences (oublie un enregistrement lorsqu'on déplace des bureaux)
 * Limiter les mises à jours
 * Augmenter la complexité des requêtes SQL
 * Complexifier la base de données

On va arrêter à 3FN, si dans un projet vous ne respectez pas volontairement le 3FN, expliquez moi pourquoi lors de la remise.

Exemple:

0FN

| Produit    | Fournisseur | Adresse                           |
|------------|-------------|-----------------------------------|
| Télévision | LG          | 1234 rue Principale, Tokyo, Japon |

L'adresse n'est pas atomique, on va la diviser pour avoir du 1FN:

| Produit    | Fournisseur | Adresse             | Ville | Pays  |
|------------|-------------|---------------------|-------|-------|
| Télévision | LG          | 1234 rue Principale | Tokyo | Japon |

La clé primaire serait une composition de Produit et Fournisseur, l'adresse dépend seulement du fournisseur, on va la sortir pour avoir du 2FN:

Table Produit:
| Produit    | Fournisseur |
|------------|-------------|
| Télévision | LG          |

Table Fournisseur:
| Fournisseur | Adresse             | Ville | Pays  |
|-------------|---------------------|-------|-------|
| LG          | 1234 rue Principale | Tokyo | Japon |

Dans la table fournisseur, le pays dépend de la ville. Comme la ville n'est pas une clé primaire, on ne l'a pas divisé au 2FN, mais on va le faire pour avoir du 3FN.

Table Fournisseur:
| Fournisseur | Adresse             | Ville |
|-------------|---------------------|-------|
| LG          | 1234 rue Principale | Tokyo |

Table Pays:
| Ville | Pays  |
|-------|-------|
| Tokyo | Japon |

## Normes

Le nom des objets en PostgreSQL (table, colonne, etc.) doivent être en minuscules (notation serpent). Si vous voulez vraiment utiliser des majuscules, vous devez utiliser des guillemets (double-quotes).

Le nom des tables doivent être au pluriel.

On ne met pas de préfixes aux noms des colonnes, si nécessaire renommez les colonnes avec le AS.

# Base de données

Voici le SQL pour créer la database exemple

```sql
CREATE DATABASE exemple
    [ OWNER = user_name ],
    [ TEMPLATE = template ],
    [ ENCODING = encoding ],
    [ STRATEGY = strategy ],
    [ LOCALE = locale ],
    [ LC_COLLATE = lc_collate ],
    [ LC_CTYPE = lc_ctype ],
    [ ICU_CTYPE = icu_locale ],
    [ LOCALE_PROVIDER = locale_provider ],
    [ COLLATION_VERSION = collation_version ],
    [ TABLESPACE = tablespace_name ],
    [ ALLOW_CONNECTIONS = allowconn ],
    [ CONNECTION LIMIT = connlimit ],
    [ IS_TEMPLATE = istemplate ],
    [ OID = oid ]
```

 * user_name: nom de l'utilisateur qui possède la BD, l'utilisateur actuel si omis
 * template: nom du template, `template1` si omis
 * encoding: charset par défaut pour la BD, UTF8 par défaut, donc omettre
 * strategy: comment PostgreSQL copie le template, par défaut c'est déjà le plus efficace
 * locale: change LC_COLLATE et LC_CTYPE en même temps. Faire la requête `SELECT * FROM pg_catalog.pg_collation;` pour voir les locales disponibles.
 * lc_collate à collation_version: selon la locale, change comment PostgreSQL tri les données, pas intéressant pour nous
 * tablespace: permet de spécifier un autre `TABLESPACE` pour cette BD, c-a-d un autre dossier contenant les données de PostgreSQL
 * allowconn: si false on ne peut pas se connecter à la BD, true par défaut. L'admin peut toujours se connecter à la BD
 * connlimit: combien de connections simultanés sont possibles, illimité par défaut (-1)
 * istemplate: *self-explaining*, false par défaut
 * oid: ID de la BD, par défaut PostgreSQL en crée un

## Template

PostgreSQL possède 2 templates par défaut: `template0` est défini par PostgreSQL pour la version actuelle, il ne faut pas le modifier. `template1` est une copie de `template0`, mais vous pouvez le modifier selon vos besoins.

Vous pouvez mettre une vrai BD au lieu des templates lors du `CREATE DATABASE`, mais si un utilisateur se connecte durant la copie, ça crash. La doc suggère d'utiliser `COPY DATABASE` à la place pour ce cas.

## Modifier une BD

Les requêtes sont assez simples à comprendre, ce qui est en minuscule doit être changé selon vos besoins.

```sql
ALTER DATABASE nomdelabd WITH ALLOW_CONNECTIONS allowconn;
ALTER DATABASE nomdelabd WITH CONNECTION LIMIT connlimit;
ALTER DATABASE nomdelabd WITH IS_TEMPLATE istemplate;
ALTER DATABASE nomdelabd RENAME TO nouveaunom;
ALTER DATABASE nomdelabd OWNER TO { new_owner | CURRENT_ROLE | CURRENT_USER }
ALTER DATABASE nomdelabd SET TABLESPACE new_tablespace
ALTER DATABASE nomdelabd REFRESH COLLATION VERSION
ALTER DATABASE nomdelabd SET configuration_parameter TO value
ALTER DATABASE nomdelabd RESET { ALL | configuration_parameter }
```

## Exemple

Créer une BD:

```sql
CREATE DATABASE test
    LOCALE = 'fr_CA.utf8';
```

Supprimer une BD:

```sql
DROP DATABASE test;
```

# Jointures

Considérants les tables suivantes (les FK ont été omis pour faire les INSERT weirds):

```sql
CREATE TABLE a (
    id INT PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
    name TEXT
);

CREATE TABLE b (
    id INT PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
    name TEXT,
    a_id INT
);

INSERT INTO a (id, name) VALUES
    (1, 'aaa'),
    (2, 'aab'),
    (3, 'aac'),
    (4, 'aad');

INSERT INTO b (id, name, a_id) VALUES
    (1, 'bba', NULL),
    (2, 'bbb', 1),
    (3, 'bbc', 3),
    (4, 'bbd', NULL);
```

 * [ INNER ] JOIN: Fait la jointure lorsque ce n'est pas null (a1b2, a3b3)
 * LEFT [ OUTER ] JOIN: Avec `FROM a LEFT JOIN b`, affichera tous les a même s'il n'y a pas de b (a1b2, a2-, a3b3, a4-)
 * RIGHT [ OUTER ] JOIN: L'inverse de left, avec `FROM a RIGHT JOIN b`: (-b1, a1b2, a3b3, -b4)
 * FULL [ OUTER ] JOIN: Donne toutes les possibilités, une union entre le left et le right: (a1b2, a2-, a3b3, a4-, -b1, -b4)

## NATURAL JOIN

Le NATURAL JOIN fait une jointure avec les colonnes du même nom.
Donc si j'ai les tables:

 * categories: cat_id, cat_nom
 * produits: pro_id, pro_nom, cat_id

Les 2 requêtes suivantes utiliseront cat_id pour faire une jointure:

```sql
SELECT * FROM categories JOIN produits USING (cat_id);
SELECT * FROM categories NATURAL JOIN produits;
```

Mais vu qu'on ne met pas de préfixe dans ce cours, ça sert à rien.

## CROSS JOIN

Fait le produit cartésien entre les 2 tables. C'est-à-dire qu'il fait toutes les possibilités de relations même sans FK. Donc si j'ai les table a et b avec 2 enregistrement chaques:

```sql
SELECT * FROM a CROSS JOIN b;

/*
  a1 b1
  a1 b2
  a2 b1
  a2 b2
*/
```

## Aggrégation

Vous pouvez utiliser les fonctions d'aggrégations: `AVG`, `COUNT`, `MAX`, `MIN`, `SUM`.

Je peux faire ces requêtes sur toutes les colonnes, ou faire des sous-totaux.

Exemples:

```sql
CREATE TABLE films (
    id INT PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
    nom TEXT,
    cote SMALLINT CHECK (cote >= 1 OR cote <= 5>),
    categorie_id INT
);

INSERT ...

SELECT AVG(cote) FROM films; -- Moyenne de tous les films
SELECT categorie_id, AVG(cote) FROM films GROUP BY categorie_id; -- Moyenne par catégorie
```

## HAVING ou WHERE

WHERE permet de filtrer les enregistrements.

HAVING permet de filtrer les aggrégats.

L'exemple suivant fait la moyenne des cote plus grandes que 2 et n'affiche que les catégories ayant une cote moyenne plus grande que 3 (moyenne fait après le WHERE):

```sql
SELECT AVG(cote) avg_cote FROM films WHERE cote > 2 GROUP BY categorie_id HAVING AVG(cote) > 3;
```

## Multiples requêtes

 * `UNION`: Combine 2 requêtes ensembles
 * `INTERSECT`: Garde les enregistrements présents dans chaque requête
 * `EXCEPT`: Garde les enregistrements de la 1ère requête et qui ne sont pas dans la 2e requête

```sql
SELECT * FROM a1
[ UNION | INTERSECT | EXCEPT ]
SELECT * FROM a2;
```

# Aléatoire

La fonction prédéfinie `random()` donne un nombre entre 0 et 1. Pour obtenir un nombre entre 1 et 10, vous pouvez exécuter l'instruction suivante:

```pgsql
SELECT floor(random() * 9 + 1)::int;
--ou
rnd := floor(random() * 9 + 1);
```

Pour générer plusieurs nombres aléatoires avec le SELECT, vous pouvez utiliser:

```pgsql
SELECT random_between(1,100)
FROM generate_series(1,5);
```

## Tableau

PostgreSQL accepte les tableaux autant dans les colonnes que dans PGSQL:

```pgsql
ARRAY[1, 2, 3];
ARRAY['abc', 'ble'];
'{1,2,3}'
'{"abc", "qwe"}'
```


# Récursivité

PostgreSQL permet les requêtes récursives. Exemple:

J'ai la table suivante:

```sql
CREATE TABLE employes (
    id INT GENERATED BY DEFAULT AS IDENTITY,
    nom TEXT NOT NULL,
    superieur_id INT
);
```

La colonne `superieur_id` contient l'ID du supérieur, ou NULL s'il n'y en a pas.

J'ai, entre autre, les employés suivants:

```sql
INSERT INTO employes (id, nom, superieur_id) VALUES
    (1, 'Berte', NULL),
    (2, 'Gaston', 1),
    (3, 'Euclyde', 2);
```

La requête suivante permet d'avoir la liste de Euclyde jusqu'à Berte:

```sql
WITH RECURSIVE boss AS (
    SELECT *
    FROM employes
    WHERE id = 3
    UNION
    SELECT e.*
    FROM employes e
    JOIN boss b ON e.id = b.superieur_id
)
SELECT *
FROM boss;
```

Explications:

Dans la fonction récursive, vous avez 2 SELECT séparés par un UNION. Le premier SELECT définis l'enregistrement de base. Le 2e SELECT définis la récusion, il faut renommer la table avec un AS (implicite dans l'exemple) pour éviter les conflits avec le premier SELECT.

En détail, Postgresql commence avec une "table" boss qui est vide. Il ajoute Euclyde (1er SELECT), ensuite fait un SELECT avec le dernier enregistrement de boss (boss de Euclyde est Gaston). Il fait une autre récursion sur le dernier enregistrement de boss (boss de Gaston est Berte). Comme Berte n'a pas de boss (superieur_id est null), on arrête la récursion.

# Schéma

Un schéma permet d'organiser les tables, de les grouper ensembles. C'est comparable à un namespace en programmation.

Par défaut il y a le schema public, si vous utilisez plusieurs schemas c'est conseillé de supprimer public. Par défaut tous les utilisateurs ont accès au schema public.

## SQL

Les requêtes sont très simples:

```sql
CREATE SCHEMA [IF NOT EXISTS] nomduschema;
ALTER SCHEMA nomduschema RENAME TO nouveaunom;
DROP SCHEMA [IF EXISTS] nomduschema [CASCADE];
```

Par défaut le DROP est en mode RESTRICT, ce qui veut dire qu'on ne peut pas supprimer le schéma s'il contient quelque chose. Le CASCADE dit de supprimer le schéma et tout ce qu'il contient.

Pour créer une table dans un schema:

```sql
CREATE TABLE nomduschema.nomdelatable ...
```

Et pour la changer de schéma:

```sql
ALTER TABLE [IF EXISTS] nomdelatable SET SCHEMA nouveauschema
```

## Chemin de recherche

Lorsqu'on a plusieurs schémas, vous pouvez les spécifier dans vos requêtes (nomduschema.nomdelatable). Si on ne spécifie pas le nom du schéma, PostgreSQL va essayer de trouver la table en cherchant dans le *search path*. Pour voir le search path actuel:

```sql
SHOW search_path;
```

Pour changer le search path:

```sql
SET search_path TO sales, hr, admin;
```

Notez que PostgreSQL cherche toujours dans le schéma qui a le même nom que l'utilisateur actuel, s'il existe.

# Tables

## Création

```sql
CREATE TABLE [IF NOT EXISTS] nom_de_la_table (
    colonnes...
    contraintes...
);
```

Les colonnes sont définis par le type, suivit du nom et des particularités, par exemple:

```sql
id SERIAL PRIMARY KEY,
quantite INT NOT NULL,
nom VARCHAR(50) UNIQUE NOT NULL
```

La majorité des particularités des colonnes peuvent être ajoutés dans les contraintes.

### CHECK

On peut ajouter un `CHECK` sur les colonnes. Le `NOT NULL` de l'exemple précédent est l'équivalent de `CHECK(quantite IS NOT NULL)`. Je pourrais aussi mettre un `CHECK(quantite > 0)` ou `CHECK(quantite IS NOT NULL AND quantite > 0)`.

Je peux ajouter un `CHECK` dans les contraintes, par exemple une contrainte qui fait un lien entre 2 colonnes:

```sql
ADD CONSTRAINT price_discount_check 
CHECK (
	price > 0
	AND discount >= 0
	AND price > discount
);
```

Par convention, le nom d'une contrainte est: `nomdelatable_nomdelacolonne_typedecontrainte`.

### Clé primaire

Vous pouvez identifier une clé primaire dans les particularités de la colonne ou dans les contraintes:

```sql
CREATE TABLE users (
   id serial PRIMARY KEY
);
CREATE TABLE account_roles (
  id INT NOT NULL,
  PRIMARY KEY (id)
);
```

Si votre clé primaire est composé de plusieurs colonnes, vous devez utiliser la deuxième forme.

### Clé secondaire

Une clé secondaire se déclare dans les contraintes:

```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  role_id INT NOT NULL,
  FOREIGN KEY (role_id)
      REFERENCES roles (id)
);
```

Et voici un exemple pour une table de jointure (table avec 2 colonnes qui sont PK et FK):

```sql
CREATE TABLE account_roles (
  user_id INT NOT NULL,
  role_id INT NOT NULL,
  PRIMARY KEY (user_id, role_id),
  FOREIGN KEY (role_id)
      REFERENCES roles (id),
  FOREIGN KEY (user_id)
      REFERENCES accounts (id)
);
```

### Valeur par défaut

Vous pouvez mettre une valeur par défaut à une colonne, utile pour rendre une colonne `NOT NULL` optionnelle.

Pour les colonnes de type date et timestamp, vous pouvez utiliser les mots-clés `NOW` et `CURRENT_DATE` (équivalent à `NOW()::date`).

Exemple:

```sql
CREATE TABLE users (
	username TEXT NOT NULL DEFAULT 'admin',
	register_date DATE NOT NULL DEFAULT CURRENT_DATE
)
```

## Héritage

Une table peut hériter d'une autre (exemple de: https://www.postgresql.org/docs/current/ddl-inherit.html):

```sql
CREATE TABLE cities (
    name            text,
    population      float,
    elevation       int     -- in feet
);

CREATE TABLE capitals (
    state           char(2)
) INHERITS (cities);
```

De cette manière on n'a pas de colonne "state" nullable ou de FK supplémentaire entre les 2 tables.
En arrière, de manière caché il y a une FK géré automatiquement. Aussi, en ayant 2 tables ça réduit les tables ce qui accélère le processus.
Les rêquettes SQL vont être très simple:

```sql
INSERT INTO cities (name, population, elevation) VALUES ('St-Félix', 5, 12);
INSERT INTO capitals (name, population, elevation, state) VALUES ('Joliette', 42, 12, 'LA');

SELECT * FROM cities;
SELECT * FROM capitals;
```

Si vous voulez savoir si 'St-Félix' ou 'Joliette' provient de la table cities ou capitals, il faut interroger pg_class.

```sql
SELECT p.relname, c.* FROM cities c, pg_class p WHERE c.tableoid = p.oid;
```

Vous pouvez aussi avoir toutes les colonnes avec le UNION:

```sql
SELECT *, NULL FROM cities UNION SELECT * FROM ONLY capitals;
```

## Partition

Des tables trop volumineuses peuvent ralentir le système. La maintenance de ces tables (supprimer les enregistrements trop vieux) sont aussi plus difficiles. Par exemple, une table qui journalise toutes les actions des utilisateurs. Les partitions permettent de créer des sous-tables à une table principale. Exemple de https://www.postgresql.org/docs/10/ddl-partitioning.html :

```sql
CREATE TABLE measurement (
    city_id         int not null,
    logdate         date not null,
    peaktemp        int,
    unitsales       int
) PARTITION BY RANGE (logdate);

CREATE TABLE measurement_y2023m08 PARTITION OF measurement
    FOR VALUES FROM ('2023-08-01') TO ('2023-09-01');
CREATE TABLE measurement_y2023m09 PARTITION OF measurement
    FOR VALUES FROM ('2023-09-01') TO ('2023-10-01');

INSERT INTO measurement (city_id, logdate, peaktemp, unitsales) VALUES (1, '2023-09-10', 30, 20);
```

Le INSERT va ajouter la donnée dans la bonne table selon les bornes. Faire du ménage consiste simplement à supprimer une partition ce qui est très rapide.

Notez qu'un INSERT dans un range inexistant ne fonctionnera pas.

Je peux SELECT sur `measurement` pour avoir toutes les données, ou juste sur une partition pour avoir juste l'échantillon souhaité.

Dans l'exemple précédent, la borne minimale est inclus tandis que la borne maximale est exclus. Vous pouvez utiliser les valeurs `MINVALUE` et `MAXVALUE` pour enlever la borne minimale et maximale, par exemple:

```sql
CREATE TABLE measurement_past PARTITION OF measurement
    FOR VALUES FROM (MINVALUE) TO ('2023-08-01');
CREATE TABLE measurement_futur PARTITION OF measurement
    FOR VALUES FROM ('2024-01-01') TO (MAXVALUE);
```

Au lieu de la forme "RANGE" avec le "FROM ... TO", vous pouvez utiliser le "LIST" avec le "IN":

```sql
CREATE TABLE measurement (
    city_id         int not null,
    logdate         date not null,
    peaktemp        int,
    unitsales       int
) PARTITION BY LIST (city_id);

CREATE TABLE measurement_city01 PARTITION OF measurement
    FOR VALUES IN (1);
CREATE TABLE measurement_city02 PARTITION OF measurement
    FOR VALUES IN (2);
```

Notez qu'un update qui change un enregistrement de partition doit se faire sur la table mère, pas sur l'enfant (un update de doit pas changer de table).

# Types

## Boolean

`boolean` ou `bool`

Lorsqu'on insert/update, PostgreSQL convertis les valeurs:

 * 't', 'true', 'y', 'yes', '1', 1 et true => true
 * 'f', 'false', 'n', 'no', '0', 0 et false => false

Lorsqu'on fait un SELECT, PostgreSQL va afficher le booléen:

 * true => t
 * false => f
 * null => espace

## Texte

Les 3 "sortes" de champs texte sont entreposés de la même manière, dans la table TOAST. On ne sauve donc pas d'espace avec des types plus limités. On va limiter pour éviter les erreurs seulement.

### char(1)

`char`

Type ne permettant qu'une seule lettre.

### char

`character(n)` ou `char(n)`

Type pour les textes de longeur fixe, si le texte est plus court PostgreSQL va ajouter des espaces à la fin jusqu'à la limite. Le *n* dans le type correspont à la taille maximale. Si la limite est dépassé PostgreSQL va refuser le insert/update.

Possibilité la plus lente car PostgreSQL va vérifier la longueur et remplir si nécessaire.

Utile quand la taille est toujours fixe comme un *zip code*, un code postal ou une clé de licence.

### varchar

`character varying(n)` ou `varchar(n)`

Champ texte à longeur limite, mais contrairement à `char`, ne remplis pas avec des espaces.

Type plus rapide que `char` car il ne fera pas de remplissage, mais plus lent que `texte` car il fera une vérification.

Utile pour limiter les abus (username de 10000000 de caractères) ou lorsque ce système est synchronisé à un système qui présente cette limite.

### texte

`texte` ou `varchar`

Champ texte sans limite.

Utilisez ce type 99% du temps, mais il faut empêcher les abus via l'interface graphique.

## Numérique

### integer

`smallint`, `int` ou `bigint`. Sont les mêmes que `int2`, `int4` et `int8` dans l'ordre. `integer` peut être utilisé au lieu du `int`.

Les 3 types ont respectivement une longueur de 2, 4 et 8 octets. Le `unsigned` n'existe pas en PostgreSQL.

### réel exact

`decimal` ou `numeric`

Entreposé comme une chaîne, ce qui le rend précis mais plus lent pour faire des calculs dans la BD.

Lorsque vous déclarez le type, vous pouvez spécifier la précision et l'échelle (`scale`):

 * Précision: Nombre de chiffre maximal au total dans le nombre
 * Échelle: Nombre de chiffres maximal après la virgule

Exemple:

```
NUMERIC(4, 2) // Permet 1234 et 12.34
NUMERIC(4)    // Permet 4 chiffres, donc 0.123 ou 1234
NUMERIC       // Permet jusqu'à 131072 chiffres avant la virgule et 16383 après
```

La précision peut être de -1000 à 1000 et l'échelle de 0 à *précision*.

Si la précision est négative, on va insérer des plus gros chiffres. Par exemple `NUMERIC(2, -3)` peut contenu de -99000 à 99000. Bref, l'échelle arrondis d'un côté ou de l'autre de la virgule et la précision détermine le nombre de chiffres.

Si on insère plus de décimales que l'échelle, PostgreSQL va arrondir lui-même. Mais si on insère plus que la précision, PostgreSQL va refuser la valeur.

Le type `NUMERIC` (sans échelle ni précision) accepte `Infinity` et `-Infinity` ou `inf` et `-inf`. Ces valeurs doivent être des strings lors de la saisie.

C'est aussi possible d'avoir `NaN`. Lors d'un tri, `NaN` est la plus grande valeur. `NaN` n'est pas égal aux autres `NaN`.

### Réel approximatif

Entreposé en binaire, ce qui accélère les calculs et réduis l'espace disque nécessaire à l'entreposage des données. Par contre, certains nombres comme 0.6 ne sont pas possible en binaire ce qui empêche d'avoir le nombre exacte.

`real`, `float4` ou `float(24)` 1E-37 à 1E+37, précision de 6 décimales.

`double precision`, `float8` ou `float` 1E-307 à 1E+308, précision de 15 décimales.

### Serial

`smallserial`, `serial` et `bigserial`, des entiers signé de 2, 4 et 8 octets ne pouvant pas être négatifs.

Un exemple venant de: https://www.postgresql.org/docs/current/datatype-numeric.html

Écrire:

```sql
CREATE TABLE tablename (
    colname SERIAL
);
```

Reviens au même que de faire:

```sql
CREATE SEQUENCE tablename_colname_seq AS integer;
CREATE TABLE tablename (
    colname integer NOT NULL DEFAULT nextval('tablename_colname_seq')
);
ALTER SEQUENCE tablename_colname_seq OWNED BY tablename.colname;
```

Les types serial sont utiles pour les entiers qui doivent s'incrémenter automatiquement, mais pour les PK (id) nous allons voir une meilleure technique plus tard.

## Dates et heures

### Date

`date` représente la date sans heure. Une date est écrite comme une string (avec les `'`) avec le format iso, par exemple: `'2023-12-31'`.

PostgreSQL va afficher le résultat avec le format ISO, mais on peut changer avec la fonction TO_CHAR:

```sql
SELECT TO_CHAR(CURRENT_DATE, 'dd/mm/yyyy');
```

La fonction TO_CHAR utilise les lettres suivantes:

 * YY ou YYYY, année sur 4 chiffres
 * MON, mois en lettre sur 3 lettres (jan pour january)
 * MONTH, mois en lettre complet
 * MM, mois en chiffre (janvier = 1)
 * WW, numéro de semaine dans l'année
 * DY, jour de la semaine sur 3 lettre
 * DD, jour du mois
 * HH24, heure en format 24h
 * HH ou HH12, heure en format 12h
 * MI, minutes
 * SS, secondes

Dans le cas de MONTH, la casse du format sera la même que le résultat, donc MONTH sortira JANUARY tandis que Month sortira January.

La langue doit être définis à la création de la bd, je n'ai pas trouvé comment changer sur demande.

Vous pouvez aussi utiliser la fonction AGE, par exemple `AGE(birth_date)` donne un résultat comme `36 years 5 mons 22 days`.

Si vous voulez juste l'année, utilisez la fonction EXTRACT, par exemple `EXTRACT(YEAR FROM birth_date)` qui va afficher `1992`. Les champs possibles sont: CENTURY, DAY, DECADE, DOW (day of week), DOY (day of year), EPOCH (unix timestamp), HOUR, ISODOW, ISOYEAR, MICROSECONDS, MILLENNIUM, MILLISECONDS, MINUTE, MONTH, QUARTER, SECOND, TIMEZONE, TIMEZONE_HOUR, TIMEZONE_MINUTE, WEEK, YEAR.

### Timestamp

`timestamp` sans timezone ou si vous voulez une timezone: `timestamptz` ou `timestamp with timezone`.

Un timestamp ressemble à `2023-08-25 08:58:55.082951` et avec un timezone: `2023-08-25 08:58:55.082951 PDT`.

`NOW()` ou `CURRENT_TIMESTAMP` donne le timestamp avec timezone. `TIMEOFDAY()` donne le timestamp avec timezone en string. `CURRENT_TIME` donne l'heure sans la date.

Vous pouvez configurer la timezone du serveur DB et la consulter avec les commandes:

```sql
SET timezone = 'America/Montreal';
SHOW TIMEZONE;
```

Notre timezone sur 3 lettre est `EDT` pour `Eastern Time`.

Pour afficher un timestamp dans un autre timezone, utilisez la fonction timezone. L'exemple suivant va afficher l'heure d'ici suivit de l'heure selon le timezone de PostgreSQL (probablement UTC).

`SELECT timezone('America/Montreal', NOW()), NOW();`

Le résultat fait à 08:58 ressemble à:
`2023-08-25 08:58:55.082951, 2023-08-25 12:58:55.082951 +00:00`.

Vous pouvez aussi changer le timezone avec `at time zone`, par exemple: `SELECT NOW() at time zone 'utc'`.

Pour avoir des dates en français, voici des exemples:

```sql
select to_char(AGE(timestamp '2023-1-10'), 'YY "années" MM "mois" DD "jours"');
select to_char(current_date, 'dd TMMonth YY')
```

# Utilisateurs

Appelés role dans PostgreSQL. Le compte admin créé à l'installation de PostgreSQL est un rôle avec un privilège SUPERUSER.

Voici les options possibles pour créer un rôle, les choix en gras sont la valeur par défaut:

 * SUPERUSER | **NOSUPERUSER**: Un SUPERUSER est un utilisateur admin, il peut tout faire
 * CREATEDB | **NOCREATEDB**: Donne le droit de créer des DB
 * CREATEROLE | **NOCREATEROLE**: Donne le droit de gérer les rôles
 * **INHERIT** | NOINHERIT: Hérite des droits données à tous les rôles
 * LOGIN | **NOLOGIN**: Avec LOGIN le rôle devient un utilisateur, il peut s'authentifier. Avec NOLOGIN le rôle peut être vu comme un groupe.
 * REPLICATION | **NOREPLICATION**: Permet de gérer la réplication de serveurs dans le cas d'instance multiple en *load-balancing*
 * BYPASSRLS | **NOBYPASSRLS**: Le RLS (Row Level Security) permet de créer des règles plus stricts sur les tables, par exemple seulement certains rôles peuvent modifier les commandes traités.
 * CONNECTION LIMIT: Par défaut -1, limite de connection simultanés
 * PASSWORD 'motdepasse': Sans cette commande le mot de passe est null et l'utilisateur ne peut pas se connecter. Si le mot de passe est déjà encripté, PostgreSQL le sauvegarde telquel
 * VALID UNTIL 'timestamp': Désactive le rôle au moment donné. Si omis, aucune date limite.
 * IN ROLE nomdurole: Liste de rôles que ce rôle va devenir membre
 * ROLE nomdurole: Inverse du précédent, liste de rôles qui vont être dans ce rôle, le rôle actuel devient un groupe
 * ADMIN nomdurole: Comme le précédent, mais les rôles peuvent ajouter plus de rôle dans le groupe
 * IN GROUP, USER et SYSID: Deprecated

Exemples:

```sql
CREATE ROLE michel WITH LOGIN PASSWORD 'vive_mac';
CREATE USER michel WITH PASSWORD 'vive_mac';
CREATE ROLE admin WITH CREATEDB CREATEROLE ROLE michel; 
```

Pour supprimer un rôle:

```sql
DROP ROLE [IF EXISTS] michel;
```

Mais vous ne pouvez pas supprimer un rôle qui possède du contenu, vous pouvez exécuter une des commandes suivante:

```sql
REASSIGN OWNED BY michel TO { nom_du_role | CURRENT_ROLE }
DROP OWNED BY michel [CASCADE]
```

## Propriétaire

Le propriétaire de chaque objet dans PostgreSQL a tous les droits sur cet objet.

Pour la base de données, c'est simple:

```sql
CREATE DATABASE nom_de_la_bd
  WITH OWNER michel;

ALTER DATABASE nom_de_la_bd OWNER TO { michel | CURRENT_ROLE };
```

Pour un schéma:

```sql
CREATE SCHEMA nom_du_schema AUTHORIZATION michel;
CREATE SCHEMA AUTHORIZATION michel; /* Schéma michel */

ALTER SCHEMA nom_du_schema OWNER TO { michel | CURRENT_ROLE };
```

Pour un `CREATE TABLE`, la nouvelle table appartient à l'utilisateur actuel. Vous pouvez changer le propriétaire par la suite:

```sql
ALTER TABLE nom_de_la_table OWNER TO { michel | CURRENT_ROLE }
```

## Permissions

On peut donner des droits à un rôle non-propriétaire.

Par exemple:

```sql
GRANT SELECT ON students TO michel;
```

Le rôle michel peut maintenant faire des select sur la table students, même s'il ne peut pas insert ni update.

Vous pouvez donner les permissions suivantes:

 * SELECT: Nécessaire pour les requêtes SELECT, UPDATE, DELETE et MERGE
 * INSERT
 * UPDATE
 * DELETE
 * TRUNCATE
 * REFERENCES: Pour les FK
 * TRIGGER
 * CREATE
 * USAGE: Variable, pour un schéma ça permet de voir le schéma
 * CONNECT: Pour se connecter à la DB

Les permissions peuvent se donner sur les tables, schéma ou BD. On peut même donner des droits plus strict sur une table:

```sql
GRANT UPDATE (password) ON users TO michel;
```

Michel aurait juste le droit de modifier la colonne password pour la table users.

Voici un autre exemple pour un schéma:

```sql
GRANT ALL ON ALL TABLES IN SCHEMA hr TO michel;
```

# Vues

Une vue est une table logique, c-a-d un alias sur une requête SELECT. Un utilisateur doit avoir accès à toutes les tables et colonnes utilisés par la vue.

La performance va être similaire à une fonction en SQL pure, mais est beaucoup plus simple à créer/gérer. Une vue se comporte comme une table normale, mais permet seulement les SELECT.

## Vue de base

```pgsql
CREATE OR REPLACE VIEW view_name AS select_query...;
```

Exemple:

```pgsql
CREATE VIEW utilisateurs_actifs AS
    SELECT *
    FROM utilisateurs
    WHERE supprime IS NULL;

SELECT * FROM utilisateurs_actifs;
```

On peut renommer:

```pgsql
ALTER VIEW utilisateurs_actifs RENAME TO users_enabled;
```

On peut supprimer:

```pgsql
DROP VIEW IF EXISTS users_enabled;
```

## Vue modifiable

Si les conditions suivantes sont respectés, vous pouvez INSERT, UPDATE et DELETE dans votre vue:

 * La vue n'utilise qu'une seule table
 * La requête n'utilise pas: GROUP BY, HAVING, LIMIT, OFFSET, DISTINCT, WITH, UNION, INTERSECT et EXCEPT (bref, aggrégation et combinaison)
 * La requête n'utilise pas: fonction window, sous-requête, fonction d'aggrégat (SUM, COUNT, MIN, ...)

Bref, la vue précédente (utilisateurs_actifs) respectent ces conditions et est une vue modifiable.

## Vues matérialisés

Une vue matérialisé est sauvegardé sur le disque, vous pouvez seulement faire des SELECT sur ce type de vue.

Avantages/désavantages d'une vue matérialisé:

 * Prend plus de l'espace disque
 * Doit être mis à jour manuellement, donc pas 100% à jour
 * Des exemples montre des gains de performances de plus de 1000x

Ce type de table est très utile pour les grosses analyses qui peuvent prendre beaucoup de temps avec beaucoup de requêtes, comme l'IA et l'analyse de données.

Créer une vue matérialisé:

```pgsql
CREATE MATERIALIZED VIEW nom_vue AS requete... WITH NO DATA;
```
Le `WITH NO DATA` est optionnel, ça dit de créer la vue vide, on pourra peupler la vue par la suite.

Pour la mettre à jour:

```pgsql
REFRESH MATERIALIZED VIEW nom_vue;
```

Le temps du refresh, la table n'est plus utilisable. Pour éviter le problème, vous devez avoir au moins un `UNIQUE INDEX` (comme un ID):

```pgsql
REFRESH MATERIALIZED VIEW CONCURRENTLY nom_vue;
```

# Fonction fenêtre (window function)

Une fonction d'aggrégation avec `GROUP BY` (comme `AVG`, `COUNT`, `MAX`, etc.) assemble plusieurs lignes ensembles (ce qui réduit le nombre de résultat) et fait un calcul sur les enregistrements combinés. Une fonction fenêtre est similaire, mais ne réduit pas le nombre de lignes.

Donc par exemple, avec une table produit (id, prix, quantite, categorie_id), avec une fonction d'aggrégat je pourrait sortir la moyenne des prix de chaque catégorie. Avec une fonction fenêtre je pourrais avoir ma liste de produit avec une colonne supplémentaire qui contient la moyenne de la catégorie de ce produit.

La requête de l'exemple précédent donnerait:

```sql
SELECT
    produits.*,
    AVG(prix) OVER (
        PARTITION BY categorie_id
    ),
FROM
    produits;
```

Si vous faites la même fenêtre plusieurs fois, vous pouvz utiliser une close cadre (*frame clause*):

```sql
--Mauvais
SELECT
    produits.*,
    AVG(prix) OVER (
        PARTITION BY categorie_id
    ),
    MAX(prix) OVER (
        PARTITION BY categorie_id
    ),
FROM
    produits;

--Bon
SELECT
    produits.*,
    AVG(prix) OVER w,
    MAX(prix) OVER w,
FROM produits
WINDOW w AS (PARTITION BY categorie_id)
```

Pour certains fonction fenêtre, l'ordre des enregistrement est importante, vous pouvez donc trier une fenêtre par la clause `ORDER BY`:

```
ORDER BY prix;
ORDER BY prix ASC NULLS LAST;
ORDER BY prix DESC NULLS FIRST;
```

## Fonctions fenêtres

Les fonctions d'aggrégats fonctionnent pour les fonctions fenêtres (`AVG, MIN, MAX, SUM, COUNT`).

En plus vous avez:

 * ROW_NUMBER(): Écrit le numéro de ligne pour la fenêtre (ex. 1, 2, 3, 4, 1, 2, 3)
 * RANK(): Idem, mais si 2 enregistrements ont le même "ORDER BY", ils vont avoir le même chiffre. (ex. 1, 2, 2, 4)
 * DENSE_RANK(): Idem, mais on ne passe pas de chiffre si un chiffre est répété. (ex. 1, 2, 2, 3)
 * PERCENT_RANK(): Comme RANK, mais donne le numéro en pourcentage. Le dernier vaut 1, le milieu 0,5.
 * CUME_DIST(): Similaire à PERCENT_RANK, mais donne le nombre de lignes en pourcentages qui ont une valeur égale ou moindre.
 * NTILE(buckets): Buckets est un int qui représente le nombre de groupes égaux à créer. Par exemple, si j'ai 3 buckets le premier tier aura le chiffre 1, le 2e tier aura 2 et le dernier tier aura 3. Exemple 2: si ma fenêtre a 7 enregistrements et que j'ai 3 buckets, j'aurai les chiffres: 1, 1, 1, 2, 2, 3, 3

 * FIRST_VALUE(colonne): Affiche la valeur du premier enregistrement dans la fenêtre
 * LAST_VALUE(colonne): Idem, mais pour le dernier
 * NTH_VALUE(colonne, no): Idem, mais pour le "no" enregistrement (le 2e par exemple)

 * LAG(colonne, [offset]): Donne le résultat de l'enregistrement précédent dans la fenêtre, utile pour faire des différences. NULL si c'est le premier, offset vaut 1 par défaut.
 * LEAD(colonne, [offset]): Même chose, mais pour l'enregistrement suivant


# Curseurs

Un curseur permet de contrôler le parcours d'enregistrements. Les curseurs permettent de ne pas charger tout le résultat en mémoire, juste 1 enregistrement à la fois. Un `for loop` possède un curseur implicite afin de ne pas faire d'overflow de mémoire.

## Déclaration

```pgsql
DECLARE
    curs1 refcursor;
    curs2 CURSOR FOR SELECT * FROM tenk1;
    curs3 CURSOR (key integer) FOR SELECT * FROM tenk1 WHERE unique1 = key;
```

Les 3 curseurs sont de type `refcursor`, par contre le premier est valide pour n'importe quelle requête. Les 2 autres sont des curseurs liés (bound cursor). Le dernier est un curseur lié avec paramètre.

Dans une procédure, c'est possible de retourner un curseur (type refcursor).

## Ouverture

Il faut ouvrir un curseur afin de pouvoir l'utiliser. Le curseur non-lié s'ouvre en spécifiant la requête:

```pgsql
OPEN curs1 FOR SELECT * FROM foo WHERE key = mykey;
OPEN curs1 FOR EXECUTE format('SELECT * FROM %I WHERE col1 = $1',tabname) USING keyvalue;
```

Le EXECUTE permet d'exécuter une requête en format String.

Dans le cas d'un curseur lié, il faut simplement l'ouvrir:

```pgsql
OPEN curs2;
OPEN curs3(42);
OPEN curs3(key := 42);
```

Exemple avec un curseur lié à une autre variable:

```pgsql
DECLARE
    key integer;
    curs4 CURSOR FOR SELECT * FROM tenk1 WHERE unique1 = key;
BEGIN
    key := 42;
    OPEN curs4;
```

## Manipulation

### Fetch

```pgsql
FETCH [ direction { FROM | IN } ] cursor INTO target;
```

Fetch va chercher le prochain enregistrement. Vous pouvez mettre le résultat dans une variable de type `record` ou dans plusieurs variables des mêmes nombres et types que les colonnes. On peut aussi spécifier une direction:

 * NEXT (défaut)
 * PRIOR
 * FIRST
 * LAST
 * ABSOLUTE count
 * RELATIVE count
 * FORWARD
 * BACKWARD

Exemples:

```pgsql
FETCH curs1 INTO rowvar;
FETCH curs2 INTO foo, bar, baz;
FETCH LAST FROM curs3 INTO x, y;
FETCH RELATIVE -2 FROM curs4 INTO x;
```

### Move

```pgsql
MOVE [ direction { FROM | IN } ] cursor;
```

Comme Fetch, mais ne retourne aucune donnée. Ça permet de bouger le curseur sans prendre de place en mémoire.

Exemples:

```pgsql
MOVE curs1;
MOVE LAST FROM curs3;
MOVE RELATIVE -2 FROM curs4;
MOVE FORWARD 2 FROM curs4;
```

### Update et delete

Vous pouvez faire des updates et deletes directement sur le curseur:

```pgsql
UPDATE table SET ... WHERE CURRENT OF cursor;
DELETE FROM table WHERE CURRENT OF cursor;
```

### Close

Un curseur est fermé automatiquement à la fin d'une transaction, mais c'est une bonne pratique de les fermer manuellement:

```pgsql
CLOSE cursor;
```

### Loop

Plusieurs manières de faire des boucles sur des curseurs:

```pgsql
FOR enregistrement IN curseur LOOP
    statements
END LOOP;
```

```pgsql
loop
    fetch cur_films into rec_film;
    exit when not found;
    ...
end loop;
```

# Explique (explain)

Cette fonction permet d'analyser la performance d'une requête PostgreSQL. La forme la plus simple est:

```pgsql
EXPLAIN SELECT * FROM users;
```

Cette fonction estime le coût d'une requête, mesuré en "*disk page fetches*". Ce coût est une estimation selon plusieurs variables (nombres de lectures disques, taille du résultat, taille des tables, index, etc.) Vous allez voir 2 chiffres, le premier est le coût pour avoir le 1er enregistrement et le 2e est le coût pour tout avoir.

À la suite du EXPLAIN, vous pouvez mettre des options:

 * ANALYSE: Par défaut PostgreSQL fait une estimation, l'analyse donne plus de détail et exécute la requête pour vrai.
 * VERBOSE: Donne plus de détail, incluant les variables.
 * COSTS: Affiche le coût (TRUE par défaut)
 * BUFFERS: Affiche le détail des buffers, seulement disponible avec le VERBOSE
 * TIMING: Affiche le temps d'exécution de chaque étape, seulement disponible avec le VERBOSE
 * SUMMARY: Ajoute un temps d'exécution total à la fin, actif automatiquement avec le ANALYSE
 * FORMAT: Par défaut en TEXT, vous pouvez voir le résultat en XML, JSON et YAML

Si votre requête à analyser fait des INSERT ou UPDATE, vous pouvez utiliser une transaction:

```pgsql
BEGIN;
    EXPLAIN ANALYSE procedure_stocke();
ROLLBACK;
```

# Historique PostgreSQL

## Michael Stonebraker

C'est le fondateur de Ingres et PostgreSQL.
Il a été assistant professeur à la *University of California, Berkley* de 1971 à 2000. Depuis 2001 il est professeur au MIT, il a reçu le prix Turing en 2014 pour son travail dans les bases de données relationnelles.


## Ingres

Diminutif de _**IN**teractive **G**raphics **RE**trieval **S**ystem_, la première version est sortie en 1974. C'est une des premières base de données relationnelle, comparable au *System R* de IBM. En 1980 il était même comparé à Oracle, considéré comme le meilleur SGBD au monde (encore aujourd'hui). Par contre Oracle a vanté les bienfaits du SQL, ce qui a fait perdre de la popularité à Ingres qui utilisait QUEL (_**QUE**ry **L**angage_).
Au début le code était OpenSource (à condition d'acheter les *tapes*), mais est vite devenu propriétaire.

## PostgreSQL

Michael a débuté la recherche sur ce nouveau SGBD en 1985 et a sorti une première version en 1989. Au début le SGBD s'appelait POSTGRES pour _**POST** in**GRES**_. Le système utilisait encore QUEL jusqu'en 1996 où il a passé au SQL, renommant le SGBD pour PostgreSQL.
Depuis le début et encore aujourd'hui, PostgreSQL est *open source*, multiplateforme et gratuit. Il est souvent comparé à Orcale (qui est le meilleur SGBD).

Côté performance PostgreSQL est excellent, chaque requête ouvre un thread ce qui permet plusieurs connexions simultanés. Il est même plus rapide à faire du noSQL que mongodb, un SGBD spécialisé pour le noSQL.

## Autres SGBD

Plusieurs autres SGBD existent, lequel utiliser selon votre projet?

### MS Access

Bon système pour un néophyte qui utilise Windows. Permet seulement 1 écriture à la fois ce qui est mauvais si vous avez plusieurs utilisateurs.

Vous qui êtes des experts, évitez cette solution à moins d'une demande client.

### SQL Server

Très bon SGBD, comparable à PostgreSQL sur plusieurs points. Si vous avez un réseau Microsoft, SQL Server peut se brancher à votre LDAP afin de gérer les accès plus facilement.

À utiliser dans une architecture 100% Microsoft, ce n'est pas un mauvais choix dans les autres cas.

### SQLite

Pas de gestion de droit, 1 seul accès à la fois, la BD se trouve dans un fichier texte.

À utiliser pour un développement local rapide, à éviter pour une BD en production.

### MongoDB

SGBD noSQL très populaire et très perfomante.

À utiliser si vous utiliser JavaScript avec une API très simple comme nous allons faire en Projet Web. À éviter dans les autres cas.

### Oracle

Meilleur SGBD que vous pouvez avoir. Très bon si vous avez une énorme BD.

À éviter si vous ne pouvez pas payer cher pour un SGBD.

### MySQL/MariaDB

Très bon pour des petits systèmes, inclus dans la majorité des hébergements web.

À utiliser pour un site web PHP.

# PL/pgSQL

pgSQL est un langage procédurale, comparable au T-SQL de SQL Server de Microsoft et au PL/SQL d'oracle.

Ce langage permet d'exécuter du code directement dans le sgbd. Ça a l'avantage de limiter les échanges code et BD. Par exemple, avec l'approche standard, le code C# envoie un SELECT au sgbd, fait des vérification puis envoie un INSERT, ce qui fait 2 échanges code/bd. Le pgSQL permet d'envoyer nos variables à la bd et cette dernière fait les vérification et le traitement.

Par contre, il faut faire attention, si notre logique d'affaire se trouve dans la BD, il faut toute la mettre dedans. Pas une partie de logique dans la bd et une autre dans le GUI.

## String

3 manières d'écrire des strings en pgSQL:

```pgsql
SELECT 'I''m a string'; -- version standard
SELECT E'I\'m a string'; -- version postfix
SELECT $$I'm a string$$; -- nouvelle version
```

## Bloc de code

Pour exécuter du code anonyme il faut écrire le mot-clé `do` suivit d'une string contenant notre bloc de code. Un bloc de code débute en déclarant toutes les variables, suivit des instructions qui débutent par `begin` et terminent pas `end`.

Par [exemple](https://www.postgresqltutorial.com/postgresql-plpgsql/dollar-quoted-string-constants/):

```pgsql
do 
$$
declare
   film_count integer;
begin 
   select count(*) into film_count
   from film;
   raise notice 'The number of films: %', film_count;
end;
$$
```

Le `SELECT INTO` permet d'insérer le résultat d'une requête dans une variable.
Le `raise` (choix de debug, log, info, notice, warning et exception) permet d'afficher du code dans la console, comparable au printf d'autres langages.

## Variables

Une variable peut avoir les mêmes types que les colonnes de PostgreSQL. Vous pouvez définir une valeur par défaut avec `:=`, sinon la valeur par défaut est `NULL`.

Exemple:

```pgsql
do $$
declare
  counter integer := 1;
  name varchar(50) := 'toto';
  price numeric(11,2) := 20.5;
  created_at time := now();
  film_title film.title%type;
```

La dernière variable (`film_title`) a une syntaxe spéciale, on va utiliser le type de la colonne title de la table film, utile pour le select into.

On peut aussi faire un select into dans une variable de type `rowtype`, ça donnera quelque chose de similaire à un objet (exemple de [postgres tutorial](https://www.postgresqltutorial.com/postgresql-plpgsql/pl-pgsql-row-types/)):

```pgsql
do $$
declare
   selected_actor actor%rowtype;
begin
   -- select actor with id 10   
   select * 
   from actor
   into selected_actor
   where actor_id = 10;

   -- show the number of actor
   raise notice 'The actor name is % %',
      selected_actor.first_name,
      selected_actor.last_name;
end; $$
```

Le `rowtype` permet seulement de stocker un enregistrement total, si votre SELECT prends seulement quelques colonnes, utilisez un `record`:

```pgsql
declare
   selected_actor record
```

Finalement, vous pouvez déclarer une constante avec la syntaxe suivante:

```pgsql
declare
   taxes constant numeric := 0.14975;
```

## Conditions

Les if-else sont standard:

```pgsql
if cond then
   raise notice 'ok'
end if;

if cond then
   raise notice 'ok';
elsif cond2 then
   raise notice 'ok2';
elsif cond3 then
   raise notice 'ok3';
else
   raise notice 'non';
end if;
```

Pour les switch:

```pgsql
case nombre
   when 0 then
      msg = 'Aucun';
   when 1 then
      msg = 'Oui';
   else
      msg = 'Trop';
end case;

case
   when nombre < 0 then
      msg = 'Erreur';
   when nombre = 0 then
      msg = 'Aucun';
   when nombre > 1 then
      msg = 'Oui';
   end case;
```

## Débogage

Il n'y a pas de débogeur, il faut y aller à la vielle méthode (des raise) ou le assert:

```pgsql
assert film_count > 0, 'Aucun film dans la bd' 
```

Si le assert échoue (false ou null), le code arrête et affiche le message, si le assert réussis (true), le code continue.

## Boucles

pgSQL introduit la `loop`, une boucle infinie qui doit être terminée manuellement, 2 exemples:

```pgsql
loop
   if i <= 0 then
      exit;
   end if
   i := i - 1;
end loop;

loop
   exit when i <= 0;
   i := i - 1;
end loop;
```

Le `while` est standard, c'est une version simplifié du 2e exemple du loop:

```pgsql
while i > 0 loop
   i := i - 1;
end loop;
```

Le `for` est spécial:

```pgsql
for loop_counter in [reverse] from.. to [ by step ] loop
   instructions
end loop;
```

Par défaut un for fait des +1 au compteur à chaque itération, modifiable avec le by step. Le reverse fait des soustraction du step. Exemple:

```pgsql
for i in reverse 10..2 by 2 loop
  raise notice '%', i;
end loop
```

Ça va afficher: 10, 8, 6, 4, 2.

Le for devient intéressant avec une requête au lieu d'un itérateur (exemple de [postgresqltutorial](https://www.postgresqltutorial.com/postgresql-plpgsql/plpgsql-for-loop/)):

```pgsql
do
$$
declare
    f record;
begin
    for f in select title, length 
        from film 
        order by length desc, title
        limit 10 
    loop 
        raise notice '%(% mins)', f.title, f.length;
    end loop;
end;
$$
```

## Fonction

Tout comme pour le `do`, le corp de la fonction doit être une string. Vous devez spécifier le type de retour:

```pgsql
create function get_film_count(len_from int, len_to int)
    returns int
    language plpgsql
as
$$
declare
   film_count integer;
begin
   select count(*) 
   into film_count
   from film
   where length between len_from and len_to;
   
   return film_count;
end;
$$;
```

Pour modifier une fonction, vous devez la réécrire et changer le début de la déclaration par `create or replace function...`. Le langage pourrait être chose (SQL, C, etc.), mais on va toujours utiliser plpgsql.

Pour utiliser une fonction, vous devez utiliser le select:

```pgsql
select get_film_count(40, 90);
```

Les paramètres peuvent être en mode `IN`, `OUT` ou `INOUT`.

 * `IN`, par défaut, on reçoit la valeur demandé
 * `OUT`, la fonction doit mettre une valeur dans ce paramètre
 * `INOUT`, on reçoit une valeur, mais la fonction peut la modifier

En utilisant plusieurs `OUT`, on peut retourner plusieurs valeurs ce qui peut ressembler à un enregistrement.

Plusieurs fonctions peuvent avoir le même nom, mais le nombre de paramètres doivent être différents. Si vous avez des valeurs par défaut ça peut jouer aussi. Par exemple j'ai fct1(aucun param) et fct1(2 params avec valeurs par défaut), pgsql ne saurait pas quel fct1 appeler si je ne met pas de paramètres.

Vous pouvez aussi retourner une table temporaire (exemples de [postgresqltutorial](https://www.postgresqltutorial.com/postgresql-plpgsql/plpgsql-function-returns-a-table/)):

```pgsql
create or replace function get_film (
  p_pattern varchar
) 
	returns table (
		film_title varchar,
		film_release_year int
	) 
	language plpgsql
as $$
begin
	return query 
		select
			title,
			release_year::integer
		from
			film
		where
			title ilike p_pattern;
end;$$
```

Si vous voulez retourner les enregistrements d'une table, vous pouvez utiliser `SETOF`:

```pgsql
RETURNS SETOF users;
```

Pour supprimer une fonction:

```pgsql
DROP FUNCTION [ IF EXISTS ] nom_de_la_fct;
```

## RETURN QUERY/NEXT

En plus du return standard, vous avez accès au `RETURN QUERY` et `RETURN NEXT`. Ces 2 instructions ne retournent rien, mais ajoutent le résultat à la valeur de retour.

Exemple:

```pgsql
create or replace fct_a()
   return table(...)
   language plpgsql
as $$
begin
   return query select * from a; -- ajoute a à la valeur de retour
   return query select * from b; -- ajoute b à la valeur de retour
   return;                       -- return pour vrai
end $$;
```

```pgsql
create or replace fct_b()
   return table(...)
   language plpgsql
as $$
begin
    for f in select title, length 
        from film 
        order by length desc, title
        limit 10 
    loop 
        return next f; -- ajoute f à la valeur de retour
    end loop;
    return;
end $$;
```

Notez que dans l'exemple précédent, le `f` de `return next f` est optionnel, le `return next` va automatique prendre la valeur du "for in".

## Procédure

Appelé procédure stocké par SSMS.

Une procédure est une fonction qui n'a pas de retour et qui ne peut pas avoir de paramètre en mode `OUT` (mais le `INOUT` est permis). Contrairement à la fonction, vous pourrez utiliser les transactions, les commits et les rollbacks. Par défaut une procédure s'exécute dans une transaction.

La syntaxe est très similaire à la fonction:

```pgsql
create [or replace] procedure procedure_name(parameter_list)
    language plpgsql
as $$
declare
    -- variable declaration
begin
    -- stored procedure body
end; $$
```

Pour appeler une fonction il faut faire un `SELECT`, pour la procédure il faut utiliser l'instruction `CALL`.

```pgsql
CALL procedure_name();
```

## SQL

Vous pouvez concaténer du SQL avec les `||` puis l'exécuter avec `EXECUTE`:

```pgsql
EXECUTE 'INSERT INTO ' || ' nom_de_la_table' ...
```

PostgreSQL possède plusieurs caches et optimisateurs, dont un pour le PGSQL et un pour le SQL pure. Vu que l'optimisateur de SQL pure est plus en demande, il est aussi plus efficace. Si vous n'avez pas besoin de variables ou fonctionnalités PGSQL, utilisez une fonction SQL pure. La syntaxe est un peu différente:

```pgsql
create or replace function get_film (
  p_pattern varchar
) 
	returns setof films
	language sql
as $$
   select
      title,
      release_year::integer
   from
      film
   where
      title ilike p_pattern;
$$

create or replace function get_count ()
	returns int
	language sql
as $$
   select count(*) as result -- as result important
   from film;
$$
```

# PostgreSQL

## Limites

Les limites d'un serveur PostgreSQL sont:

| Élément                                 | Limite             |
|-----------------------------------------|--------------------|
| Taille de BD                            | Aucune             |
| Nombre de BD sur un serveur (*cluster*) | 4 294 950 911      |
| Relations par BD                        | 1 431 650 303      |
| Lignes par tables                       | 4 294 967 295      |
| Colonnes par tables                     | Entre 455 et 1600* |
| Longueur d'identifiant                  | 63 octets          |
| Colonnes par index                      | 32                 |

* Limite de 8192 octets par *heap page*. Un enregistrement ne peut donc pas faire plus de 8192 octets. Les champs textes comptent tous pour 18 octets, ça fait un pointeur vers la table TOAST (*the best thing since sliced bread*). Donc avec des int de 4 octets, on a 1600 colonnes et avec des champs texte, il y a seulement 455 colonnes possibles.
Attention! Les colonnes supprimées comptent encore dans le maximum de la *heap page*.

## Identifiants

Tous les identifiants (colonne, table, schéma, etc.) doivent être en notation serpent (*snake case*, nom_de_la_table). Si vous tenez à mettre des majuscules, vous devrez utiliser les guillemets (*double quotes*). Par exemple:

```sql
SELECT * FROM etudiants;
SELECT * FROM "Etudiants";
```

## Normes

Le nom des tables doivent être au pluriel.

Les identifiants peuvent être en français ou en anglais, mais soyez constants. Les interfaces utilisateurs doivent être en français.

Pas de préfixes aux colonnes. C'est une vielle pratique qui n'est plus utilisé en entreprise. En cas de conflit, utilisez le *AS*:

```sql
SELECT users.name as user_name, contacts.name as contact_name...
```

## *Cluster*

C'est une instance de serveur PostgreSQL. Chaque *cluster* vient avec son processus gestionnaire, son port, son dossier de données et ses configurations. Normalement vous n'avez besoin que d'un *cluster* qui existe déjà lors de l'installation de PostgreSQL.

**Attention aux utilisateurs Linux!**
Un *cluster* est lié à un numéro de version. Si vous mettez à jour PostgreSQL vous devrez mettre à jour manuellement votre *cluster*. Je vous suggère de ne pas mettre à jour votre PostgreSQL durant la session et surtout pas durant un TP.

## Schéma

Un *cluster* a plusieurs base de données, une bd a plusieurs schéma, un schéma a plusieurs tables et une table a plusieurs enregistrements.

On est habitué à cette suite, sauf au schéma. Le schéma est tout simplement un "namespace" pour nos tables.

Si dans une bd tp1, on a un schéma admin qui possède une table users et que je veux faire un select, je peux faire la requête suivante:

```sql
SELECT * FROM tp1.admin.users;
```

Nous allons voir plus tard dans quels cas le nom de la bd et du schéma sont optionnels.

# Table temporaire

Une table temporaire est une table créé le temps d'une session seulement (ou d'une transaction).

Cas utile: vous faites des manipulations sur une table, 3-4 requêtes SELECT identiques avec plusieurs JOIN. Au lieu de refaire les JOIN 3-4 fois, on va mettre le résultat du 1er dans une table temporaire pour éviter de se répéter.

On peut créer une table temporaire avec un simple:

```sql
CREATE TEMPORARY TABLE tmp_name(
    ...
) ON COMMIT DROP;
```

Vous pouvez remplacer `TEMPORARY` PAR `TEMP`, le `ON COMMIT DROP` est optionnel, vous pouvez gérer la table manuellement avec un `DROP TABLE tmp_name`. Si votre table temporaire à la même structure qu'une table existante, vous pouvez utiliser le AS:

```sql
CREATE TEMP TABLE tmp_name AS
TABLE users WITH NO DATA;
```

La 2e ligne peut être une requête SELECT complexe, les colonnes vont être créés dynamiquement selon le résultat du SELECT. Le `WITH NO DATA` est optionnel, ça copie seulement la structure, sans ça vous allez copier aussi toutes les données.

# Transaction

Une transaction s'assure que pour plusieurs requêtes, si une ne fonctionne pas on annule tout. Donc dans une transaction, tout fonctionne ou on annule tout.

Exemple:

 * Début de la transaction
 * INSERT 1
 * INSERT 2
 * INSERT 3 -- fail un check, annule les 2 premiers INSERT

On débute une transaction avec une des 3 lignes suivantes:

```pgsql
BEGIN TRANSACTION;
BEGIN WORK;
BEGIN;
```

Une transaction termine lors d'un commit (finalise la transaction) ou un rollback (annule tout). Si une requête provoque une erreur ça fait un ROLLBACK automatique.

```pgsql
COMMIT WORK;
COMMIT TRANSACTION;
COMMIT;

ROLLBACK WORK;
ROLLBACK TRANSACTION;
ROLLBACK;
```

Vous pouvez aussi annuler la partie d'une transaction avec un SAVEPOINT, exemple de https://www.postgresql.org/docs/current/tutorial-transactions.html :

```pgsql
BEGIN;
UPDATE accounts SET balance = balance - 100.00
    WHERE name = 'Alice';
SAVEPOINT my_savepoint;
UPDATE accounts SET balance = balance + 100.00
    WHERE name = 'Bob';
-- oops ... forget that and use Wally's account
ROLLBACK TO my_savepoint;
UPDATE accounts SET balance = balance + 100.00
    WHERE name = 'Wally';
COMMIT;
```

Les procédures appelés par CALL ou DO débutent automatiquement une transaction, elle sera automatiquement commit à la fin de la procédure. Une nouvelle procédure est repartie automatiquement après un commit ou rollback dans ce cas.

## Mutex

PostgreSQL gère automatiquement des Mutex dans une transaction. Il va utiliser un mutex de table (bloque la table) sur un insert, un mutex de ligne (bloque la ligne) sur un update et un mutex de table en lecture (bloque la modification d'une table) sur un select. C'est possible de créer un deadlock sans le vouloir, dans ce cas la fonction va échouer, il faut réessayer...


# Déclencheur

Un déclencheur (trigger) est une procédure ou fonction qui s'exécute automatiquement lors d'un événement.

Exemple:

```pgsql
CREATE OR REPLACE TRIGGER check_update
    BEFORE UPDATE ON accounts
    FOR EACH ROW
    WHEN (OLD.password IS DISTINCT FROM NEW.balance)
    EXECUTE PROCEDURE check_account_update();
```

La prodédure ou fonction doit retourner un trigger:

```pgsql
CREATE OR REPLACE FUNCTION log_last_name_changes()
  RETURNS TRIGGER 
  LANGUAGE PLPGSQL
  AS
$$
BEGIN
	IF NEW.last_name <> OLD.last_name THEN
		 INSERT INTO employee_audits(employee_id,last_name,changed_on)
		 VALUES(OLD.id,OLD.last_name,now());
	END IF;

	RETURN NEW;
END;
$$
```

## CREATE TRIGGER

Un trigger peut être `BEFORE`, `AFTER` ou `INSTEAD OF` sur les actions `INSERT`, `UDPATE`, `DELETE` et `TRUNCATE`. Pour le update on pourrait spécifier une colonne: `UPDATE OF username`. Les actions peuvent être séparés par le mot-clé `OR`.

Le `FOR EACH ROW` signifie que le trigger est déclenché pour chaque enregistrement concerné (par exemple un `DELETE * FROM accounts`). Vous pouvez utiliser `FOR EACH STATEMENT` pour que le trigger soit déclanché une seule fois par requête (mais c'est moins utile).

Le `WHEN` est optionnel, le `OLD` représente l'enregistrement avant update ou delete, le `NEW` représente l'enregistrement après insert ou update.

## Fonction TRIGGER

Le return TRIGGER est nécessaire, vous ne devez avoir aucun paramètre.
Dans la fonction vous avez accès au mot-clé `TG_OP` qui contient l'action actuelle en string (exemple `'DELETE'`).

Vous avez aussi accès aux mots-clés `OLD` et `NEW`:

 * INSERT: OLD est null, NEW contient le nouveau enregistrement
 * DELETE: OLD contient l'enregistrement à supprimer, NEW est null
 * UPDATE: OLD est l'enregistrement avant l'update, NEW est l'enregistrement après l'UPDATE

Lorsqu'on utilise le mode `BEFORE` avec le `FOR EACH ROW`, il faut faire un RETURN. Un `RETURN NULL` annule l'action, un `RAISE WARNING 'message'` aussi. Pour exécuter l'action, le delete doit retourner le OLD, le update et le insert doivent retourner le NEW.

Dans un `BEFORE`, vous pouvez modifier NEW ou OLD afin de changer les valeurs. Voici 2 exemples:

```pgsql
NEW.password := 'bonjour';
SELECT users.password INTO NEW.password
    FROM users
    WHERE users.id = NEW.id;
```

Si votre trigger fait une action (INSERT, UPDATE ou DELETE supplémentaire), faites attention aux boucles infinis!

## Drop Trigger

```pgsql
DROP TRIGGER IF EXISTS nom_du_trigger ON nom_de_la_table
```



# Atelier :

# Atelier 2 - BD, schéma et rôles

## Numéro 1

Remettre les requêtes SQL qui créent les objets suivants:

 * BD atelier2
 * Schéma finances et hr
 * Table finances.produits (id, nom, prix, quantité)
 * Table hr.employe (id, nom, prenom, no_employe, salaire, date_embauche)
 * Ajouter quelques enregistrements dans les 2 tables
 * Rôle (groupe): employe_finances
 * Rôles (utilisateurs): gerant_finances et michel

## Numéro 2

Remettre les requêtes SQL qui donnent les droits minimals pour faire les actions suivantes. Inclure aussi des tests sur les droits.

 * gerant_finances:
   * Propriétaire du schéma finances, il peut créer des tables dans ce schéma
   * Il ne doit pas voir le schéma hr
   * Propriétaire de la table produits

 * employe_finances:
   * Peut faire des SELECT sur la table finances.produits
   * Peut faire un UPDATE seulement sur le champ quantité de la table finances.produits

 * michel:
   * Ne peut rien faire c'est triste
   * L'ajouter dans le groupe employe_finances, il peut maintenant modifier les quantités
   * Il ne doit pouvoir rien faire d'autres que consulter les produits et modifier les quantités

   ----------
```sql
CREATE DATABASE Atelier_2;
CREATE SCHEMA IF NOT EXISTS finances;
CREATE SCHEMA IF NOT EXISTS hr;

CREATE TABLE IF NOT EXISTS finances.produits(
	id SERIAL PRIMARY KEY,
	nom VARCHAR(255) NOT NULL,
	prix FLOAT4 NOT NULL,
	quantite INT NOT NULL
	);


CREATE TABLE IF NOT EXISTS hr.employe(
	id SERIAL PRIMARY KEY,
	nom VARCHAR(255) NOT NULL,
	prenom VARCHAR(255) NOT NULL,
	no_employe INT NOT NULL,
	salaire FLOAT4 NOT NULL,
	date_embauche DATE DEFAULT CURRENT_DATE
);

INSERT INTO finances.produits (nom, prix, quantite) VALUES
('Lave-vaiselle', 699.99, 2),
('Casserole', 9.99, 5),
('Compétent', 123.12, 0);

INSERT INTO hr.employe(nom, prenom, no_employe,salaire, date_embauche) VALUES
('Haut-Drré','Bah Rihz', 1, 0.50, '2023-08-20'),
('Yannick', 'Nault', 2, 15.89, '2022-08-07');

CREATE ROLE employe_finances;
CREATE ROLE gerant_finances WITH LOGIN PASSWORD 'admin';
CREATE ROLE michel WITH LOGIN PASSWORD 'admin';

ALTER SCHEMA finances OWNER TO gerant_finances;
ALTER TABLE finances.produits OWNER TO gerant_finances;
GRANT CREATE ON SCHEMA finances TO gerant_finances;

GRANT SELECT, UPDATE(quantite) ON finances.produits TO employe_finances; 

ALTER GROUP employe_finances ADD USER michel;




--TEST

--Michel :
CREATE TABLE test(
id SERIAL FOREIGN KEY,
nom TEXT);

SELECT * FROM finances.produits;
UPDATE finances.produits SET (quantite = 2);
UPDATE finances.produits SET (nom = 'test'); 


--gerant_finance

CREATE TABLE test(
id SERIAL FOREIGN KEY,
nom TEXT);

SELECT * FROM finances.produits;
UPDATE finances.produits SET (quantite = 2);
UPDATE finances.produits SET (nom = 'test'); 

--employe_finance

CREATE TABLE test(
id SERIAL FOREIGN KEY,
nom TEXT);

SELECT * FROM hr.employe;
CREATE TABLE test(
id SERIAL FOREIGN KEY,
nom TEXT);

SELECT * FROM finances.produits;
UPDATE finances.produits SET (nom = 'test'); 
```
-----------

# Atelier 3 - Héritage et partitionnage

## Numéro 1

Créez les tables suivantes:

 * produits (id, nom, prix)
 * disque_dur (hérite de produit, ssd, vitesse)
 * carte_graphique (hérite de produit, memoire)

Faites des INSERT sur les 3 tables et des select pour chaque table.

Faites un SELECT de produits qui spécifie aussi la catégorie de produit et les spécificités.

## Numéro 2

Créez la table suivante:

 * utilisateurs (id, nom_utilisateur, mot_de_passe, date_de_creation, etat)

L'état peut être (validez avec un CHECK): 'A_CONFIRME', 'ACTIF', 'BANNIS'.

Créez une partition par état, puis insérez au moins 2 enregistrements par partition.

Avec un UPDATE, banissez un utilisateur actif puis vérifier que le compte a bien bougé de table.

## Numéro 3

Avec vos tables et enregistrements du numéro 1:

 * Faire la moyenne de prix des produits
 * Sortir la moyenne de prix des produits par catégories (disque_dur et carte_graphique) en utilisant seulement la table `produits`
 * Idem que le point précédent, mais sans utiliser la table `produits`


 ------------
 ```sql

 CREATE TABLE IF NOT EXISTS produits  (
	id SERIAL PRIMARY KEY,
	nom VARCHAR(255) NOT NULL,
	prix FLOAT4 NOT NULL
);
SELECT * FROM produits;

CREATE TABLE IF NOT EXISTS disque_dur (
	ssd BOOL NOT NULL,
	vitesse INT NOT NULL
) INHERITS (produits);

INSERT INTO disque_dur(nom, prix, ssd, vitesse) VALUES
('Ordi du cégep', 12345.77, 'true', 777),
('Legion 16po', 2344.99, 'true', 666),
('Dell XPS 15', 2780, 'true', 777),
('PC', 123, 'false', 321);

SELECT * FROM disque_dur;


CREATE TABLE IF NOT EXISTS carte_graphique (
	memoire INT NOT NULL
) INHERITS (produits);

INSERT INTO carte_graphique(nom, prix, memoire) VALUES
('Esport', 666.66, 32),
('Legion 12po', 2344.99, 16),
('Dell XPS 13', 2780, 32),
('PC-2', 123, 12);

SELECT * FROM carte_graphique;

SELECT 'disque_dur' AS categ, *, NULL as memoire FROM disque_dur
	UNION
SELECT 'carte_graphique' AS categ, id, nom, prix, NULL, NULL, memoire FROM carte_graphique
	UNION
SELECT 'produits' AS categ, *, NULL, NULL, NULL FROM produits;


-- Numéro 2

CREATE TABLE IF NOT EXISTS utilisateurs(
	id SERIAL,
	nom_utilisateur VARCHAR(255) NOT NULL,
	mot_de_passe TEXT NOT NULL,
	date_de_creation DATE DEFAULT CURRENT_DATE,
	etat VARCHAR(255) NOT NULL CHECK ((etat = 'A_CONFIRME') OR (etat = 'ACTIF') OR (etat = 'BANNIS')),
	PRIMARY KEY (id, etat)
) PARTITION BY LIST (etat);

CREATE TABLE utilisateurs_etat_A_CONFIRME PARTITION OF utilisateurs
	FOR VALUES IN ('A_CONFIRME');
CREATE TABLE utilisateurs_etat_ACTIF PARTITION OF utilisateurs
	FOR VALUES IN ('ACTIF');
CREATE TABLE utilisateurs_etat_BANNIS PARTITION OF utilisateurs
	FOR VALUES IN ('BANNIS');
	
INSERT INTO utilisateurs(nom_utilisateur, mot_de_passe, date_de_creation,etat) VALUES
('admin', 'admin', '2023-08-29', 'ACTIF'),
('LuluLePerdu', '123', '2023-08-21', 'ACTIF'),
('Yannick', 'chat', '2023-09-01', 'A_CONFIRME'),
('Mathis', 'Citron', '2023-08-02', 'A_CONFIRME'),
('Audrey', 'Costco', '2023-07-19', 'BANNIS'),
('Michel', 'linux', '2023-08-29', 'BANNIS');


SELECT * FROM utilisateurs_etat_A_CONFIRME;
SELECT * FROM utilisateurs_etat_ACTIF;
SELECT * FROM utilisateurs_etat_BANNIS;

UPDATE utilisateurs SET etat = 'BANNIS' WHERE nom_utilisateur = 'LuluLePerdu';
SELECT * FROM utilisateurs_etat_ACTIF;
SELECT * FROM utilisateurs_etat_BANNIS;


-- Numéro 3

SELECT AVG(prix) FROM produits;
SELECT p.relname, AVG(c.prix) FROM produits c, pg_class p WHERE c.tableoid = p.oid GROUP BY (p.relname);
SELECT 'carte_graphique' AS categ, AVG(prix) FROM carte_graphique GROUP BY categ UNION SELECT 'disque_dur' AS categ, AVG(prix) FROM disque_dur GROUP BY categ;

```

---------------

# Atelier 4 - PGSQL

Pour chaque numéro, remettre le code qui crée la procédure ou fonction PGSQL pour le problème donné, ainsi que des codes de tests.

## Numéro 1

Fonction/procédure "initialiser".

Créer les tables:

 * produits (id, nom, quantite, prix)
 * transactions (id, quantite, produit_id, moment)
 * ventes (hérite de transactions)
 * achats (hérite de transactions)
 * ajustements (hérite de transactions)

Pour une transaction, le moment doit être "maintenant" par défaut.

Ajoutez aussi quelques produits.

## Numéro 2

Fonction/procédure "ajuster_quantite".

Prend un id de produit et une quantité en paramètre. Ajoute la transaction à la table ajustements et met à jour l'enregistrement correspondant de la table produit.

Si le produit n'existe pas, affichez un message personnalisé dans la console (le produit avec l'id 42 n'existe pas).

## Numéro 3

Fonction/procédure "ajouter_transaction".

Prend un id de produit et une quantité en paramètre. Si la quantité est négative, l'ajouter en valeur positive aux achats. Si la quantité est positive, validez qu'il reste assez de produits et si oui, l'ajouter aux ventes.

Si le produit n'existe pas, affichez un message personnalisé dans la console (le produit avec l'id 42 n'existe pas).

Écrire dans la console la quantité restante et mettre à jour l'enregistrement correspondant de la table produit.

## Numéro 4

Fonction/procédure "valeur_inventaire".

Retourne la valeure totale de l'inventaire (somme de quantité * prix de chaque enregistrement). Faites une version en SQL pur et une 2e version en utilisant seulement un SQL simple `SELECT * FROM produits` avec du pgsql.

## Numéro 5

Fonction/procédure "transactions".

Prend un id de produit en paramètre. Retourne toutes les transactions pour le produit demandé, affiche un message si le produit n'existe pas. Le résultat doit être trié en ordre de `moment`, les quantités d'achats doivent être négatifs et les quantités d'ajustements doivent être préfixés de '='.

Faire une deuxième fonction du même nom qui fonctionne de manière similaire, mais prend en paramètre une date de début et de fin. Retourne toutes les transactions pour tous les produits entre les dates données.

Faire une troisième fonction du même nom qui combine les 2 précédentes: prend en paramètre un id de produit, une date de début et une date de fin.

```sql
--#1
CREATE OR REPLACE PROCEDURE initialiser()
	LANGUAGE PLPGSQL
AS $$
BEGIN
	CREATE TABLE IF NOT EXISTS produits(
		id SERIAL PRIMARY KEY,
		nom TEXT,
		quantite INT,
		prix FLOAT4
	);
	
	CREATE TABLE IF NOT EXISTS transactions(
		id SERIAL PRIMARY KEY,
		quantite INT,
		produit_id INT,
		moment TIMESTAMP DEFAULT now(),
		FOREIGN KEY (produit_id) REFERENCES produits(id)
	);
	
	CREATE TABLE IF NOT EXISTS ventes(
	)INHERITS (transactions);
	
	CREATE TABLE IF NOT EXISTS achats(
	)INHERITS (transactions);
	
	CREATE TABLE IF NOT EXISTS ajustements(
	)INHERITS (transactions);
	
	INSERT INTO produits (nom, quantite, prix) VALUES
		('Cellulaire', 2, 299.99),
		('Laptop', 4, 499.99),
		('Biscuit de la cafet', 30, 15.99),
		('Ecouteur', 2, 20.99);
END$$;

CALL initialiser();
--#2

CREATE OR REPLACE PROCEDURE ajuster_quantite(_id INT, _quantite INT)
	LANGUAGE PLPGSQL
AS $$
BEGIN
	IF (SELECT _id FROM produits WHERE _id = id) THEN
		UPDATE produits SET quantite = _quantite WHERE id = _id;
		INSERT INTO ajustements(quantite, produit_id) VALUES
			(_quantite, _id);
	ELSE
		RAISE NOTICE 'Le produit avec l''id % n''existe pas', _id;
	END IF;
END$$;

CALL ajuster_quantite(1,2);

--#3

CREATE OR REPLACE PROCEDURE ajouter_transaction(_id INT, _quantite INT)
	LANGUAGE PLPGSQL
AS $$
BEGIN
	IF _quantite < 0 THEN
	_quantite := _quantite * -1
		INSERT INTO achats (quantite, produit_id) VALUES (_quantite, _id);
		INSERT INTO ajustements(quantite, produit_id) VALUES
			(_quantite, _id);
		RAISE NOTICE 'La qauntité restante est de %', quantite;
	ELSE
		RAISE NOTICE 'Le produit avec l''id % n''existe pas', _id;
	END IF;
END$$;

CALL ajouter_transaction(1,4);
--#4

CREATE OR REPLACE PROCEDURE valeur_inventaire()
	LANGUAGE PLPGSQL
AS $$
DECLARE
item RECORD
total FLOAT4
BEGIN
	FOR item IN SELECT quantite, prix FROM produits
	LOOP 
	total := quantite * prix
	END FOR
	RAISE NOTICE 'La valeur est de %', total;
END$$;

--V2
SELECT SUM (quantite * prix) AS valeur FROM produits;

--#5

CREATE OR REPLACE FUNCTION transactions(product_id INT)
LANGUAGE plpgsql
RETURNS TABLE (
    moment TIMESTAMP,
    description TEXT
)
AS $$
BEGIN
    RETURN QUERY
    SELECT moment,
           CASE
               WHEN type = 'achat' THEN '-' || quantity::TEXT
               WHEN type = 'ajustement' THEN '=' || quantity::TEXT
           END AS description
    FROM transactions
    WHERE product_id = product_id
    ORDER BY moment;
    
    IF NOT FOUND THEN
        RAISE EXCEPTION 'Le produit avec l''ID % n''existe pas', product_id;
    END IF;
END;
$$

CREATE OR REPLACE FUNCTION transactions(start_date DATE, end_date DATE)
LANGUAGE plpgsql
RETURNS TABLE (
    product_id INT,
    moment TIMESTAMP,
    description TEXT
)
AS $$
BEGIN
    RETURN QUERY
    SELECT product_id,
           moment,
           CASE
               WHEN type = 'achat' THEN '-' || quantity::TEXT
               WHEN type = 'ajustement' THEN '=' || quantity::TEXT
           END AS description
    FROM transactions
    WHERE moment BETWEEN start_date AND end_date
    ORDER BY product_id, moment;
END;
$$;

CREATE OR REPLACE FUNCTION transactions(product_id INT, start_date DATE, end_date DATE)
LANGUAGE plpgsql
RETURNS TABLE (
    moment TIMESTAMP,
    description TEXT
)
AS $$
BEGIN
    RETURN QUERY
    SELECT moment,
           CASE
               WHEN type = 'achat' THEN '-' || quantity::TEXT
               WHEN type = 'ajustement' THEN '=' || quantity::TEXT
           END AS description
    FROM transactions
    WHERE product_id = product_id
    AND moment BETWEEN start_date AND end_date
    ORDER BY moment;
    
    IF NOT FOUND THEN
        RAISE EXCEPTION 'Le produit avec l''ID % n''existe pas', product_id;
    END IF;
END;
$$;
```



--------------
# Atelier 5 - Déclencheur

## Numéro 1 - a

Créer une procédure qui crée la table suivante (aucune valeur par défaut ni check):

 * produits
    * id, nom, prix, quantite, creation, mise_a_jour
    * creation et mise_a_jour sont des timestamp (date avec heure)

## Numéro 1 - b

Faire un Trigger sur le insert qui:

 * Vérifie que le prix et la quantite sont positifs ou zéro
 * Vérifie que le nom est unique
 * Met la création à maintenant et la mise à jour à null

Si une des 2 première vérification échoue, annuler le insert avec un message d'erreur clair.

Faire des inserts pour tester les vérifications.

## Numéro 1 - c

Faire un trigger sur le update qui:

 * Met la mise à jour à maintenant

Faire au moins 1 test.

## Numéro 2 - a

Modifiez la table du numéro 1 pour ajouter la colonne `supprime` qui est un timestamp, par défaut à NULL.

Faire un trigger sur le delete et met supprime à maintenant au lieu de supprimer pour vrai.

## Numéro 2 - b

Assurez-vous d'avoir des produits supprimés et des produits "actifs".

Créez les 3 fonctions suivantes:

 * produits_actifs: retourne tous les produits non-supprimés
 * produits_supprime: retourne tous les produits supprimés
 * produits_tous: retourne tous les produits (feature tellement inutile)

Testez les 3 fonctions précédentes.

P.S. On appel ce comportement un "Soft Delete", ça permet de supprimer un enregistrement de l'interface sans vraiment perdre les données. Habituellement la colonne de "Soft delete" va être la dernière de la table.

## Numéro 3

Reprendre les tables de l'atelier 4 (produits, transactions, ventes, achats et ajustements) avec quelques produits.

Avec des triggers, lorsque j'insère une vente, un achat ou un ajustement, mettre à jour automatiquement la quantité correspondante dans les produits.

Faire des inserts et des select afin de tester le tout.

## Numéro 4

**Numéro à réviser pour 2024**

Créez la table factures suivante partitionné sur la date:

 * id, no_facture, nom_fournisseur, montant, date
 * il y aura 1 partition par mois

Créez au moins 1 partition manuellement avec au moins 1 enregistrement.

Créez maintenant un trigger sur le insert qui crée des nouvelles partitions à la volée.

Insérez quelques factures afin de créer dynamiquement les tables.
```sql
-- No 1 A

CREATE OR REPLACE PROCEDURE initialiser()
	LANGUAGE PLPGSQL
AS $$
BEGIN
	CREATE TABLE IF NOT EXISTS produits (
	id SERIAL PRIMARY KEY,
	nom VARCHAR(255),
	prix FLOAT4,
	quantite INT,
	creation TIMESTAMP,
	mise_a_jour TIMESTAMP
	);
END; $$

CALL initialiser();

-- No 1 B

CREATE OR REPLACE TRIGGER insert_produits
    BEFORE INSERT ON produits
    FOR EACH ROW
    EXECUTE FUNCTION insert_produits();
	
	
CREATE OR REPLACE FUNCTION insert_produits()
	RETURNS TRIGGER 
	LANGUAGE PLPGSQL
AS $$
BEGIN
	IF NEW.prix >= 0 AND NEW.quantite >= 0 AND (SELECT id FROM produits WHERE nom = NEW.nom) IS NULL THEN
		NEW.creation = NOW()::TIMESTAMP;
		NEW.mise_a_jour = NULL;
		RETURN NEW;
	END IF;
	RAISE NOTICE 'Rentrez des données valide'
	RETURN NULL;
END
$$;

INSERT INTO produits (nom, prix, quantite) VALUES 
	('Lulu', 100, 50);
	
INSERT INTO produits (nom, prix, quantite) VALUES 
	('Audrey', 2, -3);
	
INSERT INTO produits (nom, prix, quantite) VALUES 
	('Yannick', -4, 10);
	
INSERT INTO produits (nom, prix, quantite) VALUES 
	('Nathan', 4, 10);
	
INSERT INTO produits (nom, prix, quantite) VALUES 
	('Mathis', 11, 15);

INSERT INTO produits (nom, prix, quantite) VALUES 
	('Jonathan', 29, 4);

INSERT INTO produits (nom, prix, quantite) VALUES 
	('Philippe', 6, 150);
	
SELECT * FROM produits;

TRUNCATE produits

-- No 1 C

CREATE OR REPLACE TRIGGER update_produits
    BEFORE UPDATE ON produits
    FOR EACH ROW
    EXECUTE FUNCTION update_produits();
	
	
CREATE OR REPLACE FUNCTION update_produits()
	RETURNS TRIGGER 
	LANGUAGE PLPGSQL
AS $$
BEGIN	
	NEW.mise_a_jour = NOW()::TIMESTAMP;
	RETURN NEW;
END
$$;

UPDATE produits SET quantite = 51 WHERE id = 9;

-- No 2 A

ALTER TABLE produits ADD supprime TIMESTAMP DEFAULT NULL;

CREATE OR REPLACE TRIGGER delete_produits
    BEFORE DELETE ON produits
    FOR EACH ROW
    EXECUTE FUNCTION delete_produits();
	
	
CREATE OR REPLACE FUNCTION delete_produits()
	RETURNS TRIGGER 
	LANGUAGE PLPGSQL
AS $$
BEGIN
	UPDATE produits SET supprime = NOW()::TIMESTAMP WHERE id = OLD.id;
	RETURN NULL;
END
$$;

DELETE FROM produits WHERE nom = 'Nathan';
DELETE FROM produits WHERE nom = 'Mathis';

-- No 2 B

CREATE OR REPLACE FUNCTION produits_actifs()
	RETURNS SETOF produits
	LANGUAGE PLPGSQL
AS $$
BEGIN
	RETURN QUERY SELECT * FROM produits WHERE supprime IS NULL;
END
$$;


CREATE OR REPLACE FUNCTION produits_supprime()
	RETURNS SETOF produits
	LANGUAGE PLPGSQL
AS $$
BEGIN
	RETURN QUERY SELECT * FROM produits WHERE supprime IS NOT NULL;
END
$$;


CREATE OR REPLACE FUNCTION produits_tous()
	RETURNS SETOF produits
	LANGUAGE PLPGSQL
AS $$
BEGIN
	RETURN QUERY SELECT * FROM produits;
END
$$;


SELECT * FROM produits_actifs();
SELECT * FROM produits_supprime();
SELECT * FROM produits_tous();

-- No 3

CREATE OR REPLACE PROCEDURE initialiser()
	LANGUAGE PLPGSQL
AS $$
BEGIN
	CREATE TABLE IF NOT EXISTS produits2 (
	id SERIAL PRIMARY KEY,
	nom VARCHAR(255),
	quantite INT,
	prix FLOAT4
	);
	
	CREATE TABLE IF NOT EXISTS transactions (
	id SERIAL PRIMARY KEY,
	quantite INT,
	produit_id INT,
	moment TIMESTAMP DEFAULT now(),
	FOREIGN KEY (produit_id) REFERENCES produits2 (id)
	);
	
	CREATE TABLE IF NOT EXISTS ventes (
	) INHERITS (transactions);
	
	CREATE TABLE IF NOT EXISTS achats (
	) INHERITS (transactions);
	
	CREATE TABLE IF NOT EXISTS ajustements (
	) INHERITS (transactions);
	
	IF (SELECT COUNT(*) FROM produits) = 0 THEN
		INSERT INTO produits (nom, quantite, prix) VALUES
			('Ordi normal', 100, 500),
			('Switch reseau', 50, 150),
			('routeur', 50, 200),
			('Iphone', 10000, 999);
	END IF;
END; $$

CALL initialiser();

SELECT * FROM produits2;
SELECT * FROM transactions;
SELECT * FROM achats;
SELECT * FROM ventes;
SELECT * FROM ajustements;


CREATE OR REPLACE TRIGGER insert_ventes
    BEFORE INSERT ON ventes
    FOR EACH ROW
    EXECUTE FUNCTION insert_ventes();
	
	
CREATE OR REPLACE FUNCTION insert_ventes()
	RETURNS TRIGGER 
	LANGUAGE PLPGSQL
AS $$
BEGIN
	UPDATE produits2 SET quantite = quantite - NEW.quantite WHERE id = NEW.produit_id;
	RETURN NEW;
END
$$;

INSERT INTO produits2 (nom, quantite, prix) VALUES ('test', 2, 10);
INSERT INTO ventes (quantite, produit_id) VALUES (1, 1);



CREATE OR REPLACE TRIGGER insert_achats
    BEFORE INSERT ON achats
    FOR EACH ROW
    EXECUTE FUNCTION insert_achats();
	
	
CREATE OR REPLACE FUNCTION insert_achats()
	RETURNS TRIGGER 
	LANGUAGE PLPGSQL
AS $$
BEGIN
	UPDATE produits2 SET quantite = quantite + NEW.quantite WHERE id = NEW.produit_id;
	RETURN NEW;
END
$$;

INSERT INTO achats (quantite, produit_id) VALUES (1, 1);



CREATE OR REPLACE TRIGGER insert_ajustements
    BEFORE INSERT ON ajustements
    FOR EACH ROW
    EXECUTE FUNCTION insert_ajustements();
	
	
CREATE OR REPLACE FUNCTION insert_ajustements()
	RETURNS TRIGGER 
	LANGUAGE PLPGSQL
AS $$
BEGIN
	UPDATE produits2 SET quantite = NEW.quantite WHERE id = NEW.produit_id;
	RETURN NEW;
END
$$;

INSERT INTO ajustements (quantite, produit_id) VALUES (5, 1);
```

-----

# Atelier 6 - Curseurs et transactions

## Numéro 1 - a

Créez la table suivante:

 * compte_banque: id, no_compte, nom, balance
 * balance est un float qui doit être positif (check)

Insérez 2 comptes, avec des balances de 20$ et 100$ (par exemple).

## Numéro 1 - b

Créez une procédure qui:

 * Prend en paramètre 2 ids de compte_banque et le montant
 * Faites 2 updates, ajoutez l'argent au vendeur puis le retirer de l'acheteur (dans cet ordre)
 * Ne faites aucune vérification, fiez-vous sur la transaction automatique

 * Essayez de mettre un COMMIT temporairement entre les 2 updates, remarquez la différences "sans transaction"

## Numéro 1 - c

Ajoutez une table transaction: id, compte_in, compte_out, moment, etat.
etat va contenir "Réussite" ou "Échec".

Retirez le check de la colonne balance.

Créez une procédure qui:

 * Prend en paramètre 2 ids de compte_banque et un montant
 * Ajoute une transaction avec un état vide
 * Ajoutez l'argent au vendeur
 * Vérifie la balance de l'acheteur, si c'est inssufisant faire un rollback
 * Finalement mettre à jour la transaction pour mettre l'état à "Réussite" ou insérer un "Échec"
 * Testez avec une réussite et un échec

Indice: vous pouvez avoir l'id inséré avec un CURRVAL (var_id étant une variable, table_id_seq le nom de la séquence du serial):

```pgsql
var_id := CURRVAL('table_id_seq');
```

## Numéro 2 - a

Créez la table suivante:

 * stats: id, force, dexterite, endurance, intelligence, sagesse, charisme, niveau

Insérez 50 enregistrements, les colonnes de force à charisme (inclusivement) avec une valeur aléatoire entre 1 et 20.

## Numéro 2 - b

Avec un curseur explicite, parcourir chaque enregistrement. Faites la somme de toutes les stats (de force à charisme).

Selon la somme faire:

 * Moins de 50: Supprimer l'enregistrement
 * 50 à 59: Mise à jour pour le niveau 1
 * 60 à 69: Niveau 2
 * 70 à 79: Niveau 3
 * 80 et plus: Niveau 4

## Numéro 2 - c

Créez une vue qui représente seulement les stats niveau 4.
```sql
--#1 -a
CREATE TABLE IF NOT EXISTS compte_banque (
	id INT GENERATED ALWAYS AS IDENTITY,
	no_compte INTEGER,
	nom TEXT,
	balance FLOAT4
);

INSERT INTO compte_banque(no_compte, nom, balance) VALUES 
	(111, 'premier_compte', 20),
	(222, 'second_compte', 100)


--#1 - b

CREATE OR REPLACE PROCEDURE virement(id_vendeur INT, id_acheteur INT, montant FLOAT4)
	LANGUAGE PLPGSQL
AS $$
BEGIN
	UPDATE compte_banque SET balance = balance + montant WHERE id = id_vendeur;
	COMMIT;
	UPDATE compte_banque SET balance = balance - montant WHERE id = id_acheteur;
END$$;

--#1 - c

CREATE TABLE IF NOT EXISTS transactions(
	id INT GENERATED ALWAYS AS IDENTITY,
	compte_in INT,
	compte_out INT,
	moment TIMESTAMP DEFAULT NOW(),
	etat VARCHAR(10)
);

TRUNCATE transactions

INSERT INTO transactions(compte_in, compte_out, moment) VALUES
(
1,2,NOW()
);

CALL effectuer_transaction(1,2,20)
SELECT * FROM transactions
SELECT * FROM compte_banque

CREATE OR REPLACE PROCEDURE effectuer_transaction(
    IN p_compte_in INT,
    IN p_compte_out INT,
    IN p_montant FLOAT4
)
LANGUAGE PLPGSQL
AS $$
DECLARE
    v_balance_acheteur FLOAT4;
BEGIN
    
    INSERT INTO transactions (compte_in, compte_out, moment, etat)
    VALUES (p_compte_in, p_compte_out, NOW(), '');

    UPDATE compte_banque
    SET balance = balance + p_montant
    WHERE id = p_compte_out;

    SELECT balance INTO v_balance_acheteur
    FROM compte_banque
    WHERE id = p_compte_in;

    IF v_balance_acheteur < p_montant THEN
        UPDATE transactions
        SET etat = 'Échec'
        WHERE compte_in = p_compte_in AND compte_out = p_compte_out;
        ROLLBACK;
    ELSE
        UPDATE transactions
        SET etat = 'Réussite'
        WHERE compte_in = p_compte_in AND compte_out = p_compte_out;
        COMMIT;
    END IF;
END;
$$;



--#2 - a

CREATE TABLE IF NOT EXISTS stats (
	id INT GENERATED ALWAYS AS IDENTITY,
	forces INT,
	dexterite INT,
	endurance INT,
	intelligence INT,
	sagesse INT,
	charisme INT,
	niveau INT
);


DO $$
BEGIN
   FOR iterator in 1..50 LOOP
	   INSERT INTO stats(forces, dexterite, endurance, intelligence, sagesse, charisme) VALUES 
		(trunc(random()*20 + 1),
		trunc(random()*20 + 1),
		trunc(random()*20 + 1),
		trunc(random()*20 + 1),
		trunc(random()*20 + 1),
		trunc(random()*20 + 1));
   END LOOP;
END
$$; 

SELECT * FROM stats;

--#2 - b

DO $$
DECLARE
	cur CURSOR FOR SELECT * FROM stats;
BEGIN
	FOR stats IN cur LOOP
		IF (stats.forces + stats.dexterite + stats.endurance + stats.intelligence + stats.sagesse + stats.charisme < 50) THEN
			DELETE FROM stats WHERE CURRENT OF cur;
		ELSIF ((stats.forces + stats.dexterite + stats.endurance + stats.intelligence + stats.sagesse + stats.charisme) < 60) THEN
			UPDATE stats SET niveau = 1 WHERE CURRENT OF cur ;
		ELSIF((stats.forces + stats.dexterite + stats.endurance + stats.intelligence + stats.sagesse + stats.charisme) < 70) THEN
			UPDATE stats SET niveau = 2 WHERE CURRENT OF cur;
		ELSIF((stats.forces + stats.dexterite + stats.endurance + stats.intelligence + stats.sagesse + stats.charisme) < 80) THEN
			UPDATE stats SET niveau = 3 WHERE CURRENT OF cur;
		ELSIF ((stats.forces + stats.dexterite + stats.endurance + stats.intelligence + stats.sagesse + stats.charisme) >= 80) THEN
			UPDATE stats SET niveau = 4 WHERE CURRENT OF cur;
		END IF;
	END LOOP;
END
$$;

SELECT * FROM stats

--#2 - c

CREATE OR REPLACE VIEW niveau_4 AS SELECT * FROM stats WHERE stats.niveau = 4;
SELECT * FROM niveau_4;
```

---
# Numéro 7 - Analyse

## Question 0

Créer les tables suivantes:

 * cours (id, nom, sigle)
 * prealables (cours_id, prealable_id)
 * etudiants (id, da, cohorte) -- la cohorte est l'année de graduation
 * notes (cours_id, etudiant_id, note)

Insérez les cours et préalables suivants:

```sql
INSERT INTO cours (id, nom, sigle) VALUES
    (1, 'Système 1', '420-M13'),
    (2, 'Programmation 1', '420-C17'),
    (3, 'Rédaction', '420-N15'),
    (4, 'Math info', '201-T15'),
    (5, 'Système 2', '420-M24'),
    (6, 'Programmation 2', '420-C27'),
    (7, 'Web 1', '420-N26'),
    (8, 'Interagir dans un contexte professionnel', '401-T24'),
    (9, 'Réseau 1', '420-M34'),
    (10, 'Programmation 3', '420-C35'),
    (11, 'BD1', '420-B35'),
    (12, 'Modélisation', '420-A33'),
    (13, 'Soutien technique', '420-M43'),
    (14, 'Conception', '420-C46'),
    (15, 'Web 2', '420-N46'),
    (16, 'Math vectoriel', '201-T45'),
    (17, 'Réseau 2', '420-M54'),
    (18, 'Mobile', '420-C56'),
    (19, 'BD2', '420-B56'),
    (20, 'Jeux vidéo', '420-G56'),
    (21, 'Projet mobile', '420-C64'),
    (22, 'Projet web', '420-N64'),
    (23, 'Projet d''intégration 1', '420-P64'),
    (24, 'Stage', '420-P62');

INSERT INTO prealables (cours_id, prealable_id) VALUES
    (5, 1),
    (6, 2),
    (7, 2),
    (9, 5),
    (10, 6),
    (11, 12), --corequis
    (13, 9),
    (13, 8),
    (14, 10),
    (14, 12),
    (17, 9),
    (17, 14),
    (18, 14),
    (19, 11),
    (20, 16),
    (20, 14),
    (21, 18),
    (21, 19),
    (22, 15),
    (23, 21),
    (23, 22),
    (24, 21),
    (24, 22),
    (24, 23);
```

Créez aléatoirement, entre 20 et 60 étudiants par cohorte pour 10 cohortes.

Pour chaque étudiant, mettre 1 note entre 60 et 100 pour chaque cours.

## Question 1 - a

Faire une seule requête qui, pour tous les étudiants, donne leur percentile.

## Question 1 - b

Faire une seule requête qui, pour tous les étudiants, donne leur rang dans leur cohorte (donc savoir qui est le meilleur de chaque cohorte, le 2e meilleur, etc.).

## Question 2 - a

Faire une seule requête SQL qui donne:
Pour chaque cohorte:

 * Son année
 * Sa moyenne
 * La différence entre cette année et la précédente

## Question 2 - b

Faire la même chose, mais utilisez une table temporaire qui se souvient du "etudiants JOIN notes".

## Question 2 - c

Avec le Explain, trouvez laquelle des 2 solutions précédentes est la plus efficace.

## Question 3 - a

Sans utiliser de SQL récursif (utilisez du PGSQL), donnez le cours "Stage" avec tous ses préalables.

## Question 3 - b

Refaire la même chose mais avec une seule requête SQL (utilisez RECURSIVE).

## Question 3 - c

Avec le EXPLAIN, quelle est la solution la plus efficace?

```sql
-- Numéro 0

CREATE TABLE cours (
	id SERIAL PRIMARY KEY,
	nom TEXT NOT NULL,
	sigle VARCHAR(7)
);

CREATE TABLE prealables (
	cours_id INTEGER REFERENCES cours(id),
	prealable_id INTEGER REFERENCES cours(id),
	PRIMARY KEY(cours_id,  prealable_id)
);

CREATE TABLE etudiants (
    id SERIAL PRIMARY KEY,
    da VARCHAR(7) NOT NULL,
    cohorte INTEGER NOT NULL
);

CREATE TABLE notes (
    cours_id INTEGER REFERENCES cours(id),
    etudiant_id INTEGER REFERENCES etudiants(id),
    note DECIMAL(5,2),
	PRIMARY KEY(cours_id, etudiant_id)
);

INSERT INTO cours (id, nom, sigle) VALUES
    (1, 'Système 1', '420-M13'),
    (2, 'Programmation 1', '420-C17'),
    (3, 'Rédaction', '420-N15'),
    (4, 'Math info', '201-T15'),
    (5, 'Système 2', '420-M24'),
    (6, 'Programmation 2', '420-C27'),
    (7, 'Web 1', '420-N26'),
    (8, 'Interagir dans un contexte professionnel', '401-T24'),
    (9, 'Réseau 1', '420-M34'),
    (10, 'Programmation 3', '420-C35'),
    (11, 'BD1', '420-B35'),
    (12, 'Modélisation', '420-A33'),
    (13, 'Soutien technique', '420-M43'),
    (14, 'Conception', '420-C46'),
    (15, 'Web 2', '420-N46'),
    (16, 'Math vectoriel', '201-T45'),
    (17, 'Réseau 2', '420-M54'),
    (18, 'Mobile', '420-C56'),
    (19, 'BD2', '420-B56'),
    (20, 'Jeux vidéo', '420-G56'),
    (21, 'Projet mobile', '420-C64'),
    (22, 'Projet web', '420-N64'),
    (23, 'Projet d''intégration 1', '420-P64'),
    (24, 'Stage', '420-P62');

INSERT INTO prealables (cours_id, prealable_id) VALUES
    (5, 1),
    (6, 2),
    (7, 2),
    (9, 5),
    (10, 6),
    (11, 12), --corequis
    (13, 9),
    (13, 8),
    (14, 10),
    (14, 12),
    (17, 9),
    (17, 14),
    (18, 14),
    (19, 11),
    (20, 16),
    (20, 14),
    (21, 18),
    (21, 19),
    (22, 15),
    (23, 21),
    (23, 22),
    (24, 21),
    (24, 22),
    (24, 23);
	
	
DO
$$
DECLARE
    etudiant_total INTEGER;
    quantite_cohort INTEGER := 10;
BEGIN
    WHILE quantite_cohort > 0 LOOP
		quantite_cohort := quantite_cohort - 1;
        etudiant_total := (RANDOM() * 40 + 20);

        FOR i IN 1..etudiant_total LOOP
            INSERT INTO etudiants(da, cohorte)
            VALUES (CAST(CAST((RANDOM() * 7999999 + 1000000) AS INTEGER) AS TEXT),
					EXTRACT(YEAR FROM CURRENT_DATE) - quantite_cohort);
        END LOOP;
    END LOOP;
END
$$;

DO
$$
DECLARE
	curs_etudiants CURSOR FOR SELECT * FROM etudiants;
	etudiant_record RECORD;
	curs_cours CURSOR FOR SELECT * FROM cours;
	cours_record RECORD;
BEGIN
    OPEN curs_etudiants;

    LOOP
        FETCH curs_etudiants INTO etudiant_record;
        EXIT WHEN NOT FOUND;
		    OPEN curs_cours;

			LOOP
				FETCH curs_cours INTO cours_record;
				EXIT WHEN NOT FOUND;

				INSERT INTO notes (cours_id, etudiant_id, note)
				VALUES (cours_record.id, etudiant_record.id, RANDOM() * 40 + 60);
			END LOOP;
			
        CLOSE curs_cours;
    END LOOP;
    
    CLOSE curs_etudiants;
END
$$;

-- Question 1 - a

SELECT
    etudiant_id,
    da,
    cohorte,
    AVG(note) AS avg_note,
    PERCENT_RANK() OVER w AS percentile
FROM 
    notes
JOIN 
    etudiants ON notes.etudiant_id = etudiants.id
GROUP BY
    etudiant_id, da, cohorte
WINDOW w AS (ORDER BY AVG(note))
ORDER BY
    avg_note DESC;

-- Question 1 - b

SELECT
    da,
    cohorte,
    AVG(note) AS avg_note,
    RANK() OVER w AS rang
FROM 
    notes
JOIN 
    etudiants ON notes.etudiant_id = etudiants.id
GROUP BY
    etudiant_id, da, cohorte
WINDOW w AS (PARTITION BY cohorte ORDER BY AVG(note) DESC)
ORDER BY 
    cohorte, rang;
	
-- Question 2 - a

EXPLAIN ANALYSE SELECT
    cohorte,
    AVG(note) AS moyenne,
    AVG(note) - LAG(AVG(note)) OVER w AS difference
FROM
    etudiants
JOIN
    notes ON notes.etudiant_id = etudiants.id
GROUP BY
    cohorte
WINDOW w AS (ORDER BY cohorte);

-- Question 2 - b

CREATE TEMP TABLE temp_table AS
SELECT
    etudiants.cohorte,
    notes.note
FROM
    etudiants
JOIN
    notes ON notes.etudiant_id = etudiants.id;

EXPLAIN ANALYSE SELECT
    cohorte,
    AVG(note) AS moyenne,
    AVG(note) - LAG(AVG(note)) OVER w AS difference
FROM
    temp_table
GROUP BY
    cohorte
WINDOW w AS (ORDER BY cohorte);

-- Numéro 2 - c
-- a
--"Planning Time: 0.211 ms"
--"Execution Time: 3.381 ms"

-- b
--"Planning Time: 0.055 ms"
--"Execution Time: 1.387 ms"

-- Le b est beaucoup plus rapide.

-- Numéro 3 - a
CREATE OR REPLACE FUNCTION fetch_prealables_stage()
RETURNS TABLE (id INTEGER, nom TEXT)
LANGUAGE plpgsql
AS $$
DECLARE
    stage_id INT;
    index_tmp_table INT := 0;
BEGIN
    CREATE TEMP TABLE tmp_prealables_stage (id INTEGER, nom TEXT, creation_date TIMESTAMP) ON COMMIT DROP;

    SELECT INTO stage_id cours.id 
    FROM cours
    WHERE cours.nom = 'Stage';
    
    INSERT INTO tmp_prealables_stage(id, nom, creation_date)
    VALUES (stage_id, 'Stage', NOW());
        
    WHILE index_tmp_table < (SELECT COUNT(*) FROM tmp_prealables_stage) LOOP
        INSERT INTO tmp_prealables_stage(id, nom)
        SELECT prealables.prealable_id, cours_prealable.nom
        FROM prealables 
        JOIN cours AS cours_prealable ON prealables.prealable_id = cours_prealable.id
        WHERE prealables.cours_id = (SELECT tmp_prealables_stage.id FROM tmp_prealables_stage
                                     LIMIT 1 OFFSET index_tmp_table)
        AND prealables.prealable_id NOT IN (SELECT tmp_prealables_stage.id FROM tmp_prealables_stage);

        index_tmp_table := index_tmp_table + 1;
    END LOOP;

    RETURN QUERY SELECT tmp_prealables_stage.id, tmp_prealables_stage.nom FROM tmp_prealables_stage;
END
$$;

EXPLAIN ANALYSE SELECT * FROM fetch_prealables_stage();

-- Numéro 3 - b

EXPLAIN ANALYSE WITH RECURSIVE prealables_stage AS (
    SELECT id, nom
    FROM cours
    WHERE nom = 'Stage'
    
    UNION

    SELECT c.id, c.nom
    FROM prealables p
    JOIN cours c ON p.prealable_id = c.id
    JOIN prealables_stage ps ON p.cours_id = ps.id
)
SELECT * FROM prealables_stage;

-- Numéro 3 - c
-- a
--"Planning Time: 0.021 ms"
--"Execution Time: 3.196 ms"

-- b
--"Planning Time: 0.295 ms"
--"Execution Time: 0.193 ms"

-- Le b est beaucoup plus rapide.

```