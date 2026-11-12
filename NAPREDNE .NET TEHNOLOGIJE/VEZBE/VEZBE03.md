## MinimalAPI

Builder nam sluzi da odredimo strukturu nase aplikacije, tj. koje ce sve servise koristiti i kofiguracije, dok app definise kako cemo implementirati tu strukturu koju nam je builder zadao. U nastavku imamo primer kreiranja Veb aplikacije.
```csharp
var builder = WebApplication.CreateBuilder(args);
// builder.Services... – servisi (DI)
var app = builder.Build();
// app.Use... – middleware (pipeline)
// app.Map... – endpoint-i (rutiranje)
app.MapGet("/", () => "Delivery API radi!");
app.Run(); // Kestrel pocinje da slusa
```

## HTTP zahtev

![[Putanja HTTP zahteva.png]]
_Slika: izgled putanje HTTP zahteva_

Za svaki zahtev koji klijent posalje Kestrel kreira novi HttpContext, tj. objekat koji nosi zahtev i odgovor. Nakon toga taj objekat prolazi kroz middleware i zavrsava na rutiranju, koje je i samo middleware, koje gleda HTTP metodu i putanju i bira odgovarajuci endpoint. Ukoliko se endpoint ne nadje -> 404 Not Found

Struktura HTTP zahteva je sledeca:
```http
POST /api/products HTTP/1.1       # metoda, putanja, verzija
Host: localhost:5246              # zaglavlja
Content-Type: application/json
{"name":"Cokolada","price":250}   # telo (GET metoda ga nema obicno)
```
Dok je struktura HTTP odgovora sledeca:
```http
HTTP/1.1 201 Created                    # verzija, statusni kod, opis
Location: /api/products/5               # zaglavlja
Content-Type: application/json
{"id":5,"name":"Cokolada","price":250}  # telo
```

Medju statusnim kodovima razlikujemo:
1. 2xx - uspeh
2. 3xx - preusmerenje
3. 4xx - greska klijenta
4. 5xx - greska servera
## Rutiranje

Ukoliko zelimo da posaljemo neki HTTP zahtev na server, to mozemo uciniti preko nekoliko metoda u ASP.NET-u.
```csharp
app.MapGet("/api/products", ...);             // vraca listu proizvoda
app.MapGet("/api/products/{id:int}", ...);    // vraca jedan proizvod
app.MapPost("/api/products", ...);            // dodaje novi proizvod
app.MapPut("/api/products/{id:int}", ...);    // menja postojeci proizvod
app.MapDelete("/api/products/{id:int}", ...); // brise postojeci proizvod
```
Endpoint predstavlja HTTP metodu + sablon putanje.

Ono sto se desava u ovom kodu jeste nesto sto se zove Model Binding, tj. ASP.NET primecuje da lambda funkcija definisana u GET zahtevu prihvata parametar i povezuje ga sa id-jem koji se nalazi u URL-u.
```csharp
app.MapGet("/api/products/{id:int}", (int id) => {
	var product = products.FirstOrDefault(p => p.Id == id);
	return product is null ? Results.NotFound() : Results.Ok(product);
});
```

Ukoliko definisemo ulazni parametar lambda funkcije kao nullable tip podatka, onda on predstavlja opcioni parametar. To znaci da ce HTTP zahtev proci iako ne prosledimo taj parametar. U URL-u to izgleda kao na primer :
`/api/products/search?naziv=a`

Vrednosti parametara za HTTP zahtev dolaze iz raznih izvora:
1. Ruta - ime je u sablonu rute; {id:int}
2. Query string - prost tip van rute; ?naziv=a
3. Telo (JSON) - slozen tip
4. Posebni - HttpContext, CancellationToken

## Obrada gresaka

Ukoliko zelimo da se prikaze neki nas odredjen statusni kod kada se pronadje endpoint ili ne pronadje to mozemo da radimo preko klase Results, tako sto odredimo da nam je povratna vrednost lambda funkcije definisane u okviru nekog HTTP zahteva, npr. Results.Ok(poruka/objekat), ako je proslo ili Results.NotFound(poruka/objekat).

Ako zelimo detaljnije da opisemo gresku mozemo da koristimo, nasu prilagodjenu gresku u JSON formatu preko sledece naredbe:
```csharp
return Results.Problem(
	title: "Neispravna cena",
	detail: "Cena mora biti veca od 0.",
	statusCode: 400);
```

Ako zelimo da nam se ispise detaljniji opis greske koja je nastala prilikom HTTP zahteva, a ne samo statusni kod greske koristimo ove naredbe na pocetku pipeline-a:
```csharp
builder.Services.AddProblemDetails();
app.UseStatusCodePages();
```

Ukoliko zelimo da imamo obradu gresaka samo na serverskoj strani bez da klijenti vide detaljnu poruku o gresci to mozemo raditi na sledeci nacin:
```csharp
app.MapGet("/api/error", () => {
	throw new InvalidOperationException("Nesto je puklo na serveru.");
});
```
Ovim se osiguravamo da greska ne bude u HTTP telu vec samo u konzoli na serveru.