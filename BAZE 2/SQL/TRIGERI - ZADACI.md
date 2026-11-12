Napisati triger kojim se zabranjuje je bilo kakvo brisanje iz tabele stvaka prijemnice.

```plsql
CREATE OR REPLACE TRIGGER zabranjeno_brisanje
BEFORE DELETE ON STAVKAPRIJEMNICE
BEGIN
    RAISE_APPLICATION_ERROR(-20000, 'Zabranjeno brisanje iz stavka prijemnice');
END;
/
```

Napisati triger kojim se pri ažuriranju stanja zaliha proverava da li je količina proizvoda na 
zalihama posle promena manja od nule.

```plsql
CREATE OR REPLACE TRIGGER provera_kolicine
AFTER INSERT OR UPDATE OF KOLSTANJE ON STANJENAZALIHAMA
FOR EACH ROW
BEGIN
    IF :NEW.KOLSTANJE < 0 THEN
        RAISE_APPLICATION_ERROR(-20001, 'Kolicina stanja na zalihama ne sme biti ispod 0!');
    END IF;
END;
/
```

Napisati triger kojim se dozvoljava rad nad tabelom StanjeNaZalihama samo u radno
vreme. Ukoliko radnik pokuša da radi nad tabelom pre 8 casova, ili nakon 18 treba 
prikazati adekvatnu poruku o grešci. Napomena: Za operaciju ažuriranja je 
karakteristično da ovo ograničenje važi samo za kolicinu.

```plsql
CREATE OR REPLACE TRIGGER zabranjen_rad
BEFORE INSERT OR UPDATE OR DELETE ON STANJENAZALIHAMA
DECLARE
    v_vreme INTEGER;
BEGIN
    v_vreme := TO_NUMBER(TO_CHAR(SYSDATE, 'HH24'));
    IF v_vreme < 8 OR v_vreme >= 18 THEN
        IF INSERTING THEN
            RAISE_APPLICATION_ERROR(-20002, 'Ne smete ubacivati nove podatke u vremenu koje nije 8-18');
        ELSIF DELETING THEN
            RAISE_APPLICATION_ERROR(-20003, 'Ne smete brisati podatke u vremenu koje nije 8-18');
        ELSIF UPDATING('KOLSTANJE') THEN
            RAISE_APPLICATION_ERROR(-20004, 'Ne smete azurirati kolicinu na stanju u vremenu koje nije 8-18');
        END IF;
    END IF;
END;
/
```

Napisati triger koji ažurira stanje zaliha ukoliko je došlo do izmena nad stavkama
prijemnice.

```plsql
CREATE OR REPLACE TRIGGER azuriraj_stanje
AFTER UPDATE OF CENA, KOL ON STAVKAPRIJEMNICE
FOR EACH ROW
BEGIN
    UPDATE STANJENAZALIHAMA
    SET KOLSTANJE = KOLSTANJE - :OLD.KOL + :NEW.KOL,
        IZNSTANJE = IZNSTANJE - ROUND(:OLD.KOL * :OLD.CENA, 2) + ROUND(:NEW.KOL * :NEW.CENA, 2)
    WHERE IDSKLADISTA = :NEW.IDSKLADISTA
        AND IDPROIZVODA = :NEW.IDPROIZVODA;
END;
/
```

Napisati triger koji za svaku novu stavku prijemnice ažurira stanje zaliha. Ukoliko dati
proizvod nije postojao na datom skladištu, onda se ubacuje novi red u Stanje zaliha.

```plsql
CREATE OR REPLACE TRIGGER azuiraj_stanje_za_stavku
AFTER INSERT ON STAVKAPRIJEMNICE
FOR EACH ROW
BEGIN
    UPDATE STANJENAZALIHAMA
    SET KOLSTANJE = KOLSTANJE + :NEW.KOL,
        IZNSTANJE = IZNSTANJE + ROUND(:NEW.KOL * :NEW.CENA, 2)
    WHERE IDPROIZVODA = :NEW.IDPROIZVODA AND
        IDSKLADISTA = :NEW.IDSKLADISTA;
    
    IF SQL%ROWCOUNT = 0 THEN
        INSERT INTO STANJENAZALIHAMA (IDSKLADISTA, IDPROIZVODA, KOLSTANJE, IZNSTANJE)
        VALUES (:NEW.IDSKLADISTA, :NEW.IDPROIZVODA, :NEW.KOL, ROUND(:NEW.KOL * :NEW.CENA, 2));
    END IF;
END;
/
```

Napisati triger kojim se validira plata radnika pri unosu i ažuriranju (plata ne sme biti 
manja od 50.000 ni u kom slučaju)

```plsql
CREATE OR REPLACE TRIGGER validacija_plate
BEFORE INSERT OR UPDATE OF PLATA ON RADNIK
FOR EACH ROW
BEGIN
    IF :NEW.PLATA < 50000 THEN
        RAISE_APPLICATION_ERROR(-20005, 'Plata ne sme biti manja od 50000');
    END IF;
END;
/
```

Dodati kolonu DatumRodjenja u tabeli Radnik. Napisati triger koji pri unosu novog
radnika na osnovu JMBG-a upisuje datum rođenja.

```plsql
ALTER TABLE RADNIK
ADD DATUMRODJENJA DATE;

CREATE OR REPLACE TRIGGER datum_rodjenja_na_osnovu_jmbg
BEFORE INSERT ON RADNIK
FOR EACH ROW
DECLARE
    V_DAN VARCHAR(2);
    V_MESEC VARCHAR(2);
    V_GODINA VARCHAR(3);
    V_CELA_GODINA VARCHAR(4);
BEGIN
    V_DAN := SUBSTR(:NEW.JMBG, 1, 2);
    V_MESEC := SUBSTR(:NEW.JMBG, 3, 2);
    V_GODINA := SUBSTR(:NEW.JMBG, 5, 3);
    
    IF SUBSTR(:NEW.JMBG, 5, 1) = '9' THEN
        V_CELA_GODINA := '1' || V_GODINA;
    ELSIF SUBSTR(:NEW.JMBG, 5, 1) = '0' THEN
        V_CELA_GODINA := '2' || V_GODINA;
    END IF;
    
    :NEW.DATUMRODJENJA := TO_DATE(V_DAN || '.' || V_MESEC || '.' || V_CELA_GODINA || '.');
END;
/
```

Napisati triger koji pri unosu ili ažuriranju naziva proizvoda u tabeli Proizvod 
automatski prebacuje naziv u velika slova i uklanja nepotrebne razmake sa početka i kraja. 
Također, ako je naziv proizvoda kraći od 3 karaktera, triger treba da izbaci grešku i spriječi unos.

```plsql
CREATE OR REPLACE TRIGGER FORMATIRAJ_NAZIV
BEFORE INSERT OR UPDATE OF NAZIVPROIZVODA ON PROIZVOD
FOR EACH ROW
BEGIN
    :NEW.NAZIVPROIZVODA := UPPER(TRIM(BOTH ' ' FROM :NEW.NAZIVPROIZVODA));

    IF LENGTH(:NEW.NAZIVPROIZVODA) < 3 THEN
        RAISE_APPLICATION_ERROR(-20010, 'Naziv proizvoda ne sme biti kraci od 3 karaktera');
    END IF;
END;
/
```

Napisati triger koji zabranjuje brisanje bilo kojeg zapisa iz tabele Skladiste ukoliko u 
tom skladištu trenutno postoji bilo koji proizvod (tj. ako postoji zapis u tabeli 
StanjeNaZalihama za taj ID skladišta).

```plsql
CREATE OR REPLACE TRIGGER NE_BRISI_SKLADISTE
BEFORE DELETE ON SKLADISTE
FOR EACH ROW
DECLARE
    P_BROJ_PROIZVODA NUMBER;
BEGIN
    SELECT COUNT(*) INTO P_BROJ_PROIZVODA
    FROM STANJENAZALIHAMA
    WHERE IDSKLADISTA = :OLD.IDSKLADISTA;
    
    IF P_BROJ_PROIZVODA > 0 THEN
        RAISE_APPLICATION_ERROR(-20011, 'Ne smete brisati skladiste koje sadrzi proizvode');
    END IF;
END;
/
```

Napisati triger koji prati promjene količine u tabeli StanjeNaZalihama. 
Ukoliko korisnik pokuša ažurirati količinu tako da ona postane negativna, 
triger treba da izbaci grešku i prekine operaciju.

```plsql
CREATE OR REPLACE TRIGGER SPRECI_NEGATIVNU_KOLICINU
BEFORE UPDATE OF KOLSTANJE ON STANJENAZALIHAMA
FOR EACH ROW
BEGIN
    IF :NEW.KOLSTANJE < 0 THEN
        RAISE_APPLICATION_ERROR(-20012, 'Kolicina na skladistu ne sme biti negativna');
    END IF;
END;
/
```

Napisati triger koji automatski ažurira stanje zaliha u tabeli StanjeNaZalihama
u slučaju da se iz tabele StavkaPrijemnice obriše neki red.

```plsql
CREATE OR REPLACE TRIGGER AZURIRAJ_OBRISANU_KOLICINU
BEFORE DELETE ON STAVKAPRIJEMNICE
FOR EACH ROW
BEGIN
    UPDATE STANJENAZALIHAMA
    SET KOLSTANJE = KOLSTANJE - :OLD.KOL,
        IZNSTANJE = IZNSTANJE - ROUND(:OLD.KOL * :OLD.CENA, 2)
    WHERE IDPROIZVODA = :OLD.IDPROIZVODA AND
        IDSKLADISTA = :OLD.IDSKLADISTA;
END;
/
```

Napisati triger koji kontroliše kapacitet skladišta. Pri 
svakom pokušaju unosa novog reda u tabelu SanjeNaZalihama, 
triger treba da proveri da li u tom skladištu već postoji 10 
različitih proizvoda. Ukoliko je taj limit dostignut, triger 
treba da izbaci grešku -20020 sa porukom: "Skladište je puno! 
Ne možete dodati novi proizvod u ovo skladište".

```plsql
CREATE OR REPLACE TRIGGER KAPACITET_SKLADISTA
BEFORE INSERT ON STANJENAZALIHAMA
DECLARE
    P_BROJ NUMBER;
BEGIN
    SELECT COUNT(DISTINCT IDPROIZVODA) INTO P_BROJ
    FROM STANJENAZALIHAMA
    WHERE IDSKLADISTA = :NEW.IDSKLADISTA;
    
    IF P_BROJ >= 10 THEN
        RAISE_APPLICATION_ERROR(-20020, 'Skladište je puno! Ne možete dodati novi proizvod u ovo skladište');
    END IF;
END;
/
```

 Napisati triger koji proverava konzistentnost podataka između
 zaglavlja i stavki prijemnice. Prilikom ažuriranja kolone 
 IznUkupno u tabeli Prijemnica, triger treba da izračuna sumu 
 svih stavki za tu konkretnu prijemnicu iz tabele StavkaPrijemnice. 
 Ako se izračunata suma ne poklapa sa novom vrednošću u koloni 
 IznUkupno, triger treba da izbaci grešku -20021 sa porukom: 
 "Ukupan iznos prijemnice nije usklađen sa zbirom njenih stavki!".
 
```plsql
 CREATE OR REPLACE TRIGGER KONZISTENTNOST_PODATAKA
 BEFORE UPDATE OF IZNUKUPNO ON PRIJEMNICA
 DECLARE
    P_UKUPNO NUMBER;
 BEGIN
    SELECT SUM(KOL*CENA) INTO P_UKUPNO
    FROM STAVKAPRIJEMNICE
    WHERE BRDOK = :NEW.BRDOK;
    
    IF P_UKUPNO <> :NEW.IZNUKUPNO THEN
        RAISE_APPLICATION_ERROR(-20021, 'Ukupan iznos prijemnice nije usklađen sa zbirom njenih stavki!');
    END IF;
 END;
 /
```
