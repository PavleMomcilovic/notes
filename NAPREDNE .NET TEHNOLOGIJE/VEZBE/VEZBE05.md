Kada koristimo WebAPI koji je baziran na kontrolerima mozemo na vise nacina da definisemo nase endpointe.
```csharp
namespace RestWebApi.Controllers {
    [Route("api/[controller]/v1")]
    [ApiController]
    public class ProductsController : ControllerBase {
        [HttpGet("search")]
        public List<Product> GetProducts() {
            return new List<Product> { new Product { Id = 1, Name = "P1" } };
        }
    }
}
```
Da objasnim malo ovaj primer. Atribut Route koji se nalazi na samom pocetku koda za kontroler predstavlja default rutu na koji se salju svi zahtevi. Ovo \[controller] se menja imenom samog kontrolera, sto je u ovom slucaju Products. 

Ako zelimo da promenimo rutu nekog HTTP zahteva, to mozemo da definisemo svojstvom ispred neke odredjene metode, u ovom slucaju kad napisemo samo atribut "search", kreiramo novu rutu za GET zahtev koja je api/products/v1/search. Kada bismo zeleli celu rutu da pregazimo napisali bismo "/search" i onda bi to bila nova ruta za zahtev.