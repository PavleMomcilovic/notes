## Delegati

Za potrebe ovog predmeta treba se podsetiti sta su delegati i koje vrste delagata postoje. Delegati predstavljaju promenljive tipa metoda koje mogu da imaju parametre i povratnu vrednost.
Specijalne vrste delegata su:
```csharp
// Func prima parametre i vraca vrednost (poslednji argument je tip povratne vrednosti)
Func<int, int, int> add = (a, b) => a + b;
Console.WriteLine(add(2, 3)); // Ispis: 5

// Action prima parametre i nema povratnu vrednost
Action<string> print = text => Console.WriteLine($"> {text}");
print("poruka"); // Ispis: > poruka
```
## Lambda funkcije

Lambda funkcije predstavljaju funkcije napisane u obliku
```csharp
(argumenti...) => telo_funckije;    // jednolinijska lambda f-ja
(argumenti...) => { ... };          // viselinijska lambda f-ja
```

**Closure** predstavlja koncept gde lambda funkcija "hvata" neku promenljivu van svog opsega i menja joj vrednost. Na primer:
```csharp
var actions = new List<Action>();
for (int i = 0; i < 3; i++) {
	actions.Add(() => Console.WriteLine(i));
}

foreach (var act in actions) {
	act();
}
// Ispisuje se 3, 3, 3, jer je petlja zavrsila izvrsavanje i vrednost za i je sada jednaka 3.
```
## Task/Thread pool

Task predstavlja posao koji ce se izvrsiti u buducnosti. Dakle, ne mora nijedna druga naredba da ceka na njeno izvrsenje vec mozes da se odvija paralelno sa Taskom.

Kada se kreira Task i cim se zapocne Task.Run, blok odlazi u ThreadPool i izvrsava se na nekoj drugoj niti paralelno sa mainom i ostalim nitima.
```csharp
Task<int> sumTask = Task.Run(() => {
	// celo telo radi se prebacuje na nit iz ThreadPool-a
});
```

Sta je uopste ThreadPool? On predstavlja kolekciju spremnih niti koje mozemo da "angazujemo" za obradu nasih Taskova i kada zavrse sa obradom se vracaju nazad u isti taj ThreadPool. On postoji zato sto je kreiranje novih niti skupo i zahteva odredjeno vreme, a prilikom pokretanja hiljade Taskova to bi trajalo previse vremena.

TaskRun, obrada HTTP zahteva i sve sto ide nakon await ide na niti iz ThreadPool-a.

Ako zelimo da definisemo sta ce da se desi kad nit zavrsi sa cekanjem onda koristimo ContinueWIth:
```csharp
Log("шаљем захтев бази (траје 1 s)");
// Task.Delay НЕ заузима нит –то је само тајмер
Task<string> loadTask = Task.Delay(1000)
	.ContinueWith(_ => {
		Log("подаци су стигли");
		return "Marko, Jelena, Ana";
	});
Task printTask = loadTask
	.ContinueWith(t => Log($"исписујем: {t.Result}"));
Log("Main НЕ чека –нит је слободна за други посао");
```z
### async/await

Ukoliko se kljucna rec **async** nadje u potpisu metode, to dozvoljava pozivanje kljucne reci **await** u okviru tela te metode.
Kljucna rec **await** oznacava da cekamo na Task bez blokiranja niti
```csharp
static async Task<int> SumAsync(int id) {
	await Task.Delay(1000); // чекамо, нит је слободна
	int a = Random.Shared.Next(1, 100);
	int b = Random.Shared.Next(1, 100);
	return a + b; // враћамо int, не Task<int>
}
int sum = await SumAsync(1); // await „распакује“ Task<int>
```

Alternativa za async/await je zapravo da u okviru celog returna pozovemo ContinueWith, medjutim async/await je mnogo citljivije.

Asinhrone Taskove mozemo pozvati na dva razlicita nacina:
```csharp
// 1. nacin - metode se izvrsavaju jedna za drugom (ako je delay 1s, ukupno je 2s)
await Metoda1Async();
await Metoda2Async();

//2. nacin - metode se izvrsavaju paralelno (traje koliko i najduza metoda, tj. 1s)
await Task.WhenAll(Metoda1Async, Metoda2Async);
```

