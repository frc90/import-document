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
