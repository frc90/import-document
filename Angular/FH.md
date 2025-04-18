# Angular

**Índice**

0. [Estructura de angular](#id0)
1. [Signal](#id1)

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
