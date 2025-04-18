# Angular

**Índice**

1. [Signal](#id1)

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

<div id='id1' />