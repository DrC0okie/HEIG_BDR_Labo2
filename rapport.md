# BDR : Laboratoire n⁰ 2 - Exercices d’approfondissement Implémentation d’un schéma relationnel

Thomas Germano, Timothee Van Hove

### Exercice 1.3

#### Exercice 1.3.1

`Insérez les projets numéro 3 et 5 dans la table works_on pour l’employé avec le ssn = '123456789' et attribuez 10 heures de travail sur chaque projet. Indiquez les commandes SQL utilisées. Que constatez-vous ? Critiquez et commentez le résultat obtenu.` 

##### Commandes :

```sql
INSERT INTO works_on VALUES('123456789', 3, 10.0);
INSERT INTO works_on VALUES('123456789', 5, 10.0);
```

##### Résultats :

```sql
bdr.company> INSERT INTO works_on VALUES('123456789', 3, 10.0)
[2022-10-11 16:24:45] 1 row affected in 17 ms
bdr.company> INSERT INTO works_on VALUES('123456789', 5, 10.0)
[2022-10-11 16:24:51] 1 row affected in 16 ms
```

Le résultat correspond à celui qui est attendu.

#### Exercice 1.3.2

`Supprimez le département numéro 5 dans la table department . Que constratez-vous ? Critiquez et commentez le résultat obtenu.`

##### Commande :

```sql
DELETE FROM departement where dnumber = 5;
```

##### Résultat :

```sql
bdr.public> DELETE FROM departement where dnumber = 5
[2022-10-18 15:44:45] completed in 1 ms
```

tout s'est bien passé car pour le moment il n'y a pas de clés étrangères liées à ce département

 ### Exercice 1.4

#### Exercice 1.4.1

`Premièrement, il vous faut vider toutes vos tables des différents tuples existants. Indiquez les commandesSQL utilisées.`

```sql
delete from department;
delete from dependent;
delete from dept_location;
delete from employee;
delete from location;
delete from project;
delete from works_on;
```

#### Exercice 1.4.2
`Ensuite, pour chacune des tables, ajoutez les contraintes d’intégrité référentielle (voir Figure 1 du schéma
relationnel COMPANY)`

```sql

alter table  employee alter column fname set not null;
alter table  employee alter column lname set not null;
alter table  employee alter column dno set not null;
alter table employee add foreign key (dno) references department(dnumber);
alter table employee add foreign key (super_ssn) references employee(ssn);

alter table  department alter column dname set not null;
alter table  department alter column mgr_ssn set not null;
alter table  department alter column mgr_start_date set not null;
alter table department add foreign key (mgr_ssn) references employee(ssn);

alter table  project alter column pname set not null;
alter table  project alter column dnum set not null;
alter table project add foreign key (dnum) references department(dnumber);
alter table project add foreign key (plocation) references location(lnumber);

alter table  works_on alter column hours set not null;
alter table works_on add foreign key (essn) references employee(ssn);
alter table works_on add foreign key (pno) references project(pnumber);

alter table  location alter column lname set not null;

alter table dependent add foreign key (essn) references employee(ssn);

alter table dept_location add foreign key (dlocation) references location(lnumber);
alter table dept_location add foreign key (dnumber) references department(dnumber);
```

#### Exercice 1.4.3
`Une fois toutes vos contraintes établies, vous devez peupler à nouveau vos tables avec les fichiers .sql
fournis.`

a) Non, ce n'est pas possible, car il y a des contraintes d'intégrité sur plusieurs colonnes, ce qui emêche leur modification

b) Plusieurs possibilités:
1) ajouter les contraintes après avoir peuplé la base de données
2) étudier le modèle pour savoir si il est possible d'entrer des lignes dans un ordre respectant les contraintes
3) Désactiver temporairement les contraintes de chaque table

#### Exercice 1.4.4
`On souhaite insérer un nouveau département "IT" (avec dnumber = 10 ) et son manager qui est un nouvel
employé (Steve Jobs avec ssn = '555444333' ) qui travaille dans ce nouveau département. Expliquer
la difficulté et proposer une solution.`

Nous voulons ajouter un département dirigé par un employé qui n'existe pas encore. Cela viole la contrainte d'intégrité
de mgr_ssn sur ssn de la table employee.

Pour faire cela, il faut déferrer la vérification de la contrainte avec les commandes suivantes:

```sql
alter table company.department drop constraint department_mgr_ssn_fkey;
alter table company.department add foreign key (mgr_ssn) references company.employee deferrable initially deferred;

alter table company.employee drop constraint employee_dno_fkey;
alter table company.employee add foreign key (dno) references company.department deferrable initially deferred;

begin;
insert into company.department values ('IT', 10, 555444333, '2022-10-20');
insert into company.employee values ('steve', null,'jobs', 555444333, null, null, null, null, null, 10);
commit;
```
Résultat:
```sql
bdr.company> alter table company.department drop constraint department_mgr_ssn_fkey
[2022-10-20 22:44:22] completed in 13 ms
bdr.company> alter table company.department add foreign key (mgr_ssn) references company.employee deferrable initially deferred
[2022-10-20 22:44:22] completed in 6 ms
bdr.company> alter table company.employee drop constraint employee_dno_fkey
[2022-10-20 22:44:22] completed in 6 ms
bdr.company> alter table company.employee add foreign key (dno) references company.department deferrable initially deferred
[2022-10-20 22:44:22] completed in 7 ms
bdr.company> begin
[2022-10-20 22:44:22] completed in 4 ms
bdr.company> insert into company.department values ('IT', 10, 555444333, '2022-10-20')
[2022-10-20 22:44:23] 1 row affected in 7 ms
bdr.company> insert into company.employee values ('steve', null,'jobs', 555444333, null, null, null, null, null, 10)
[2022-10-20 22:44:23] 1 row affected in 4 ms
bdr.company> commit
[2022-10-20 22:44:23] completed in 5 ms
```

#### Exercice 1.4.5
a)`Dans la table employee , modifiez le dno du tuple qui a le ssn = '999887777' à 7`:

Etant donné que le département 7 n'existe pas, nous l'avons créé, puis nous avons changé le numero de dpt à l'employé par la suite
```sql
insert into department values ('Dept7',7,555444333,'2022-10-10');
update employee SET dno = 7 where ssn = '999887777';
```

b)`Dans la table employee , supprimez le tuple correspondant à ssn = '999887777'`:

Etant donné qu'un essn était référencé sur le ssn de l'employé que nous voulions supprimer, il y a une violation de contrainte de clé.
Nous dvons donc définir un comportement lors de la suppréssion de la clé étrangère avec une suppression en cascade:

```sql
alter table company.works_on drop constraint works_on_essn_fkey;
alter table company.works_on add foreign key (essn) references company.employee on delete cascade;
delete from employee where ssn = '999887777';
```

c)`insérez les projets numéro 3 et 5 dans la table works_on pour l’employé avec le ssn = '123456789' et attribuez 10 heures de travail sur chaque projet`:

Comme le projet 4 n'existe pas dans la table project, il suffit de l'ajouter. Pour cela nous avons choisi le même numero de département que l'employé avec le ssn '123456789'

```sql
insert into works_on VALUES ('123456789', 3, 10.0);
insert into project values ('proj4', 4, null, 5);
insert into works_on VALUES ('123456789', 4, 10.0);

```

d)`Supprimez le département numéro 5 dans la table department`:

Comme 3 clés étrangères sont liées au numéro de projet, il suffit de donner une valeur par défaut aux clés étrangères, puis de définir le comportement lors de la suppression de celle-ci (set default).

```sql
-- Defining a default value for the foreign key
alter table project alter column dnum set default 4;
-- Change the behavior of the foreign key on delete
alter table project drop constraint project_dnum_fkey;
alter table project add foreign key (dnum) references department on delete set default;

-- Defining a default value for the foreign key
alter table employee alter column dno set default 4;
-- Change the behavior of the foreign key on delete
alter table employee drop constraint employee_dno_fkey;
alter table employee add foreign key (dno) references department on delete set default;

-- Defining a default value for the foreign key
alter table dept_location alter column dnumber set default 4;
alter table dept_location drop constraint dept_location_dnumber_fkey;
-- Change the behavior of the foreign key on delete
alter table dept_location add foreign key (dnumber) references department on delete set default;

-- Delete the department without constraints problems
delete from department where dnumber = 5
```
