# **Requêtes avec paramètres**

- Access-SQL permet de déclarer des param. dans une requêtes
  - Elle se place avant le select

- **ReqN**
  - On veut connaître les clients demeurant dans une province donnée.
- **ReqO**
  - Je veux connaître les propriétaires qui ont des animaux qui sont nés à un mois donné (paramètre).
- **ReqP**
  - On veut connaître l’âge des animaux vivants.

  ```SQL
    SELECT Aninom, Anitype, Aninaissance,
 		DonnerAge(Aninaissance) AS Age
    FROM Animaux
    WHERE NOT AniEstDécédé;
  ```
    - 122 résultats....

---

### **Tips N Tricks**  
- Assurez-vous que la fonction ne produit aucun effet de bord.

- Soyez conscients que ceci impliquera probablement une certaine conversion si on veut porter cette requête vers un autre SGBD

---

  ## **Traitement des valeurs nulles**

- **Fonction Nz** : **N**ull to **Z**ero

```sql   
Nz(< expr > [, < valeur_si_nulle >])
```
  - Dans une requête utilisez ce champ, sinon la fonction renvoie une chaîne de longueur nulle.
  
  ex SQL :

```SQL
SELECT AutoMarque, Nz(AutoModèle, "inconnu") AS Modèle,
 		Nz(AutoPrix, 0) AS Prix
FROM Autos;
```
---
- **Fonction iif** 
  - Équivalent au si() de Excel
 ```sql
Iif(<condition>, <valeur_si_vrai>, <valeur_si_faux>)
 ```
---
 - **Fonction IsNull**
   - Retourne vrai ou faux
```sql
IsNull(< expr >)
```
ex.
```sql
SELECT AutoMarque,
 		Iif(IsNull(AutoPrix), "aucun prix", AutoPrix) AS Prix
FROM Autos;
```
---
### **Ne pas faire :**
```sql
WHERE AniRace = IsNull(Anirace), pas d’erreur mais pas de résultat!
```
### **Faire :**
```SQL
WHERE	Anirace is Null
    
WHERE	Anirace is not Null
			ou
WHERE	not(AniRace is Null)
```
---

- **Formatage** 
```sql
Format(< expr >, < chaîne de format >)
```
ex
```sql
SELECT AutoMarque, AutoModèle,
		Format(AutoPuissance, "00.00 cv") AS Puissance
FROM Autos
```
---
## **Précisions**

sdfdsf   
sdf
   

## **Sous-Requête**
- C’est une instruction SELECT imbriquée dans une autre instruction SELECT.
- Elle est toujours entre parenthèses


## Requêtes d'action 
- Requête de mise à jour, ex syntaxe:
  - Mots réservé : SET et UPDATE 
```sql
UPDATE <exp_tab_req>
SET <liste_exp>
[WHERE <critères>]
```
---
- ReqU (update)
  - Permet de modifier un ou plusieurs champs de plusieurs enregistrements d’une table ou de plusieurs tables jointes.
```sql
UPDATE Animaux
SET Anitype = "chienne", AniRace = "canine"
WHERE AniType="chien" AND AniSexe="F";
```
---
## **Suppression d'enregistrement**
- Pour supprimer un ou plusieurs enregistrements d'une table.
```sql
DELETE [Table.*]
FROM Table
[WHERE <cond>]
```

**Attention, DELETE** *
- Détruit tous les enregistrements de la table *sans* supprimer la table elle-même (la structure est préservée).
- **Attention à l'intégrité référentielle.**

---
## Supprimer une table 
- Mots clé : DROP TABLE
- Supprime les enregistrements et la structure.
```sql
DROP TABLE JoueursSupprimés2;
```

## Ajout d'enregistrements
- **Syntaxe 1 :**
- Permet d’ajouter plusieurs enregistrements dans une table.
```sql
INSERT [INTO] <nom_Table> [(liste_de_champs)]
SELECT <liste_de_Select>
FROM <Table_Req>
[WHERE <cond>]

```
---
### **Notes**
La liste de champ de la table d’insertion doit coïncider avec la liste de SELECT      

INSERT INTO table : ajoute des enregistrements à la table
SELECT   … INTO table : crée une nouvelle table ou écrase la table actuelle

---

- **Synthaxe 2**
```sql
INSERT INTO <nom_table> [(champ1,champ2,… champn)]
VALUES (val_champ1, val_champ2,… val_champn)
```

   - VALUES accepte une liste d'expression

```sql
INSERT INTO Articles (ArtNom, ArtPrix)
VALUES ("bidule", 15*10)
```
OU

```sql
INSERT INTO Articles (ArtNom, ArtPrix)
VALUES ("cossin", abs(-55))
```
---
### **Notes**
Si on ne met pas de liste de champs, toutes les valeurs doivent y être et de bon type.
---

---
Au final, ...
---