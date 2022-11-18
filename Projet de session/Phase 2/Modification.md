# **Modification**
## Table JeuxInfo inutile
- Suppression de la table "JeuxInfo"
- Ajout des champs "JeuxCompagnie, JeuxAnnéeDistri, JeuxFournisseur" dans la table "Jeux"
- Ajout de deux relations entre Compagnie - Jeux    
---
## Lien entre manche et locaux
- Compétitions est lié à locaux
- Rajout dans compétition, un champ "locaux"

---
## Lien entre manche et compétition
- Rajout dans manche, un champ "compet"
- Relation entre "manche" et compétition"
- Relation entre Compétition vers tournois(PK)
---
## Devrait pas avoir de lien entre organisateur/compagnie
- Suppression du champ  "orgaCompagnie" à la table "Compagnie"
---
## Une seule table pour les prix (devrait être séparée)

---
- Dans la table "Prix"
  - Champ "PrixEnvergure" : Supression
  - Champ "PrixCompet" : Supression
  - Rajout "PrixChamp"
  -Rennommer la table prix a prixChamp
- Suppresion relation championat - rangCompetition  

- Creation table prixCompet
    - Création champ
        - Même champs que dans prixChamp, excepté le dernier qui est est prixChamp, qui se fait remplacer par PrixCompet
---

- Dans la table "Championat"
    - Renommer "ChampCompet"par "ChampJeux"
    - Supprimer "ChampRang", "ChampAthl", "ChampAthl"
    -Rajout du champ "mode"

- Création relation entre "Jeux" et "Championnat" (CodeJeux(PK) - ChampJeux(FK)) 

---
- Modification table "ClassementsManches"
    - Renommer la table par "ClassementJoueur"
    - Ajout champ "ResultatAthl"
    - Ajout champ "ResultatAthlManche"
    - Enlever champ "TotauxPoints"
- Suppression relation entre manche et ResultatAthl
- Relation CodeRésulatatAthl vers MancheaAthl
- Relation CodeAthl vers ResultatAthl
---
- Suppression table "RangCompetiton"
---

Dans la table "Inscription" Suppression du champ InscriTour"


## Relations manquantes dans équipe
- Suppresion relation entre "Athlete" et "Equipe"
    - Suppression Champ "AthEquipe"
- Création table "InfoEquipe"
    - Création des champs : 
        - CodeInfoEquipe (PK)
        - InfoEquipe (FK)
        - InfoEquipeAthl (FK)
        - InfoEquipeAnnee
- Ajout relation "Athlete" - "InfoEquipe"
- Ajout relation "InfoEquipe" - "Equipe"
-Supression champ "EquipeSponsor"

Suppression champ ChefEquipe dans la table Athlete.

## Table des sponsors à diviser
Duplication de contrat sponsor en deux table :
- ContratSponsorEquipe
- ContratSponsorAthl   
*Supprimer les champs equipe dans Athl/Athl dans Equipe*   
*Réajuster les relations entre les deux tables*   
    - CodeEquipe ver SponEquipe   
    - CodeCompagnie vers SponCompagnie