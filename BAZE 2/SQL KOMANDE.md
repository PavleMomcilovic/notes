## Klauzule u upitima

Ukoliko zelimo da izbacimo samo neponavljajuce vrednosti nekih kolona koristimo:
```sql
SELECT DISTINCT posao
FROM zaposleni;
```

Ako zelimo da obuhvatimo neki opseg vrednosti za kolone numerickog tipa imamo 2 nacina:
```sql
1. nacin
   WHERE plata > 50000 AND plata < 100000
2. nacin
   WHERE plata BETWEEN 50000 AND 100000
```

Ako zelimo da obuhvatimo vise vrednosti koje moze da ima neka kolona tipa npr. VARCHAR, to mozemo postici na 2 nacina:
```sql
1. nacin
   WHERE posao = 'ANALITICAR' OR posao = 'SAVETNIK'
2. nacin
   WHERE posao IN ('ANALITICAR', 'SAVETNIK')
```
Takodje, ukoliko zelimo da izaberemo redove gde vrednost date kolone nije jedna od zadatih mozemo da koristimo <> (nije jednako) ili NOT IN za definisanje skupa vrednosti.

Ukoliko zelimo da izaberemo redove u kojima neka kolona tipa npr. VARCHAR ima vrednosti koje sadrze neko slovo ili pocinju/zavrsavaju se s nekim slovom, mozemo koristiti klauzulu LIKE.
```sql
WHERE posao LIKE '%A%'   -- svi poslovi koji sadrze slovo A
```
Karakteri koji se koriste uz ovu klauzulu su:
1. % - oznacava 0 ili vise karaktera
2. \_ - oznavava tacno jedan karakter
Ukoliko zelimo da koristimo neki od ova 2 karaktera u samom upitu mozemo da definisemo neki nas ESCAPE karakter koji oznacava da se specijalni karakter ispred koga se on nalazi ponistava kao specijalni i cita u samom upitu. Npr. ako zelimo da pretrazimo neke nazive koji sadrze karakter \_ napisali bismo u upitu:
```sql
LIKE '%!_%' ESCAPE '!'
```
Ovde smo definisali da nam je '!' nas ESCAPE karakter.

Ukoliko zelimo da sortiramo rezultate naseg upita, na kraju upita pisemo ORDER BY klauzulu. Ona moze da sortira rastuce ili opadajuce po razlicitim kolonama, ali moramo da navedemo sve kolone koje smo izabrali u nasem SELECT upitu.
```sql
SELECT kolona1, kolona2
FROM tabela
WHERE kolona3 = 10
ORDER BY kolona1 ASC, kolona2 DESC
```
Primeticemo da mozemo sortirati u drugacijem redosledu po drugim kolonama (ASC - rastuce, DESC - opadajuce).

## Funkcije za obradu pojedinacnih redova

### 1. Numericke

**INSTR** funkcija vraca numericku vrednost, tj. poziciju na kojoj pocinje trazeni string u okviru nekog drugog stringa.
```sql
INSTR(string2, string1 [, pocetak])
```
gde je "pocetak" pozicija od koje se pocinje pretraga. Ako pronadje string1 u string2, vraca njegovu poziciju, u suprotnom vraca 0.

**LENGTH** vraca numericku vrednost koja je jednaka duzini prosledjenog stringa.
```sql
LENGTH(string)
```

**EXTRACT** izdvaja dan, mesec ili godinu (DAY, MONTH, YEAR) iz kolone tipa DATE i vraca numericku vrednost
```sql
EXTRACT(DAY FROM datum) AS dan
EXTRACT(MONTH FROM datum) AS mesec
EXTRACT(YEAR FROM datum) AS godina

EXTRACT(HOUR FROM CAST(SYSDATE AS TIMESTAMP))
```

**ABS** vraca apsolutnu vrednost prosledjene numericke kolone, dok **MOD** vraca ostatak pri deljenju dve prosledjene numericke kolone.
**MOD** vraca pozitivnu vrednost ukoliko je prvi parametar pozitivan, a negativnu vrednost ukoliko je negativan.
```sql
MOD(12, 5)  -- Rezultat je 2
MOD(-12, 5) -- Rezultat je -2
```

Jos neke numericke funkcije koje ORACLE ima na raspolaganju su:
```sql
POWER(broj, e)            -- dize broj na e-ti stepen
ROUND(broj [,d])          -- zaokruzuje broj na "d" decimala, ukoliko d ne postoji                              zaokruzuje ga na ceo broj
TRUNC(broj [,d])          -- odbacuje ostatak od d-tog decimalnog mesta, ukoliko                                d ne postoji odbacuje sve iza decimalne tacke
SIGN(broj)                -- vraca 1 ako je broj >0, 0 ako je =0, -1 ako je <0
SQRT(broj)                -- nalazi pozitivan kvadratni koren broja
```

### 2. Tekstualne

**SUBSTRING** izdvaja podniz karaktera od zadatog pocetka duzine opciono zadate, ili ukoliko duzina nije zadata do kraja stringa.
```sql
SUBSTRING('STRING' FROM 1 FOR 3) -- Rezultat je 'STR'
```

Funkcije **UPPER** i **LOWER** prebacuju prosledjen string u uppercase, tj. u lowercase.

Funkcija **TRIM** uklanja zadat karakter sa pocetka, kraja ili i sa pocetka i sa kraja.
```sql
TRIM(LEADING ' ' FROM ' TEST ')   -- 'TEST '
TRIM(TRAILING ' ' FROM ' TEST ')  -- ' TEST'
TRIM(BOTH ' ' FROM ' TEST ')      -- 'TEST'
```

**OPERATOR ||** sluzi za konkatenaciju stringova, npr. string1 || ' ' || string2, spaja string1 i string2 sa razmakom.

### 3. Datum

 **CURRENT_DATE** prikazuje trenutni datum.
 **CURRENT_TIME** prikazuje trenutno vreme, sa vremenskim odstupanjem, tj. vremenskom zonom, dok **LOCALTIME** prikazuje trenutno vreme bez odstupanja.
**CURRENT_TIMESTAMP** i **LOCALTIMESTAMP** prikazuju i datum i vreme sa funkcionalnostima prethodno navedenih funkcija.

Ako zelimo na datum da dodamo ili oduzmemo neki interval koristimo funkciju **INTERVAL**..
```sql
CURRENT_DATE + INTERVAL '1' DAY -- povecava trenutni datum za 1 dan
```
Ali ukoliko zelimo da oduzmemo jedan datum od drugog datuma dobijamo vrednost intervalnog tipa, npr.
```sql
(CURRENT_DATE - DATZAP) YEAR TO MONTH
```
Po defaultu, ova funkcija vraca rezultat u broju dana, ali u ovom slucaju ce vratiti rezultat u godinama i mesecima. Ukoliko zelim samo godine mogu da odradim sledece.
```sql
EXTRACT(YEAR FROM (CURRENT_DATE - DATZAP) YEAR TO MONTH)
```
Ukoliko mi je potrebno da odredim broj meseci izmedju neka dva perioda u vremenu koristim:
```sql
MONTHS_BETWEEN(CURRENT_DATE, DATZAP)
```

### 4. Logicke

U ovu grupu spadaju funkcije **EVERY, ANY, SOME**.
One vracaju boolean vrednost na osnovu prosledjenog uslova ukoliko su respektivno svaki, bilo koji ili poneki uslov tacni za ove 3 funkcije.
## Specijalne naredbe (CASE, NULLIF, COALESCE, CAST)

Prva blok naredba koju cu navesti je **CASE**, koja u SQL-u funkcionise isto kao i u svakom drugom programskom jeziku, ali ovde ima ovakvu sintaksu:
```sql
CASE
	WHEN uslov1 THEN rezultat1
	WHEN uslov2 THEN rezultat2
	...
	ELSE rezultatn
END
```

Ukoliko zelimo da zamenimo neku vrednost, npr. -1 u datoj koloni sa NULL to mozemo uraditi uz pomoc CASE-a na sledeci nacin:
```sql
CASE WHEN plata = -1 THEN NULL ELSE plata
```
Ovo mozemo uraditi na mnogo jednostaviji nacin koristeci specijalnu funkciju NULLIF:
```sql
NULLIF(plata, -1)   -- vraca null ako su dve prosledjene vrednosti jednaka, a u                           suprotnom vraca vrednost prvog prosledjenog izraza
```

Ukoliko zelimo da prosledimo vise uslova koji bi isli zaredom i ako zelimo da se vrati vrednost prvog izraza koji nije NULL koristimo funkciju **COALESCE**.
```sql
COALESCE(izraz1, izraz2, ..., izrazn)
```

Ukoliko nam je potrebno da neku vrednost pretvorimo iz jednog u drugi tip podatka koristimo funkciju **CAST**
```sql
CAST(izraz1 TO tip_podatka)
```

## Agregatne funkcije

Najznacajnije agregatne funkcije su:
```sql
AVG(naziv_kolone)            -- izracunava srednju vrednost
SUM(naziv_kolone)            -- izracunava ukupnu vrednost
MIN(naziv_kolone)            -- nalazi minimalnu vrednost
MAX(naziv_kolone)            -- nalazi maksimalnu vrednost
COUNT(*)                     -- nalazi broj redova u grupi
COUNT([ALL]naziv_kolone)     -- nalazi broj definisanih (not null) vrednosti kolone
COUNT(DISTINCT naziv_kolone) -- nalazi broj razlicitih definisanih vrednosti kolone
```

Ako zelimo da sortiramo rezultate agregatnih funkcija po nekoj osnovi, koristimo klauzulu **GROUP BY**. Bitna stvar s ovom klauzulom je to da se moraju koristiti sve kolone navedene u SELECT klauzuli u okviru GROUP BY klauzule.
```sql
SELECT sifraodelj, posao, COUNT(*)
FROM radnik
GROUP BY sifraodelj, posao;
```
Po defaultu je ASC order, ali mozemo da dodamo i DESC i to ispred svake kolone.

**HAVING** klauzula odredjuje kriterijume za selekciju grupa koje su prethodno specificirane GROUP BY klauzulom.
```sql
SELECT sifraodelj, posao, COUNT(*)
FROM radnik
GROUP BY sifraodelj, posao
HAVING COUNT(*) > 2;
```

## Skupovne operacije

Skupovne operacije se navode prilikom SELECT izmedju 2 razlicita upita. Postoje 3 vrste skupovnih operacija
```sql
UNION     -- prikazuje uniju 2 select upita
INTERSECT -- prikazuje presek 2 select upita
EXCEPT    -- prikazuje razliku 2 select upita
```
Sve ove operacije je moguce navesti kod SELECT upita sa kompatibilnim tabelama.

## Spajanje tabela

Spajanje tabela vrsimo pomocu **JOIN** klauzule.
```sql
SELECT naziv, ime, posao
FROM odeljenje JOIN radnik ON odeljenje.sifraodelj = radnik.sifraodelj

ili

SELECT naziv, ime, posao
FROM odeljenje JOIN radnik
USING (sifraodelj)
```

Postoje posebne vrste spajanja:
1. **LEFT JOIN** - spaja tabelu1 sa tabelom2 i ukoliko se nadje kolona u tabeli1 koja nema nijednu vrednost s kojom je povezana u tabeli2 prikazuju se vrednosti iz tabele1 uz NULL vrednosti za kolone iz tabele2
2. **RIGHT JOIN** - isto kao levo spajanje s tim sto se obrcu tabela1 i tabela2
3. **FULL JOIN** - kombinuje levo i desno spajanje