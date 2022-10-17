# WeLcOmE tO vBa

Language utilisé dans les programmes de Microsoft. 

## Fonction utilisateur
-	Access-SQL permet l’utilisation de fonctions écrites par le programmeur dans une instruction SQL
---
Exemple vba fonction d'utilisateur :
```vb
Public Function DonnerAge(ByVal UneDate As Date) As Double

'Objectif: retourner, sous forme d'un nombre réel la différence 'de temps entre aujourd'hui et UneDate en nombre d'années.

Const JOURS_PAR_AN As Double = 365.24
Dim Différence As Double
Différence = CDbl(Date - UneDate)
DonnerAge = Différence / JOURS_PAR_AN

End Function
```