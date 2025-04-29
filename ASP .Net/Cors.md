Dentro de ***appsettings.json*** en el archivo de ***appsettings.Development.json***, crear una variable con el valor de la ruta del frontend o las urls que se quieran dar acceso a travez de los ***CORS***

***appsettings.Development.json***
```json
{
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


***Program.cs***

```cs
var builder = WebApplication.CreateBuilder(args);

// Add services to the container.

builder.Services.AddControllers();
// Learn more about configuring OpenAPI at https://aka.ms/aspnet/openapi
builder.Services.AddOpenApi();

// Agregar cors para las rutas que se quieran permitir
var origenesPermitidos = builder.Configuration.GetValue<String>("OrigenesPermitidos").Split(",");

builder.Services.AddCors(opciones =>
{
    opciones.AddDefaultPolicy(politica =>
    {
        politica.WithOrigins(origenesPermitidos).AllowAnyHeader().AllowAnyMethod();
    });
});

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.MapOpenApi();
}

app.UseHttpsRedirection();


// agregarlo como middleware  
app.UseCors();

app.UseAuthorization();

app.MapControllers();

app.Run();

```