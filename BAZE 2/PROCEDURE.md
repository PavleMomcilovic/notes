Deklaracija procedura je sledeca:
```plsql
CREATE [OR REPLACE] PROCEDURE naziv_procedure
[(parametar1 [mode] tip_podatka1,
  parametar2 [mode] tip_podatka2, ...)]
IS|AS
	[deklaracija lokalnih promenljivih]
BEGIN
	upit|naredbe
END [naziv_procedure];
```
Gde mode promenljvih moze da bude IN, OUT ili IN OUT.

Ukoliko zelimo prilikom izvrsenja procedure da stavimo neku vrednost u nasu definisanu varijablu mozemo da koristimo **INTO**.
```plsql
SELECT SUM(plata) INTO p_cena
FROM radnik
```

Ako zelimo da ispisemo neke rezultate ili tekst u konzoli za to koristimo komandu:
```plsql
DBMS_OUTPUT.PUT_LINE('Tekst...');
```
A da bismo uopste mogli da koristimo ovu komandu moramo da je omogucimo na sledeci nacin:
```plsql
SET SERVEROUTPUT ON;
```
