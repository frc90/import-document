***(Ctro + .)*** para importar librerias similar a ***(Ctro + [space])***

Se crea una clase con el nombre por norma ***ApplicationDbContext***.

***ApplicationDbContext***

```cs
using Microsoft.EntityFrameworkCore;

namespace _01_WebApi
{
    public class ApplicationDbContext : DbContext
    {
        public ApplicationDbContext(DbContextOptions options) : base(options)
        {
        }

        protected ApplicationDbContext()
        {
        }
    }
}
```