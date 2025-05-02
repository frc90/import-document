Para agregar una conexion a la ***DB*** vamos al arquivo ***appsettings.json***, dentro esta el ***appsettings.Development.json*** ahi creamos una propiedad llamada ***ConnectionStrings*** y dentro ***ConexionSql*** como `key` y el `value` los datos de la conexion


***appsettings.Development.json***
```json
{
    "ConnectionStrings": {
        "DefaultConnection": "Server=DESKTOP-BFAMBKT;Database=ApiPeliculasNET8;User ID=sa;Password=qwerty;Trusted_Connection=true;TrustServerCertificate=true;MultipleActiveResultSets=true",
    "Logging": {
        "LogLevel": {
            "Default": "Information",
            "Microsoft.AspNetCore": "Warning"
        }
    },
    "OrigenesPermitidos": "http://localhost:4200" 
     // url como arreglos ( "url1, url2, url3" ) 
}
```
---

Detalles de los datos de la conexion, todo va dentro de un `string`  
* Server=DESKTOP-BFAMBKT;        // Nombre del servidor donde se encuentra la DB
  
* Database=[DB_NAME];            // Nombre de la base de datos
* Integrated Security=true       // Solo si se quiere utilizar las credenciales de windows, si no omitir
* User ID=sa;                    // Usuario para acceder a la DB
* Password=qwerty;               // Password para acceder a la DB
* Trusted_Connection=true;       //
* TrustServerCertificate=true;   // Para no configrar certificado de seguridad ***en desarrollo***
* MultipleActiveResultSets=true  //

---

Configurar el `ApplicationDbContext` como un servicio, abrimos la clase ***Program.cs***

***Program.cs***

```cs
builder.Services.AddDbContext<ApplicationDbContext>(opciones => opciones.UseSqlServer("name= DefaultConnection"));
```