Sitaksa za insert:
```mongodb
<field1>: <value1>...
ili Dokument: {<field1>: <value1>, <field2>: <value2>}
```

Vecinu komandi radimo preko basha, tj. mongosh-a. Mongo predstavlja bazu koja koristi dokumenta za cuvanje podataka i u njoj imamo kolekcije, koje mozemo da uporedimo sa klasama u SQL-u.
```mongodb
vezbe> db.TipProizvoda.insertOne({_id:1, Naziv:"Laptop"})
```

Izmedju viticastih zagrada se nalaze dokumenti, tj objekti. U sustini MongoDB koristi JSON notaciju.