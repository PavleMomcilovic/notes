### Primer trigera
```plsql
CREATE OR REPLACE TRIGGER <naziv trigera>
{BEFORE | AFTER| INSTEAD OF}
{INSERT | UPDATE [OF <naziv kolone1>, <naziv kolone2>,...] | DELETE}
ON <naziv tabele>
[FOR EACH ROW ]
[WHEN (<uslov>)]
{<SQL naredba>
| BEGIN 
{<SQL naredba>;}...
END}
```

Definicija da li se triger izvrsava pre, posle ili umesto naredbe:
`{BEFORE | AFTER| INSTEAD OF} `

Definicija na koju vrstu operacije azuriranja se okida triger:
`{INSERT | UPDATE [OF <naziv kolone1>, <naziv kolone2>,...] | DELETE}`

Definicija da li zelimo da se triger okine za svaki red ili samo na nivou upita (po defaultu je za upit):
`[FOR EACH ROW ]`

### Korisne stvati za pisanje trigera

Ako zelimo da uhvatimo neku gresku, koristimo komandu:
`RAISE_APPLICATION_ERROR(-20000, 'Zabranjeno brisanje iz tabele StavkaPrijemnice.')
gde je kod greske bilo koji broj izmedju -20999 i -20000, jer sql koristi sifre gresaka od -19999 do 0.

Ovo je korisno ukoliko zelimo da povezemo nasu bazu sa nekim backendom definisanim u drugom programskom jeziku kao sto je npr. Java i onda Java-in kompajler zna tacno koju gresku je uhvatio.

Ukoliko zelimo da uporedimo vrednost neke kolone pre i posle izvrsavanja nekog upita, u okviru trigera mozemo koristiti:
1. **:NEW.naziv_kolone** notaciju - Ovo predstavlja vrednost kolone nakon izvrsavanja operacije azuriranja.
2. **:OLD.naziv_kolone** notaciju - Ovo predstavlja vrednost kolone pre izvrsavanja operacije azuriranja
**NAPOMENA** - OVO SE MOZE KORISTITI ISKLJUCIVO AKO SE TRIGER OKIDA PRE OPERACIJE, TJ. AKO IMA BEFORE KLAUZULU

Jos jedna napomena je da kod INSERT operacije ne postoji :OLD vrednost jer tek unosimo novu i da kod DELETE operacije ne postoji :NEW vrednost jer je nova vrednost NULL.


