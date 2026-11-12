SLEDECE: [[GIT]]
# Sta je Docker?
Docker predstavlja skup proizvoda kao usluga; PaaS - Platform as a Service; koji koriste virtuelizaciju na nivou OS za isporuku softvera u paketima koji se nazivaju kontejneri.
Osnovna ideja ovog koncepta je da se omoguci da se aplikacije izvrsavaju konzistentno bez obzira na okruzenje u kom se pokrecu.
U kontejnerima mozemo da zapakujemo aplikacije, zajedno sa svim potrebnim bibliotekama i zavisnostima.

## Docker arhitektura
1. **Docker Daemon** - pozadinski proces koji upravlja svim kontejnerima na sistemu
2. **Docker CLI** - command line interface, kojim korisnici komuniciraju sa Dockerom
3. **Docker Client** - predstavlja sponu izmedju Docker CLI i Daemona, koji moze biti lokalni ili na nekom udaljenom serveru
4. **Docker Images** - Docker slike predstavljaju upakovane aplikacije sa svim bibliotekama i zavisnostima koje su potrebne. Slike koristimo kao temelj za kreiranje i pokretanje Docker kontejnera.
5. **Docker Containers** - predstavljaju instance Docker slika koje pokrecemo u izolovanom okruzenju.
6. **Docker Registry** - predstavlja skladiste za Docker slike, ono moze biti privatno za skladistenje internih slika ili na primer Docker Hub koji je javni registar.

## Uvod u docker slike
Koriste se za konzistentno funkcionisanje softvera na razlicitim platformama tokom razvoja, testiranja i produkcije. Takodje stede vreme jer se mogu samo koristiti postojece slike sa nekog od registara.

Da bi se kreirala slika potrebno je da se koristi **Dockerfile**, sto je skripta koja odredjuje baznu sliku, softverske pakete za instaliranje, datoteke za kopiranje i druge konfiguracione postavke.
Nakon sto se slika izgradi moze se skladistiti na nekom registru ili implementirati na nekom serveru ili cloud platformi kao sto je Azure ili AWS.

Primer Dockerfile-a:
```Dockerfile
FROM node:18
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
EXPOSE 80
CMD ["node", "server.js"]
```
*Komande koje se izvrsavaju prilikom kreiranja Docker slike:*
- FROM node:18 - definise osnovnu Docker sliku i postavlja je na node verziju 18
- WORKDIR /app - postavlja radni direktorijum na /app i sve naredne komande se izvrsavaju u okviru ovog direktorijuma
- COPY package.json . - kopira konfiguracioni fajl u kome se nalaze sve zavisnosti koje treba instalirati u radni direktorijum
- RUN npm install - pokrece se komanda npm install kojom se instaliraju sve potrebne zavisnosti
- COPY . . - kopira ceo sadrzaj direktorijuma u kome se nalazi Dockerfile u radni direktorijum
*Komande koje se izvrsavaju nakon pokretanja kontejnera:*
- EXPOSE 80 - kontejner koji je zasnovan na slici ce biti dostupan na portu 80
- CMD \["node", "server.js"] - pokrece komandu > node server.js, u okviru kontejnera

*Dodatne komande:*
- ARG - definisu se varijable koje se koriste prilikom kreiranja Docker slike i nakon toga su nedostupne
- ENV - varijable koje su dostupne u okruzenju Docker kontejnera; mozemo da ih dodamo i prilikom pokretanja kontejnera uz komandu `docker run -e <varijable>`
- VOLUME - definise volumene koje ce kontejner deliti sa drugim kontejnerima i spoljasnjim sistemom

## Uvod u docker kontejnere
Kontejneri su posebno korisni u sledecim scenarijima:
- **Razvoj i testiranje** - radi konzistentno na svakoj platformi
- **Implementacija** - obezbedjuju standardizovano i predvidivo okruzenje koje se moze lako ponoviti na vise servera
- **Mikroservisi** - aplikacije se dele na manje, nezavisne servise koji se pakuju u odvojene kontejnere
- **Kontinualna integracija i kontinualna implementacija (CI/CD)** - automatizacija procesa razvoja

# Docker slike
Slike mogu biti: **built-in** i **custome**.
## Built-in slike
Built-in slike su one slike koje se preuzimaju sa Docker Huba i one predstavljaju obicno slike operativnog sistema, programskog jezika itd.
## Custome slike
Custome slike su prilagodjene Docker slike kreirane od strane korisnika. One se definisu i grade pomocu Dockerfile-a.
**package.json** - predstavlja konfiguracioni fajl koji sadrzi sve zavisnosti i informacije o node.js projektu; sadrzi spisak npm (node package manager) pakera koje aplikacija koristi, kao i skripte koje se mogu izvrsiti u okviru projekta.

Da bismo izgradili Docker sliku iz Dockerfile-a koristimo komandu **build**:
`docker build -t IME_SLIKE:TAG PUTANJA_DO_DOCKERFILEA`
Tag se koristi za razlikovanje razlicitih verzija iste slike (wishlist-app:v1, wishlist-app:v2...).

Ukoliko bismo ponovo pokusali da kreiramo Docker sliku nakon sto smo to vec uradili, koristi se mehanizam kesiranja, tj. menjaju se samo koraci u kojima je nesto promenjeno.

Ukoliko zelimo da istazimo konkretne specifikacije neke slike koristimo **inspect**:
`docker inspect ID_SLIKE/IME_SLIKE`
Neke od bitnih informacija koje mozemo da dobijemo od ove komande su:
- **Id** - identifikator slike
- **RepoTags** - popis tagova slike
- **Created** - datum i vreme kada je slika kreirana
- **ExposedPorts** - koji port ce biti izlozen iz kontejnera kada se slika ubaci u njega
- **Env** - environment varijable
- **Cmd** - spisak komandi koje se izvrsavaju kada se slika ubaci u kontejner
- **WorkingDir** - radni direktorijum u slici
- **Os** - operativni sistem na kom se pokrece aplikacija
- **Size** - ukupna velicina slike u bajtovima
- **Layers** - slojevi koji cine sliku, svaka linija koda u Dockerfile-u je jedan sloj

Ukoliko zelimo da *dodamo ili promenimo tag* na nekoj Docker slici koristimo **tag**:
`docker tag ORIGINALNA_SLIKA[:TAG] NOVA_SLIKA[:TAG]`
Tag je opcion i ukoliko ga ne napisemo koristi se **latest**.

Ukoliko ne koristimo neku Docker sliku i *zelimo da je obrisemo* to radimo pomocu komande **rmi**.
`docker rmi NAZIV_SLIKE[:TAG]`

# Docker kontejneri
Naredba **run** se koristi za *pokretanje Docker kontejnera iz Docker slike*:
`docker run [OPCIJE] SLIKA[:TAG] [KOMANDA] [ARG...]`
gde je:
- **OPCIJE** - dodatne opcije koje konfigurisu ponasanje kontejnera (npr. -it otvara kontejner u interaktivnom rezimu)
- **KOMANDA** - komanda koja ce se izvrsiti unutar kontejnera (slicno kao CMD u Dockerfileu)
- **ARG...** - argumenti prosledjeni komandi unutar kontejnera

Naredna komanda pokrece kontejner na masini na PORT_HOSTA, na koji se mapira PORT_KONTEJNERA, definisan u EXPOSE delu Dockerfilea.
`docker run -p PORT_HOSTA:PORT_KONTEJNERA`

Ukoliko zelimo da vidimo sve trenutno pokrenute kontejnere koristimo:
`docker ps`
a ukoliko zelimo *listu svih kontejnera*:
`docker ps -a/--all`
Nakon pokretanja ove komande dobijamo listu pokrenutih kontejnera sa kolonama:
- **CONTAINER ID** - jedinstveni hash identifikator svakog kontejnera
- **IMAGE** - ime slike na osnovu koje je kreiran kontejner
- **COMMAND** - komanda koja je pokrenuta unutar kontejnera prilikom pokretanja
- **CREATED** - kolona koja pokazuje vreme koje je proslo od kreiranja kontejnera
- **STATUS** - prikazuje trenutni status kontejnera
	- **UP** - pokrenut
	- **EXITED** - ugasen
	- **PAUSED** - pauziran pomocu "docker pause" naredbe
	- **RESTARTING** - restartuje se
	- **REMOVING** - brise se
	- **CREATING** - kreira se
- **PORTS** - prikazuje mapiranje portova izmedju lokalnog sistema i kontejnera, npr. "0.0.0.0:8080 -> 80/tcp", sto znaci da je port 8080 na lokalnom sistemu mapiran na port 80 u kontejneru
- **NAMES** - jedinstveno ime kontejnera koje smo mu mi dodelili ili automatski ukoliko nismo

Ukoliko zelimo da *zaustavimo izvrsavanje procesa unutar kontejnera* koristimo naredbu **stop**:
`docker stop NAZIV_KONTEJNERA/ID_KONTEJNERA`

Ukoliko zelimo da *obrisemo neki kontejner* koristimo komandu **rm**:
`docker rm NAZIV_KONTEJNERA/ID_KONTEJNERA`
pri tome se ne moze obrisati pokrenut kontejner vec ga prvo moramo zaustaviti.

Ukoliko zelimo da *ponovo pokrenemo postojeci kontejner* koristimo naredbu **start**:
`docker start NAZIV_KONTEJNERA/ID_KONTEJNERA`
ako zelimo da prikazemo i izlaz kontejnera u terminalu nakon sto se pokrene koristimo flag **-a**.

Ukoliko zelimo da dobijemo jos *detaljnije informacije o kontejnerima* koristimo **inspect**:
`docker inspect [OPCIJE] NAZIV_KONTEJNERA/ID_KONTEJNERA`

Ukoliko je potrebno da *restartujemo kontejner* to radimo pomocu komande:
`docker restart [OPCIJE] NAZIV_KONTEJNERA/ID_KONTEJNERA`

Da bismo *prikazali izlaze generisane u toku izvrsavanja procesa* unutar Docker kontejnera mozemo da koristimo komandu **logs**:
`docker logs [OPCIJE] NAZIV_KONTEJNERA/ID_KONTEJNERA`
Ovo nam je korisno ukoliko zelimo da pratimo rad aplikacije i pronadjemo greske.

Ukoliko zelimo da *prikljucimo terminal na pokrenut Docker kontejner* u kom cemo da unosimo komande koje ce se izvrsavati u okviru datog kontejnera koristimo komandu **attach**:
`docker attach [OPCIJE] NAZIV_KONTEJNERA/ID_KONTEJNERA`

Pokretanje privremenog pozadinskog kontejnera:
`docker run --rm -d -p PORT_HOSTA:PORT_KONTEJNERA --name NAZIV_KONTEJNERA NAZIV_SLIKE`
**--rm** - kontejner se brise nakon stopiranja
**-d** - kontejner se pokrece u pozadini sto omogucava korisniku da nastavi sa koriscenjem terminala
**--name** - dodeljujemo ime kontejneru kako ne bismo morali samo ID da koristimo

# Docker hub
Docker hub je *usluga koju pruza Docker za pronalazenje i deljenje slika*. On je najveci svetski repozitorijum i platforma koja podrzava razne izvore sadrzaja. Osim slika, podrzava i projekte otvorenog koda i nezavisne dobavljace softvera koji grade i distribuiraju svoj kod u kontejnerima.

Na vrhu sajta se nalazi polje za **pretrazivanje** kao i 3 kartice:
- **Explore** - blogove koje mozete procitati sa resenjima koje su drugi DevOps programeri radili, najpopularnije slike, nove ekstenzije, baze...
- **Repositories** - predstavlja spisak vasih slika koje ste postavili na Docker Hub
- **Organizations** - predstavlja spisak projekata na kojima radite u timu

# Docker volumeni
Docker volumen je *mehanizam za cuvanje i deljenje podataka* izmedju Docker kontejnera i maticnog sistema. Volumeni su posebne vrste direktorijuma i datoteka kojima Docker upravlja i izolovane su od ostatka datotecnog sistema kontejnera.

Postoji nekoliko vrsta volumena:
1. **Imenovani volumeni** - volumeni koji imaju ime. Docker ih kreira i upravlja njima i perzistentni su, tj. ne brisu se nakon gasenja kontejnera.
2. **Anonimni volumeni** - volumeni koji nemaju ime. Docker ih kreira kada se kontejner pokrene, a unistavaju se kad se zaustavi.
3. **Bind mounts** - predstavljaju nacin da se postojeci direktorijum s maticne masine montira u kontejner. Ovo omogucava da delite podatke izmedju kontejnera i maticne masine.

Kako odabrati pravi tip volumena:
- **Imenovani** - kada je potrebna perzistentnost podataka i saradnja vise kontejnera
- **Neimenovani** - za kratkotrajno skladistenje podataka, bez potrebe za deljenjem
- **Bind mounts** - za deljenje specificnih podataka sa maticne masine sa kontejnerima, uz paznju na bezbednost

Kako bismo prikazali sve trenutne volumene koristimo komandu:
`docker volume ls`

## Anonimni (neimenovani) volumeni
Da bismo kreirali neimenovani volumen, u okviru Dockerfile-a dodajemo liniju sa komandom VOLUME gde unosimo putanju na kojoj zelimo da se kreira nas neimenovani volumen:
```dockerfile
VOLUME ["/app/message"]
```
Ovo omogucava da svaki put kada pokrenemo kontejner kreiramo novi neimenovani volumen.

Ukoliko zelimo da *obrisemo neimenovani volumen* to mozemo da uradimo pomocu **rm** komande tako sto kao naziv volumena koristimo prvih par karaktera generisanog hash koda:
`docker volume rm 2ae`

## Imenovani volumeni
Imenovane volumene ne definisemo u Dockerfile-u, vec prilikom pokretanja kontejnera:
`docker run -v IME_VOLUMENA:PUTANJA_U_KONTEJNERU`
Putanja koju unosimo predstavlja putanju u kojoj zelimo da ovaj volumen opstane.

## Bind mount
Ukoliko zelimo da neke *promene na maticnoj masini budu odmah vidljive i u kontejneru* mozemo koristiti bind mount volumene.
Bind mount volumene definisemo slicno kao i imenovane, prilikom pokretanja kontejnera:
`docker run -v "APSOLUTNA_PUTANJA_MATICNE_MASINE:PUTANJA_U_KONTEJNERU"`

# Docker network
U Dockeru, **networking** igra kljucnu ulogu u omogucavanju *komunikacije izmedju kontejnera, kao i izmedju kontejnera i drugih servisa i uredjaja*.

Komunikacija se omogucava na sledece nacine:
1. **Komunikacija kontejnera sa www** -> javni API servis
2. **Komunikacija kontejnera sa lokalnim racunarom** -> baza podataka (MongoDB)
3. **Komunikacija kontejnera sa kontejnerom** -> npr. aplikacija komunicira sa SQL bazom podataka

Tipovi Docker mreznih drajvera:
1. **bridge** - podrazumevani drajver; koristi se za omogucavanje komunikacije izmedju kontejnera na istoj host masini kao i sa spoljasnjim internetom.
2. **host** - u ovom modu kontejneri dele mrezu s host masinom; koriste istu IP adresu kao host masina sto eliminise izolaciju izmedju kontejnera i host masine
3. **none** - kontejneri u ovom modu su potpuno izolovani od svega ostalog
4. **overlay** - povezuju vise Docker Daemona zajedno, sto omogucava kontejnerima da komuniciraju preko vise host masina
5. **ipvlan** - ovo omogucava kompletnu kontrolu nad IPv4 i IPv6 adresiranjem kontejnera, tj. direktno dodeljivanje IP adresa kontejnerima
6. **macvlan** - omogucava dodeljivanje MAC adresa kontejnerima

*Mreze u Docker-u se jako jednostavno kreiraju* preko terminala:
`docker network create`

Ukoliko zelimo da prikazemo *listu svih mreza* koristimo:
`docker network ls`

Ukoliko zelimo da povezemo kontejner sa nekom mrezom to radimo na dva nacina:
1. `docker run --network NAZIV_MREZE`
2. `docker network connect NAZIV_MREZE NAZIV_KONTEJNERA`

# Docker compose
Da ne bismo morali da stalno unosimo komande u terminalu za upravljanje kontejnerima, volumenima itd. mozemo da napisemo sve to u okviru jednog **YAML fajla**.
YAML (YAML ain't a markup language) je format koji se cesto koristi za konfiguracione fajlove. Docker compose koristi YAML fajlove za *definisanje servisa, mreza, volumena i drugih resursa potrebnih za pokretanje vise Docker kontejnera kao jedne aplikacije*.
Podrazumevana putanja do docker compose fajla je root direktorijum, a preferirani naziv je **compose.yaml**.

U okviru compose fajla se nalaze razna polja:
1. **Name** - predstavlja naziv projekta, koji ako nije naveden compose automatski preuzima na osnovu root direktorijuma
2. **Services** - definisu se servisi koji ce biti pokrenuti kao kontejneri, naziv servisa (npr. server, backend, frontend) ce biti naziv kontejnera
```yaml
name: "app"
services:
	mysql:
	backend:
	frontend:
```

YAML fajl je struktuiran po principu "kljuc: vrednost". S tim sto je nakon : obavezan razmak i ukoliko zelimo da za jedan kljuc vezemo listu vrednosti to mozemo postici sa crticama ispred vrednosti.

Docker compose fajl pokrecemo i zaustavljamo sa dve jednostavne komande:
`docker compose up`
`docker compose down`

## MySql servis
```yaml
mysql: 
	image: mysql:latest 
	container_name: mysql-server 
	environment: 
		MYSQL_ROOT_PASSWORD: mypassword 
		MYSQL_DATABASE: swfavorites 
		MYSQL_USER: novi 
		MYSQL_PASSWORD: novi 
		MYSQL_AUTHENTICATION_PLUGIN: caching_sha2_password 
	networks: 
		- app-network 
	volumes: 
		- mysql-data:/var/lib/mysql 
		- ./db/init.sql:/docker-entrypoint-initdb.d/init.sql 
		- ./db/my.cnf:/etc/mysql/

# na dnu fajla definisemo mreze i imenovane volumene, sa indentacijom 0
networks: 
	app-network: 
		driver: bridge

volumes:
	mysql-data:
```

Postoji i druga mogucnost za definisanje environment varijabli, a to je kreiranje .env fajla, npr. u ovom slucaju mysql.env. Onda bismo u kodu ceo kljuc environment sa vrednostima zamenili.
```yaml
env_file:
	- ./env/mysql.env
```

## Backend servis
```yaml
backend: 
	build: ./backend 
	container_name: fav-backend 
	ports: 
		- "3000:3000" 
	env_file: 
		- ./env/backend.env 
	networks: 
		- app-network 
	volumes: 
		- ./backend:/app 
		- /app/node_modules 
	depends_on: 
		- mysql #naziv servisa ne naziv kontejnera mysql-server
```
U prethodnom servisu smo imali kljuc **image** i to je podrazumevalo da ce da se povuce image sa Docker Huba *mysql:latest*. U ovom servisu imamo kljuc **build** koji podrazumeva da se koristi Dockerfile definisan u direktorijumu /backend na lokalnoj masini.

Trenutno ako bismo pokrenuli ovaj docker compose fajl, pokrenuo bi se mysql kontejner, ali se ne bi pokrenuo backend kontejner zato sto zavisi od mysql kontejnera, a pokusali su istovremeno da se pokrenu.
Medjutim, mozemo samo naknadno rucno da pokrenemo backend kontejner na osnovu slike:
`docker start fav-backend`

Da ne bismo morali ovo da radimo svaki put kad zelimo da pokrenemo aplikaciju mozemo da dodamo **healthcheck** u okviru mysql servisa, na samom kraju, gde proveravamo da li je mysql servis "ziv".
```yaml
healthcheck: 
	test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "novi", "--password=novi"]
	interval: 10s 
	timeout: 10s 
	retries: 6
	
...

depends_on:
	mysql:
		condition: service_healthy
```

Ovde imamo nekoliko stvari da objasnimo:
- **test** - definise koja komanda ce da se izvrsi u okviru kontejnera
- **interval** - definise vreme koje komanda ima na raspolaganju da se izvrsi, nakon isteka tog vremena smatra se neuspesnom
- **timeout** - definise razmak do ponovnog pokretanja komande
- **retries** - definise broj pokusaja za uspesno izvrsavanje komande

## Frontend servis
```yaml
frontend: 
	build: ./frontend 
	container_name: fav-frontend 
	ports: 
		- "3001:3000" 
	networks: 
		- app-network 
	volumes: 
		- ./frontend:/app 
		- /app/node_modules 
	depends_on: 
		- backend
```

## Dodatne komande u yaml fajlu
- **restart** - definise kako ce se servis restartovati u slucaju greske ili rucnog zaustavljanja
- **entrypoint** - definise komandu ili skriptu koja ce se izvrsiti prilikom pokretanja kontejnera
- **command** - definise dodatne argumente koji se prosledjuju kao komanda prilikom pokretanja kontejnera
- **labels** - dodaje dodatne oznake kontejneru
- **network_mode** - postavlja nacin rada mreze za kontejner
- **detach** - da bi se kontejner pokrenuo u pozadini mozemo staviti detach: true