# ASP .Net

**Índice**

* [Instalaremos las dependecias que hagan falta](#id0)
* [Conexion a la Base de datos](#id1)
* [ApplicationDbContext](#id2)
* [Models](#id3)

---

<div id='id0' />

## Instalaremos las dependecias que hagan falta

<img src="img/conexion_sqlserver_asp.net.png" /> 

### Dependencias necesarias 
"***Microsoft.SqlServer.Server***"

"***Microsoft.EntityFramework.Tools***"

"***Microsoft.EntityFrameworkCore***"

La descargaremos en el apartado ***Tools***
* ***Tools***
  * NuGet Package manager
    * Manager NuGet Packages for Solution...
* En el buscador introduciremos la dependencia que queremos instalar ***sqlserver***
* Intalaremos para la version que tenermos en el proyecto

---

<div id='id1' />

## Conexion a la Base de datos

***appsettings.json***

* Agregar el string de conexion con la DB

```json
"ConnectionStrings": {
    "connection_name":  "Server:[server_name];Database=ApiPeliculasNET8;User ID=[user_name];Password=[password];Trusted_Connection=true;TrustServerCertificate=true;MultipleActiveResultSets=true;"
}
```

String de conexin -> "Server=localhost\SQLEXPRESS;Database=master;Trusted_Connection=True;"

* Conigurar la cadena de conexion en ***Program.cs***

***Program.cs***
```c#
// Add services to the container.
builder.Services.AddDbContext<ApplicationDbContext>(opciones => 
opciones.UseSqlServer(builder.Configuration.GetConnectionString("ConexionSql")));
```
  
---

<div id='id2' />

## ApplicationDbContext

El ***archivo de contexto*** es quien se va a encargar de mappear todas las entidades a la base de datos

**nota:** Siempre se crea dentro de la clase Data

```c#
public class ApplicationDbContext: DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options): base(options){}

    // Aqui pasar todas las entidades (Modelos)
    public DbSet<Categoria> Categorias { get; set; }
}
```



<div id='id3' />

## Models

Anotaciones

* [Key] -> hace referencia a la anotacion del ID autoincremental
* [Required] -> Nos dice que el campo tiene que ser requerido

```c#
[Key]
public int Id { get; set; }
```