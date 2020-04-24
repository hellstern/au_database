[Retur](README.md)

# Kursusgang 5

### SQL fil
[Link til SQL fil](https://www.dropbox.com/s/g71ggir8vnvw0hp/Dag_5.sql?dl=0)

## STORED PROCEDURE

En **STORED PROCEDURE** (SP) er SQL-kode, som du har gemt, så koden kan bruges igen og igen.

Hvis du har en SQL-forespørgsel, du skriver igen og igen, vil det være en fordel at gemme den som en SP. Du skal så bare "kalde" den for at få udført SQL koden.

Du kan også sende parametre/variabler til en SP. Hvilket gør at én SP kan bruges til flere forskellige udtræk.

Det kan f.eks. være; kundenummer, produktnummer, dato/måned/år eller lignede du brugere som parameter/variabel.

**Syntaks for en STORED PROCEDURE**

```
CREATE PROCEDURE procedure_navn
AS
	sql_kode;
GO;
```

Når ud skal "kører"/afvikle en STORED PROCEDURE skal du bruge denne SQL kommando

```
EXEC procedure_navn;
```

En SP bliver gemt i databasen, du kan se hvilke SP der ligger på den enkelte database i mappen **Stored Proceduers**

![Stored Proceduers](images/StoredProceduers.png)

**Der er flere fordele ved at bruge SP:**

- Hvis den sammen SP bruges flere gange af den sammen forbindelse/applikation vil den blive afviklet hurtigere.
- Der er mindre data-trafik mellem en applikation og MySQL databasen, da det kun er navnet på SP og evt. parameter der sendes over netværket.
- Det er mere sikkert at bruge en SP, da adgange er begrænset til det SP har adgang til.

**Der er dog også nogle ulemper ved at bruge** SP:

- Det er svært at fejl-finde og fejl-rette en SP
- Det kan være svært at vedligehold og have et overblik over mange SP
- Hvis der er mange logiske operationer i en SP vil den bruge meget RAM.
- 

### DELIMITER

Når du skal oprette en SP er det nødvendigt først at bruge en anden SQL kommando, DELIMITER.

Problemet er at vi ønsker at sende **hele** SQL kommandoen for SP til serveren. I den SQL kode der indgår i SP'en er der en "indlejret" SQL kode afsluttes med **;**.

Det er derfor nødvendigt at ændre **DELIMITER** til noget andet end det normale **;** - oprette SP - sætte **DELIMITER** retur til **;**.

Du kan bruge flere forskellige ""tegn" til DELIMITER f.eks. // eller $$

Her sættes DELIMITER til **//** og tilbage til **;**

```
DELIMITER //

CREATE PROCEDURE procedure_navn
AS
	sql_kode;
GO;

DELIMITER ;
```

Det er ikke kun når du opretter SP at det er nødvendigt, det glæder f.eks. også når du skal oprette **Trigger**.


### VARIABLE

En af de store fordele ved en SP er at du kan bruge variable, hvilket gør dine SP'er mere fleksible.

Når du vil bruge en variabel skal du angive (dimensionere) dens datatype. Det er de sammen datatyper som ved oprettelse af felter i en tabel du skal bruge. Typisk vil datatypen være det sammen som feltet du brugere variablen op mod.

**Eksempel Northwind** med én variabel (Aar) af typen INT - her er det retur typen af funktionen YEAR der afgør datatypen.

```
DELIMITER $$

CREATE PROCEDURE OrderByYear(Aar INT)
BEGIN
	SELECT * FROM Orders
	WHERE YEAR(OrderDate) = Aar;
END $$

DELIMITER ;

```

Når du vil afvikle denne SP gøres det på følgende måde.

**Eksempel med året 1996**

```
CALL OrderByYear(1996);
```

Det er også muligt at brugere flere variabler. De skal bare angives adskilt af et **,**

**Eksempel fra Northwind**

```
DELIMITER $$
CREATE PROCEDURE OrderByYearMonth(Aar INT, Mdr INT)
BEGIN
	select 
		Products.ProductID,
		Products.ProductName
	from Products
	join Order_Details
	on Products.ProductID = Order_Details.ProductID
	join Orders 
	on Order_Details.OrderID = Orders.OrderID
	where year(Orders.OrderDate) = Aar and
		month(Orders.orderdate) = Mdr;
END $$
DELIMITER ;
```

Når du skal afvikle en SP med flere variabler skal angive dem i sammen rækkefølge som de er oprettet.

```
CALL OrderByYearMonth (1997, 5);
```

## Vis STORED PROCEDURE

Du kan få vist de SP'er der er i den enkelte database med denne SQL kommando

```
SHOW PROCEDURE STATUS WHERE db = 'database_navn';
```

Hvid du geren vil se SQL koden for en bestem SP kan du bruge denne SQL kommando:

```
SHOW CREATE PROCEDURE procedure_navn;
```

### Slet STORED PROCEDURE

Du sletter en SP ved at brugere denne SQL kommando

```
DROP PROCEDURE `datbase_navn`.`sp_navn`;
```

*Tip - Du kan også højreklikke på en SP i Workbench og vælge DROP Stored Procedure*


## TRIGGER
En **Trigger** er SQL kode, der afvikles automatisk, når en af følgende handlinger sker

- INSERT
- UPDATE
- DELETE

på den tabel hvor triggeren er oprettet.

Du kan betragte en **TRIGGER** som en speciel STORED PROCEDURE. Forskellen er at en TRIGGER bliver afviklet automatisk ved en af de tre førnævnte handlinger. Mens en STORED PROCEDURE skal afvikles "manuelt".

Triggere er gode til opgaver som:

- håndhævelse af forretningsregler
- validering af input
- audit trial - typisk ved at skrive til andre tabeller

### TRIGGER Syntaks

Du opretter en TRIGGER på følgende måde:

```
CREATE TRIGGER 
	trigger_navn 
	trigger_time 
	trigger_event
 ON tabel_navn
 FOR EACH ROW
 BEGIN
 	SQL kode
 END;
```

- **trigger_time** er enten **BEFORE** eller **AFTER**
- trigger_event er; **INSERT**, **UPDATE** eller **DELETE**
- Din SQL kode skal placeres mellem **BEGIN** og **END**

**Eksemple fra Northwind**

```
DELIMITER $$

CREATE TRIGGER before_employee_update 
    BEFORE UPDATE ON Employees
    FOR EACH ROW 
BEGIN
    INSERT INTO employees_audit
    SET action = 'update',
		EmployeeID = OLD.EmployeeID,
        LastName = OLD.LastName,
        changedat = NOW(); 
END $$

DELIMITER ;
```

En TRIGGER bliver gemt "under"/på en tabel - den tabel som en af de tre handlinger udføres på.

![Trigger](images/Trigger_1.png)

### Vis TRIGGER

Du kan med denne SQL kommando se dine TRIGGER

```
SHOW TRIGGERS;

eller

SHOW TRIGGERS FROM northwind;

eller

SHOW TRIGGERS FROM northwind
WHERE `table` = 'Employees';
```

*Bemærk tegnet før og efter table*

Der er også en anden måde du kan se de forskellige TRIGGER du har. Det er sådan at definitionen på dine TRIGGERE bliver gemt i en system tabel ved navn: **information_schema**

**Se alle TRIGGERE på en bestemt database**

```
SELECT * FROM
	information_schema.triggers
WHERE
    trigger_schema = 'database_navn';
```

Der er også muligt at finde alle TRIGGERE på en bestem tabel.

```
SELECT * FROM
    information_schema.triggers
WHERE
    trigger_schema = 'database_navn'
        AND event_object_table = 'table_navn';
```

### Slet TRIGGER
Du sletter en TRIGGER ved at brugere denne SQL kommando

```
DROP TRIGGER trigger_navn;
```


## EXPORT - IMPORT CSV

Det er muligt både at eksportere og importere data fra CSV (**C**omma **S**eparated **V**alues) filer via SQL kommandoer.

Fordelen ved at bruge CSV er at filformatet kan læses af mange andre programmer.

### EXPORT

Når du skal eksportere data skal du angive følgende parameter:

- Filnavn: **INTO OUTFILE**
- Tegn der "omslutter" felterne: **FIELDS ENCLOSED BY '"'**
- Escape karakter bruges i tilfælde af specialtegn: **TERMINATED BY ';'**
- Afslutning: **ESCAPED BY '"'**
- Linje skift: **LINES TERMINATED BY '\r\n';**

Ud over det skal du udvælge de data du geren vil eksportere. Til det brugere du en almindelig SELECT sætning evt. med en WHERE på.

**Eksempel fra Northwind** der eksportere alle kunder fra USA.

*Bemærk at der bruges en* **UNION ALL** *for at få overskrifter med.*

```
SELECT
	'CustomerID',
        'CompanyName',
        'Address',
        'City',
        'Region',
        'PostalCode',
        'Country'
UNION ALL
SELECT 
	CustomerID,
        CompanyName,
        Address,
        City,
        Region,
        PostalCode,
        Country
FROM Customers
WHERE Country = 'USA'
INTO OUTFILE '/var/lib/mysql-files/Customer_USA.csv' -- Linux
-- INTO OUTFILE 'C:\\Customer_USA.csv' -- Windows
FIELDS ENCLOSED BY '"'
TERMINATED BY ';'
LINES TERMINATED BY '\n';
```


### IMPORT

Der er også muligt at importere data fra en CSV fil.

*Her er det eksemplet fra eksporten der importeres ind i en tabel med navnet* **Customers_USA**. *Den tabel skal oprettes først.*

```
-- LOAD DATA INFILE 'C:\\Customer_USA.csv' -- Windows
LOAD DATA INFILE '/var/lib/mysql-files/Customer_USA_2.csv' -- Windows
INTO TABLE Customers_USA
FIELDS ENCLOSED BY '"'
TERMINATED BY ';'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;
```

*Bemærk at den første* **ROW** *ikke importes, da den indeholder overskrifter.*

[Retur](README.md)
