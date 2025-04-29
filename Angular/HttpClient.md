Para utilizar el HttpClient que nos provee angular, hay que modicar el archivo ***app.config.ts*** para que el framework sepa que lo vamos a usar.

***app.config.ts***

```ts
...

export const appConfig: ApplicationConfig = {
  providers: [
    provideZoneChangeDetection({ eventCoalescing: true }),
    provideRouter(routes),
    provideHttpClient(withFetch()) // HttpClient
  ]
};
```

Una vez configurado el Environment se hace la peticion asi:

- Se inyecta por dependencia el ***HttpClient***
- Se extrae en una variable la ***urlBase***
- Se crea la funcion para obtener los datos del ***endpoint***
  
```ts
import { HttpClient } from '@angular/common/http'
import { inject, Injectable } from '@angular/core'
import { environment } from '../../environments/environment'

@Injectable({
  providedIn: 'root'
})
export class WeartherforecastService {
  constructor () {}

  private http = inject(HttpClient)
  private urlBase = environment.apiUrl + '/weatherforecast'

  public obtenerClima () {
    return this.http.get<any>(this.urlBase)
  }
}
```

***appComponent.ts***

```ts
... imports ...
export class AppComponent {
  weatherforecastService = inject(WeartherforecastService)
  climas: any[] = []

  constructor () {
    this.weatherforecastService.obtenerClima().subscribe(datos => {
      this.climas = datos
    })
  }
}

// html
@for (clima of climas; track $index) {
<div>
  <div>Fecha: {{clima.date}}</div>
  <div>Temperatura (C): {{clima.temperatureC}}</div>
  <div>Resumen: {{clima.summary}}</div>
</div>
<hr>
}
```