# BDR : Laboratoire n⁰ 2 - Exercices d’approfondissement Implémentation d’un schéma relationnel

## Exercice 1

### Exercice 1.3

#### Exercice 1.3.1

`Insérez les projets numéro 3 et 5 dans la table works_on pour l’employé avec le ssn = '123456789' et attribuez 10 heures de travail sur chaque projet. Indiquez les commandes SQL utilisées. Que constatez-vous ? Critiquez et commentez le résultat obtenu.` 

##### Commandes :

```sql
INSERT INTO works_on VALUES('123456789', 3, 10.0);
INSERT INTO works_on VALUES('123456789', 5, 10.0);
```

###### Résultats :

```sql
bdr.company> INSERT INTO works_on VALUES('123456789', 3, 10.0)
[2022-10-11 16:24:45] 1 row affected in 17 ms
bdr.company> INSERT INTO works_on VALUES('123456789', 5, 10.0)
[2022-10-11 16:24:51] 1 row affected in 16 ms
```

