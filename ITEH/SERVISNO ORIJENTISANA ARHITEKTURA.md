**Servis** predstavlja samostalnu, logicku jedinicu funkcionalnosti koja nam obavlja specifican zadatak i pruza to drugim delovima sistema putem jasno definisanog interfejsa. Radi po principu crne kutije, sto znaci da je korisniku servisa nepoznato kako se izvrsavaju aktivnosti unutar samog servisa.

**Servisno orijentisana arhitektura** - predstavlja kolekciju servisa koji komuniciraju ili koordiniraju neke aktivnosti.
### Karakteristike servisa

1. **Razotkrivost i dinamicka povezanost** - korisnik koji koristi servis otkriva koji servis da koristi na osnovu skupa kriterijuma u datom trenutku
2. **Samosadrzivost i modulatnost** - servisi mogu lako da se agregiraju u aplikaciju
3. **Slaba sprega** - poseduju dobro poznate zavisnosti
4. **Samoizlecenje** - servis ima sposobnost da se samostalno oporavi od gresaka, bez ukljucenje coveka u taj proces
5. **Mrezno-adresabilan interfejs**
6. **Transparentnost po lokaciji**
7. **Mogucnost sastavljanja**
## Veb servisi

Najcesce se realizuju kroz:
 1. **REST (Representational State Transfer) protokol** - koji je tipican za mikroservise
 2. **SOAP protokol** - ne toliko zastupljen danas, ali je vecina starih veb servisa realizovano preko ovog protokola, zasnovan je na XML-u
### Povezivanje servisa

**Orkestracija** predstavlja hijerarhijsko povezivanje veb servisa u jednu aplikacija. Orkestracijom upravlja koordinator.

Druga varijanta je **koreografija servisa** gde ne postoji centralni koordinator vec komponente znaju sta trebaju da rade i medjusobno komuniciraju.