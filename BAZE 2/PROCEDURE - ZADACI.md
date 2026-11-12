Napisati proceduru koja kao parameter prima ID dobavljača i novi naziv tog dobavljača 
na osnovu čega ažurira naziv dobavljača u tabeli Dobavljač i dodaje d.o.o. i ispisuje
Informacije o promeni. Ukoliko dobavljač sa datim ID-jem ne postoji u tabeli, ispisati 
odgovarajuću poruku. 

```plsql
CREATE OR REPLACE PROCEDURE azuriraj_dobavljaca(
    id IN dobavljac.iddobavljaca%TYPE,
    novi_naziv IN dobavljac.nazivdobavljaca%TYPE
) AS
BEGIN
    UPDATE dobavljac
    SET nazivdobavljaca = novi_naziv || ' d.o.o.'
    WHERE iddobavljaca = id;
    
    IF SQL%ROWCOUNT = 0 THEN
        DBMS_OUTPUT.PUT_LINE('Ne postoji dobavljac sa ID-jem ' || id);
    ELSE
        DBMS_OUTPUT.PUT_LINE('Naziv dobavljaca je promenjen na: ' || novi_naziv || ' d.o.o.');
    END IF;
END;
/

SET SERVEROUTPUT ON;
BEGIN
    azuriraj_dobavljaca(1, 'Beta');
END;
/
```

Napisati proceduru koja izračunava ukupnu 
vrednost svih proizvoda za skladište sa datim ID-jem. 
Rezultat ispisati u okviru OUT parametra.

```plsql
CREATE OR REPLACE PROCEDURE ukupna_vrednost_u_skladistu(
    id IN stanjenazalihama.idskladista%TYPE,
    ukupno OUT NUMBER
) AS
BEGIN
    SELECT SUM(iznstanje) INTO ukupno
    FROM stanjenazalihama
    WHERE idskladista = id;
END;
/

DECLARE
    v_iznos NUMBER;
BEGIN
    ukupna_vrednost_u_skladistu(1, v_iznos);
    DBMS_OUTPUT.PUT_LINE(v_iznos);
END;
/
```

Napisati proceduru koja računa ukupan promet dobavljača u zadatom periodu 
(p_dobavljac_id, p_od_datuma i p_do_datuma su zadati kao parametri). Za 
podrazumevanu vrednost početnog datuma staviti 1.1.2000. dok je podrazumevana 
vrednost krajnjeg datuma današnji datum.

```plsql
CREATE OR REPLACE PROCEDURE promet_perioda(
    p_dobavljac_id IN prijemnica.iddobavljaca%TYPE,
    p_od_datuma IN DATE DEFAULT DATE '2000-01-01',
    p_do_datuma IN DATE DEFAULT SYSDATE,
    p_ukupno OUT NUMBER
) AS
BEGIN
    SELECT SUM(iznukupno) INTO p_ukupno
    FROM prijemnica
    WHERE datdok BETWEEN p_od_datuma AND p_do_datuma;
END;
/
```

Napisati proceduru koja kao parametar prima JMBG i procenat (u decimalnom zapisu, 
npr. 0.8 za 80%) i postavlja platu njegovih podređenih radnika na zadati procenat 
njegove plate. Uraditi provere:
- da li radnik sa zadatim JMBG-om postoji
- da li je zadati radnik ima podređene radnike
Za svaki od uslova ispisati odgovarajuću poruku ukoliko nisu ispunjeni.

```plsql
CREATE OR REPLACE PROCEDURE plata_podredjenih(
    p_nadredjeni IN radnik.jmbg%TYPE,
    p_procenat IN NUMBER
) AS
    v_plata_nadredjenog radnik.plata%TYPE;
    v_broj NUMBER;
    v_broj_podredjenih NUMBER;
BEGIN
    SELECT COUNT(*) INTO v_broj
    FROM radnik
    WHERE jmbg = p_nadredjeni;
    
    IF v_broj = 0 THEN
        DBMS_OUTPUT.PUT_LINE('Ne postoji nadredjeni sa datim ID-jem');
        RETURN;
    END IF;
    
    SELECT COUNT(*) INTO v_broj_podredjenih
    FROM radnik
    WHERE nadredjenijmbg = p_nadredjeni;
    
    IF v_broj_podredjenih = 0 THEN
        DBMS_OUTPUT.PUT_LINE('Radnik sa datim ID-jem nema podredjene radnik');
        RETURN;
    END IF;
    
    SELECT plata INTO v_plata_nadredjenog
    FROM radnik
    WHERE jmbg = p_nadredjeni;
    
    UPDATE radnik
    SET plata = v_plata_nadredjenog * p_procenat
    WHERE nadredjenijmbg = p_nadredjeni;
END;
/
```

Napisati proceduru koji izračunava prosečnu platu u zadatom odeljenju.

```plsql
CREATE OR REPLACE PROCEDURE PROSECNA_PLATA(
   V_ID IN RADNIK.ODELJENJE%TYPE,
   V_PROSEK OUT NUMBER
) AS
BEGIN
   SELECT AVG(PLATA) INTO V_PROSEK
   FROM RADNIK
   WHERE ODELJENJE = V_ID;
END;
/

SET SERVEROUTPUT ON;
DECLARE
   P_ODELJENJE RADNIK.ODELJENJE%TYPE;
   P_PROSEK NUMBER;
BEGIN
   P_ODELJENJE := 'Nabavka';
   PROSECNA_PLATA(P_ODELJENJE, P_PROSEK);
   DBMS_OUTPUT.PUT_LINE('Prosecna plata u odeljenju ' || P_ODELJENJE || ' je ' || P_PROSEK);
END;
/
```
 
Napisati proceduru koji izračunava odnos plate zadatog nadređenog u odnosu na 
prosečnu platu svih njegovih podređenih. Uračunati i slučaj kada zadati zaposleni ne 
postoji i kada nema nijednog podređenog.

```plsql
CREATE OR REPLACE PROCEDURE ODNOS_PLATA(
    V_NADREDJENI IN RADNIK.JMBG%TYPE,
    V_ODNOS OUT NUMBER
) AS
    P_PLATA_NADREDJENOG RADNIK.PLATA%TYPE;
    P_PROSEK_PODREDJENIH NUMBER;
    P_BROJ NUMBER;
BEGIN
    SELECT COUNT(*) INTO P_BROJ
    FROM RADNIK
    WHERE JMBG = V_NADREDJENI;
    
    IF P_BROJ = 0 THEN
        DBMS_OUTPUT.PUT_LINE('Nadredjeni sa datim JMBG-om ne postoji.');
        RETURN;
    END IF;
    
    SELECT COUNT(*) INTO P_BROJ
    FROM RADNIK
    WHERE NADREDJENIJMBG = V_NADREDJENI;
    
    IF P_BROJ = 0 THEN
        DBMS_OUTPUT.PUT_LINE('Nadredjeni sa datim JMBG-om nema podredjene');
        RETURN;
    END IF;
    
    SELECT AVG(PLATA) INTO P_PROSEK_PODREDJENIH
    FROM RADNIK
    WHERE NADREDJENIJMBG = V_NADREDJENI;
    
    SELECT PLATA INTO P_PLATA_NADREDJENOG
    FROM RADNIK
    WHERE JMBG = V_NADREDJENI;
    
    V_ODNOS := ROUND(P_PLATA_NADREDJENOG/P_PROSEK_PODREDJENIH, 2);
END;
/

SET SERVEROUTPUT ON;
DECLARE
    P_ODNOS NUMBER;
BEGIN
    ODNOS_PLATA('1503990456123', P_ODNOS);
    DBMS_OUTPUT.PUT_LINE(P_ODNOS);
END;
```

Napisati proceduru koja kao parametar prima IdProizvoda i kao
OUT parametar vraća ukupnu vrijednost tog proizvoda u svim 
skladištima zajedno (suma svih IznStanje za taj proizvod).

```plsql
CREATE OR REPLACE PROCEDURE UKUPNA_VREDNOST_PROIZVODA(
    V_ID_PROIZVODA IN STANJENAZALIHAMA.IDPROIZVODA%TYPE,
    V_UKUPNO OUT NUMBER
) AS
BEGIN
    SELECT SUM(IZNSTANJE * KOLSTANJE) INTO V_UKUPNO
    FROM STANJENAZALIHAMA
    WHERE IDPROIZVODA = V_ID_PROIZVODA;
END;
/

SET SERVEROUTPUT ON;
DECLARE
    P_UKUPNO NUMBER;
BEGIN
    UKUPNA_VREDNOST_PROIZVODA(1, P_UKUPNO);
    DBMS_OUTPUT.PUT_LINE(P_UKUPNO);
END;
/
```

Napisati proceduru koja prima JMBG radnika i ispisuje njegov tačan staž 
u godinama i mjesecima (na osnovu datuma zaposlenja). Ukoliko radnik 
ne postoji, ispisati odgovarajuću poruku.

```plsql
CREATE OR REPLACE PROCEDURE IZRACUNAJ_STAZ(
    V_JMBG IN RADNIK.JMBG%TYPE
) AS
    P_BROJ NUMBER;
    P_GODINE NUMBER;
    P_MESECI NUMBER;
    P_DATUM DATE;
BEGIN
    SELECT COUNT(*) INTO P_BROJ
    FROM RADNIK
    WHERE JMBG = V_JMBG;
    
    IF P_BROJ = 0 THEN
        DBMS_OUTPUT.PUT_LINE('Ne postoji radnik sa datim JMBG-om');
        RETURN;
    END IF;
    
    SELECT DATUMZAPOSLENJA INTO P_DATUM
    FROM RADNIK
    WHERE JMBG = V_JMBG;
    
    P_GODINE := EXTRACT(YEAR FROM (CURRENT_DATE - P_DATUM) YEAR TO MONTH);
    P_MESECI := EXTRACT(MONTH FROM (CURRENT_DATE - P_DATUM) YEAR TO MONTH);
    
    DBMS_OUTPUT.PUT_LINE('Radnik sa JMBG-om ' || V_JMBG || ' ima radni staz od ' || P_GODINE || ' godina i ' || P_MESECI || ' meseci');
END;
/
```

Napisati proceduru koja prima IdDobavljaca i vremenski period 
(od_datuma i do_datuma) i kao OUT parametar vraća ukupan iznos 
svih prijemnica koje su stigle od tog dobavljača u tom periodu.

```plsql
CREATE OR REPLACE PROCEDURE IZNOS_PRIJEMNICA_U_PERIODU(
    V_ID_DOBAVLJACA IN PRIJEMNICA.IDDOBAVLJACA%TYPE,
    V_OD_DATUMA IN DATE,
    V_DO_DATUMA IN DATE,
    V_UKUPNO OUT NUMBER
) AS
BEGIN
    SELECT SUM(IZNUKUPNO) INTO V_UKUPNO
    FROM PRIJEMNICA
    WHERE DATDOK BETWEEN V_OD_DATUMA AND V_DO_DATUMA;
END;
/
```

Napisati proceduru koja prima naziv odeljenja i procenat povećanja 
plate. Procedura treba da poveća plate svim radnicima u tom odeljenju, 
ali samo onima koji trenutno zarađuju manje od 80.000.

```plsql
CREATE OR REPLACE PROCEDURE POVECAJ_PLATU(
    V_NAZIV_ODELJENJA IN RADNIK.ODELJENJE%TYPE,
    V_PROCENAT IN NUMBER
) AS
BEGIN
    UPDATE RADNIK
    SET PLATA = PLATA * (100 + V_PROCENAT)/100
    WHERE ODELJENJE = V_NAZIV_ODELJENJA AND
        PLATA < 80000;
END;
/
```

Napisati proceduru koja kao ulazni parametar prima IdSkladista, 
a kao OUT parametre vraća dve informacije:

-Ukupnu vrednost svih zaliha u tom skladištu.
-ID proizvoda koji ima najveću pojedinačnu vrednost u tom skladištu. 
Ukoliko skladište ne postoji u bazi ili je prazno, procedura treba 
da ispiše odgovarajuću poruku.

```plsql
CREATE OR REPLACE PROCEDURE NAJSKUPLJI_PROIZVOD(
    V_ID IN STANJENAZALIHAMA.IDSKLADISTA%TYPE,
    V_UKUPNO OUT NUMBER,
    V_MAX_ID OUT STANJENAZALIHAMA.IDPROIZVODA%TYPE
) AS
    P_BROJ NUMBER;
BEGIN
    SELECT COUNT(IDPROIZVODA) INTO P_BROJ
    FROM STANJENAZALIHAMA
    WHERE IDSKLADISTA = V_ID;
    
    IF P_BROJ = 0 THEN
        DBMS_OUTPUT.PUT_LINE('Skladiste ne postoji ili je prazno');
        RETURN;
    END IF;
    
    SELECT SUM(IZNSTANJE) INTO V_UKUPNO
    FROM STANJENAZALIHAMA
    WHERE IDSKLADISTA = V_ID;
    
    SELECT IDPROIZVODA INTO V_MAX_ID
    FROM STANJENAZALIHAMA
    WHERE IZNSTANJE = (SELECT MAX(IZNSTANJE)
                        FROM STANJENAZALIHAMA
                        WHERE IDSKLADISTA = V_ID)
        AND ROWNUM = 1;
END;
/

DECLARE
    P_UKUPNO NUMBER;
    P_ID STANJENAZALIHAMA.IDSKLADISTA%TYPE;
BEGIN
    NAJSKUPLJI_PROIZVOD(5, P_UKUPNO, P_ID);
    DBMS_OUTPUT.PUT_LINE('MAX ID: ' || P_ID || ', UKUPAN IZNOS: '|| P_UKUPNO);
END;
/
```

Napisati proceduru za redistribuciju zaliha (prenos proizvoda
iz jednog skladišta u drugo). Procedura treba da prima sledeće
parametre: IdProizvoda, IdSkladista_Od, IdSkladista_Do i 
Kolicina_Za_Premestiti. Procedura mora izvršiti sledeće korake:

-Proveriti da li u polaznom skladištu postoji taj proizvod i da 
li je dostupna tražena količina za prenos. U suprotnom, ispisati 
grešku.
-Oduzeti količinu iz polaznog skladišta.
-Dodati količinu u odredišno skladište (ukoliko proizvod prvi put 
dolazi u to skladište, treba kreirati novi red u tabeli 
SanjeNaZalihama).
-Ažurirati ukupnu vrednost zaliha (IznStanje) u oba skladišta.

```plsql
CREATE OR REPLACE PROCEDURE REDISTRIBUCIJA_ZALIHA(
    V_PROIZVOD IN STANJENAZALIHAMA.IDPROIZVODA%TYPE,
    V_SKLADISTE_OD IN STANJENAZALIHAMA.IDSKLADISTA%TYPE,
    V_SKLADISTE_DO IN STANJENAZALIHAMA.IDSKLADISTA%TYPE,
    V_KOLICINA IN STANJENAZALIHAMA.KOLSTANJE%TYPE
) AS
    P_STANJE STANJENAZALIHAMA.KOLSTANJE%TYPE;
    P_BROJ NUMBER;
    P_IZNOS STANJENAZALIHAMA.IZNSTANJE%TYPE;
BEGIN
    SELECT SUM(KOLSTANJE) INTO P_STANJE
    FROM STANJENAZALIHAMA
    WHERE IDSKLADISTA = V_SKLADISTE_OD AND
        IDPROIZVODA = V_PROIZVOD;
        
    IF P_STANJE = 0 THEN
        DBMS_OUTPUT.PUT_LINE('U skladistu ne postoji proizod');
        RETURN;
    END IF;
    
    IF P_STANJE < V_KOLICINA THEN
        DBMS_OUTPUT.PUT_LINE('Nema dovoljno proizvoda za prenos');
        RETURN;
    END IF;
    
    SELECT (IZNSTANJE / KOLSTANJE) INTO P_IZNOS
    FROM STANJENAZALIHAMA
    WHERE IDSKLADISTA = V_SKLADISTE_OD AND IDPROIZVODA = V_PROIZVOD;
    
    UPDATE STANJENAZALIHAMA
    SET KOLSTANJE = KOLSTANJE - V_KOLICINA
    WHERE IDSKLADISTA = V_SKLADISTE_OD AND
        IDPROIZVODA = V_PROIZVOD;
        
    SELECT COUNT(*) INTO P_BROJ
    FROM STANJENAZALIHAMA
    WHERE IDSKLADISTA = V_SKLADISTE_DO AND
        IDPROIZVODA = V_PROIZVOD;
        
    IF P_BROJ = 0 THEN
        INSERT INTO STANJENAZALIHAMA(IDSKLADISTA, IDPROIZVODA, KOLSTANJE, IZNSTANJE)
        VALUES (V_SKLADISTE_DO, V_PROIZVOD, V_KOLICINA, (V_KOLICINA*P_IZNOS));
    ELSE
        UPDATE STANJENAZALIHAMA
        SET KOLSTANJE = KOLSTANJE + V_KOLICINA,
            IZNSTANJE = IZNSTANJE + (P_IZNOS*V_KOLICINA)
        WHERE IDSKLADISTA = V_SKLADISTE_DO AND
            IDPROIZVODA = V_PROIZVOD;
    END IF;
END;
/
```