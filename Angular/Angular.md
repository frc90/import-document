# Angular

**Índice**

1. [Instalación de angular](#id1)
2. [Comandos de angular](#id2)
3. [Directivas de angular(anterior)](#id3)
   1. [ngFor](#id3.1)
   2. [ngIf](#id3.2)
   3. [ngIfElse](#id3.3)
   4. [ng-template #name](#id3.4)
   5. [button (click)](#id3.5)
4. [Directivas de angular(nuevas)](#id4)
5. [Estructura de un componente](#id5)
6. [OnInit](#id6)

## Descripcion

Angular es un framework que trabaja con el lenguaje JS para hacer aplicaciones en el lado del frontend de la web.

<div id='id1' />

## 1. Instalación de angular

- Ejecutamos el sisguiente comando

```cmd
npm install -g @angular/cli
```

- Una vez instalado el paquete ejecutamos este comando para cear un proyecto en angular

```cmd
ng new <project-name>
```

**_NOTA:_** Si quieres instalar una version especifica de angular ejecuta este comando

```cmd
npx @angular/cli@17 new my-project
```

<div id='id2' />

## 2. Comandos de angular

- Para correr angular ejecutamos

```cmd
ng serve
```

- Para generar un nuevo componente

```cmd
ng generate component component-name
```

- Tambien se puede usar

```cmd
ng generate directive|pipe|service|class|guard|interface|enum|module
```

<div id='id3' />

## 3. Directivas de angular(anterior)

<div id='id3.1' />

## 3.1 ngFor

```ts
<tbody>
    <tr *ngFor="let user of users; index as i">
      <td>{{ i + 1 }}</td>
      <td>{{ user }}</td>
    </tr>
  </tbody>
```

<div id='id3.2' />

## 3.2 ngIf

```ts
<table *ngIf="users != undefined && users.length > 0">
  ...
</table>

<ng-template #isEmpty>
  <div>No existe registro en el sistema!</div>
</ng-template>
```

## 3.3 ngIfElse

<div id='id3.3' />

```ts
<table *ngIf="(users != undefined && users.length > 0) else isEmpty">
  <thead>
    <tr>
      <th>name</th>
    </tr>
  </thead>
  <tbody>
    <tr *ngFor="let user of users; index as i">
      <td>{{ i + 1 }}</td>
      <td>{{ user }}</td>
    </tr>
  </tbody>
</table>
<ng-template #isEmpty>
  <div>No existe registro en el sistema!</div>
</ng-template>
```

## 3.4 ng-template #name

<div id='id3.4' />

Es una plantilla para uso en el .html

```ts
<ng-template #isEmpty>
  <div>No existe registro en el sistema!</div>
</ng-template>
```

## 3.5 button (click)

<div id='id3.5' />

```ts
// app.component.ts
visible: boolean = false;
setVisible(): void {
  this.visible = !this.visible;
}

// app.component.html
<button (click)="setVisible()">
  {{ visible ? 'Ocultar' : 'Mostrar' }}
</button>
```

<div id='id4' />

## 4. Directivas de angular(nuevas)

@if

```ts
@if(condition){
  ...
}@else{
  ...
}
```

@for

```ts
@for (user of users; track $index) {
<tr>
  <td>{{ $index + 1 }}</td>
  <td>{{ user }}</td>
</tr>
}
```

Ejemplo de uso

```ts
@if(visible){
<h2>{{title}}</h2>
@if (users != undefined && users.length > 0) {
<table>
  <thead>
    <tr>
      <th>name</th>
    </tr>
  </thead>
  <tbody>
    @for (user of users; track $index) {
    <tr>
      <td>{{ $index + 1 }}</td>
      <td>{{ user }}</td>
    </tr>
    }
  </tbody>
</table>
}@else {
<p>No existe registro en el sistema!</p>
}
```

<div id='id5' />

## 5. Estructura de un componente

```ts
// Importaciones
import { CommonModule } from "@angular/common";
import { Component } from "@angular/core";
import { log } from "node:console";
// import { RouterOutlet } from '@angular/router';

// Estructura y paquetes que se importan
@Component({
  selector: "app-root",
  standalone: true,
  imports: [CommonModule], //RouterOutlet
  templateUrl: "./app.component.html",
  styleUrl: "./app.component.css",
})

// Exportar, funciones, variables y todo el codigo
export class AppComponent {
  title = "Hola mundo angular!";
  users: string[] = ["Juan", "Pedro", "Andrez", "Jose"];
  // users?: string[];

  visible: boolean = false;
  setVisible(): void {
    this.visible = !this.visible;
    console.log("Hemos hecho click en el setVisible");
  }
}
```

<div id='id6' />

## 6. OnInit

Es un hook de ciclo de vida de **_angular_** que se ejecuta antes de cargar todas las directivas del componente.

```ts
export class CounterComponent implements OnInit {
  counter: number = 0;
  ngOnInit(): void {
    if (typeof window !== "undefined" && window.localStorage) {
      const saved = localStorage.getItem("counter");
      this.counter = saved !== null ? parseInt(saved) : 0;
    } else {
      this.counter = 0;
    }
    console.log("creando componente");
  }

  setCounter(): void {
    this.counter = this.counter + 1;
    localStorage.setItem("counter", this.counter.toString());
  }
}
```

**_nota:_** La condicion en el if (**_typeof window !== 'undefined' && window.localStorage_**) porque el localStorage en el SSR no esta definido y hay que validar que lo estoy usando en el navegador
