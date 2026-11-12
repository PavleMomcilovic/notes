XML je prvi koji je zapoceo ideju samoopisujucih datoteka. Kreiran je da bude lako citljiv i od strane ljudi i od strane masina. JSON (JavaScript Object Notation) definise strukturu podataka koja se danas moze koristiti i van JavaScripta.

Ukoliko je i sa strane posiljaoca i sa strane primaoca poznata XML/JSON notacija, onda oni lako mogu da spoje kljuceve sa svojim vrednostima.

Sa aspekta nivoa struktuiranosti mozemo definisati 3 tipa podataka
1. Nestruktuirani podaci (tekst u dokumentu)
2. Polu-struktuirani podaci (ima strukturu nekog stabla)
3. Struktuirani podaci (tabele u kojima svaka kolona ima odredjene vrste vrednosti)

JSON cuva podatke u polu-struktuiranom obliku.
```json
{
	'key1': value1,
	'key2': {
		'key3': value3,
		'key4': value4
	}
}
```

JSON podrzava dva struktuirana tipa:
1. Objekat predstavlja neuredjenu kolekciju nula ili vise kljuceva - notacija {}
2. Niz predstavlja uredjenu kolekciju nula ili vise kljuceva. - notacija \[]

JSON je za razliku od relacionih modela dosta fleksibilniji, tj. mozemo da ubacujemo novu strukturu i nove podatke bez unapred definisanja tabela, pravila, itd. Pored toga, JSON svakako moze da definise neka pravila.
Prednosti JSON-a u odnosu na XML jeste to sto nema zatvarajuci tag, brzi je za pisanje, moze koristiti nizove.

Programski jezik Java ima brojne biblioteke za rad sa JSON formatom.

JSON Schema definise kakva ce biti struktura JSON dokumenta.
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Proizvod",
  "description": "Šema za validaciju podataka o proizvodu",
  "type": "object",
  "properties": {
    "id_proizvoda": {
      "description": "Jedinstveni identifikator proizvoda",
      "type": "integer",
      "minimum": 1
    },
    "naziv_proizvoda": {
      "description": "Naziv proizvoda koji ne sme biti prazan",
      "type": "string",
      "minLength": 3
    },
    "jedinica_mere": {
      "description": "Jedinica mere (kom, l, kg, m)",
      "type": "string",
      "enum": ["kom", "l", "kg", "m"]
    },
    "cena": {
      "description": "Jedinična cena proizvoda",
      "type": "number",
      "minimum": 0
    }
  },
  "required": ["id_proizvoda", "naziv_proizvoda", "jedinica_mere"],
  "additionalProperties": false
}
```
U okviru nje dakle prvo definisemo sablon koji koristimo za semi a nakon toga i naziv i opis seme. Onda u okviru naziva kljuceva definisemo koji tip vrednosti oni uzimaju i na kraju definisemo i koja polja su obavezna da se pojave u nasem dokumentu.

JSONPath se koristi za pristupanje vrednostima u okviru JSON dokumenta:
1. $ - root path
2. @ - trenutni path
3. .. - relativna putanja
4. ?() - uslov koji mora ispostovati
5. \[] - pristup konkretnom elementu
6. 
Primer
```json
// pristupiti stavki cija vrednost je veca od 100
$..Stavka[@.Vrednost ?(Vrednost >= 100)]
```

Kreiranje tabele u okviru Oracle baze podataka
```plsql
CREATE TABLE otpremnicaJSON (podaci CLOB
CONSTRAINT check_json CHECK(podaci IS JSON));
```
Ako zelimo da dodamo JSON dokument o ovu tabelu radimo sledece:
```plsql
INSERT INTO otpremnicaJSON (podaci)
VALUES ('neki kreirani JSON dokument');
```

