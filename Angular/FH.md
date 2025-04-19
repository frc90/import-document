# Angular

**Índice**

0. [Estructura de angular](#id0)
1. [Signal](#id1)
2. [Routers](#id2)

---
<div id='id0' />

## Estructura de angular

***Componente***

<img src="img/1. componentes.png" />

***Rutas***

<img src="img/2. rutas.png" />

***Directivas***

<img src="img/3. directivas.png" />

***Servicios***

<img src="img/4. services.png" />

***Modulos***

<img src="img/5. modulos.png" />

***Pipes***

<img src="img/6. pipes.png" />

---

<div id='id1' />

## Signal

Los ***signal*** son una forma de manejar el estado en le componente

```ts
import { Component, computed, signal } from '@angular/core'

counterSignal:number = signal(10)

this.counterSignal.update(current => current + value)
this.counterSignal.set(0)


heroDescription = computed(() => {
    return `${this.name()} ${this.age()}`
  })

  capitalizeName = computed(() => {
    return this.name().toUpperCase()
  })
```
---

<div id='id2' />

## Routers

***NavbarComponent***

```ts
import { Component } from '@angular/core'
import { RouterLink, RouterLinkActive } from '@angular/router'

@Component({
  selector: 'app-navbar',
  imports: [RouterLink, RouterLinkActive],
  // templateUrl: './navbar.component.html',
  template: `
    <nav>
      <a
        routerLink="/"
        routerLinkActive="active"
        [routerLinkActiveOptions]="{ exact: true }"
        >Contador</a
      >
      <a [routerLink]="['/hero']" [routerLinkActive]="'active'">Hero</a>
    </nav>
  `
})
export class NavbarComponent {}
```

***AppComponent***

```ts
import { Component } from '@angular/core'
import { RouterOutlet } from '@angular/router'
import { NavbarComponent } from './components/shared/navbar/navbar.component'

@Component({
  selector: 'app-root',
  imports: [RouterOutlet, NavbarComponent],
  template: `
    <app-navbar />
    <router-outlet />
  `
})
export class AppComponent {
  title = 'bases'
}
```