## Routers

***app.routes.ts***

```ts
import { Routes } from '@angular/router'
import { ProductosViewComponent } from './views/productos-view/productos-view.component'
import { LandingPageViewComponent } from './views/landing-page-view/landing-page-view.component'

export const routes: Routes = [
  {
    path: '',
    component: LandingPageViewComponent
  },
  {
    path: 'productos',
    component: ProductosViewComponent
  }
]
```

***NavbarComponent***

```ts
import { Component } from '@angular/core'
import { RouterLink, RouterLinkActive } from '@angular/router'

@Component({
  selector: 'app-navbar',
  imports: [RouterLink, RouterLinkActive],
  templateUrl: './navbar.component.html',
  // template: `
  //   <nav>
  //     <a
  //       routerLink="/"
  //       routerLinkActive="active"
  //       [routerLinkActiveOptions]="{ exact: true }"
  //       >Contador</a
  //     >
  //     <a [routerLink]="['/hero']" [routerLinkActive]="'active'">Hero</a>
  //   </nav>
  // `
})
export class NavbarComponent {}

// html
<nav>
  <a
    routerLink="/"
    routerLinkActive="active"
    [routerLinkActiveOptions]="{ exact: true }">Contador</a>
  <a [routerLink]="['/hero']" [routerLinkActive]="'active'">Hero</a>
</nav>
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