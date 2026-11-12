## Kreiranje baze

Da bismo napravili bazu potrebno je da kreiramo infrastrukturu, tj.
```csharp
namespace Infrastructure {
    public class ProductsContext : DbContext {
        public DbSet<Product> Products { get; set; }

        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder) {
            optionsBuilder.UseSqlServer(@"Data Source=(localdb)\mssqllocaldb; Initial Catalog=NNT; Integrated Security=True;");
        }

        protected override void OnModelCreating(ModelBuilder modelBuilder) {
            modelBuilder.Entity<ProductCategory>().HasKey(pc => pc.CategoryCode);
			modelBuilder.Entity<ProductCategory>().Property(pc => pc.CategoryCode).ValueGeneratedNever();
        }
    }
}
```

Takodje da bi se uopste napravila baza mi moramo da imamo kreirane domenske klase, npr.
```csharp
namespace Domain {
    public class Product {
        public int Id { get; set; }
        public string Name { get; set; }
        public decimal Price { get; set; }
        public DateTime Created { get; set; }
        public ProductCategory Category { get; set; }
    }

    public class ProductCategory {
        [Key]
        public int CategoryCode { get; set; }
        public string Name { get; set; }
    }
}
```

Da bismo kreirali novu migraciju, tj. da bismo uskladili bazu podataka (SQLServer) sa kodom, unosimo u terminal komandu:
`dotnet ef migrations add PorukaKoju zelimo`
ili u Package Manager konzoli:
`> Add-Migration NazivMigracije`

Da bismo nakon toga azurirali bazu podataka unosimo komandu:
`> dotnet ef database update`
ili u Package Manager konzoli:
`> Update-Database`

Ako zelimo da ponistimo poslednju migraciju mozemo da napisemo:
`> dotnet ef migrations remove`
a da bismo prikazali sve migracije koristimo:
`> dotnet ef migrations list`

## Upravljanje bazom

Ukoliko zelimo da dodajemo podatke u nasu bazu, mozemo to da uradimo preko neke klase npr.
```csharp
using Domain;
using Infrastructure;

ProductsContext context = new ProductsContext();

context.Add(new ProductCategory {
    CategoryCode = 111,
    Name = "PC111"
});

context.Add(new ProductCategory {
    CategoryCode = 222,
    Name = "PC222"
});

context.Add(new ProductCategory {
    CategoryCode = 333,
    Name = "PC333"
});

context.Add(new ProductCategory {
    CategoryCode = 444,
    Name = "PC444"
});

context.Add(new ProductCategory {
    CategoryCode = 555,
    Name = "PC555"
});

context.SaveChanges();      // sluzi da bismo COMMIT-ovali transakciju
```

Ako zelimo da procitamo podatke iz nase baze to mozemo na sledeci nacin:
```csharp
var categories = context.Set<ProductCategory>().Where(c => c.CategoryCode == 333).ToList();
```

Deo koda .Where(c => c.CategoryCode == 333) predstavlja LINQ naredbu.