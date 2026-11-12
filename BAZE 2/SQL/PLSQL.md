PL/SQL predstavlja prosirenje SQL jezika u ORACLE-u. Dozvoljava da izvrsavamo slozene upite proceduralno.

Postoje 3 vrste blokova u PL/SQL
1. Anonimni blokovi
```plsql
[DECLARE
lista_parametara]
BEGIN
	naredbe
[EXCEPTION]
END;
```
2. Procedure
```plsql
PROCEDURE naziv(lista_parametara) IS
BEGIN
	naredbe
[EXCEPTION]
END;
```
3. Funckije
```plsql
FUNCTION naziv(lista_parametara) RETURN tip_podatka IS
BEGIN
	naredbe
	RETURN vrednost;
[EXCEPTION]
END;
```

Deklaraciju promenljivih u PL/SQL vrsimo na sledeci nacin:
```plsql
naziv_promenljive [CONSTANT] tip_promenljive [NOT NULL] [:= izraz];
```
PL/SQL podrzava proste i slozene tipove podataka. Prosti tipovi podataka koje podrzava su: CHAR, VARCHAR2, NUMBER, DATE, BOOLEAN. Slozeni tipovi su Tabele i Rekordi.

Implicitna deklaracija promenljivih je takodje moguca u okviru PL/SQL-a.
```plsql
-- definisemo promenljivu tipa i duzine kao kolona tabele
naziv_promenljive naziv_tabele.naziv_kolone%TYPE

-- definisemo promenljivu tipa rekord koja ima ista polja kao referencirana tabela
naziv_promenljive naziv_tabele%ROWTYPE
```
### PETLJE

Postoje razlicite vrste petlji u PL/SQL jeziku kao sto su LOOP ... EXIT WHEN i WHILE ... LOOP.
```plsql
LOOP
	programske_naredbe
	EXIT WHEN uslov;
END LOOP;

WHILE uslov LOOP
	programske_naredbe
END LOOP;
```

Takodje, kao i u vecini programskih jezika, postoji i FOR petlja
```plsql
FOR brojac IN vred1..vred2 LOOP
	naredbe
END LOOP;
```
### TRIGERI

Trigeri su objekti seme podataka koji su vezani za tacno jednu bazu i pozivaju se svako put kada se desi neka operacija azuriranja (INSERT, UPDATE, DELETE). Ne dozvoljava se definisanje trigera nad SELECT upitom.

Trigeri se mogu pozivati u razlicitim trenucima izvrsenja: pre, posle ili umesto neke operacije azuriranja. Takodje, trigeri se mogu pozivati na nivou reda ili na nivou upita.

U nastavku se nalaze bitne komande za trigere: [[TRIGERI]]
Takodje i zadaci sa resenjima: [[TRIGERI - ZADACI]]
### PROCEDURE

Procedure se ponasaju kao i u svakom drugom programskom jeziku, a to je da su one skup naredbi koje su navedene pod jednim nazivom i opciono primaju spisak parametara. Takodje, opciono vracaju neku vrednost i pisu se da bi nam olaksale ponovno pozivanje nekog bloka naredbi.

U nastavku se nalaze bitne komande za procedure: [[PROCEDURE]]
Takodje i zadaci sa resenjima: [[PROCEDURE - ZADACI]]