Vecina microsoftovih projekata je radjenje C# u Visual Studio Code-u, sto je gas ako budem prelazio na linuks, tako da cu ovde da izlistam komande koje su mi potrebne ako to budem radio.

## Komande
Da bih proverio verzije .NET-a koje imam na racunaru radim
`dotnet --list-sdks`

Ovom komandom kreiram novi WebAPI:
`dotnet new webapi -controllers -f net10.0`

Da bih buildovao i pokrenuo aplikaciju radim:
`dotnet build` `dotnet run`

Dodavanje dependency-ja:
`dotnet add package <naziv>`
Brisanje:
`dotnet remove package <naziv>`
Izlistavanje svih:
`dotnet list package`

## Dependencies
Ukoliko bih zeleo da instaliram neke dependency-je potrebno je da odem na [www.nuget.org](https://www.nuget.org) i odatle kopiram komande za preuzimanje u terminal u okviru VS Code-a.