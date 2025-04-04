# Threads en Java (Hilos) - Programacion concurrente

**Índice**

1. [Concurrencia](#id1)
2. [Hilos](#id2)

## Concurrencia

<div id='id1' />

### Descripcion

La concurrencia se define a la **capacidad del sistema** o programa para **realizar multiples tareas al mismo tiempo**, de manera aparentemente simultanea.

En un contexto de **programacion**, la concurrencia implica que **multiples hilos de ejecucion** o procesos estan en fucnionamiento al mismo tiempo y realizando sus tareas de manera independiente o concurrente.

En **Java**, la principal forma de implementar concurrencia es mediante los **HILOS (Threads)**

- Paralelismo
- Multitareas

## Hilos

<div id='id2' />

### Descripcion

Un **_hilo_** en **_Java_**, tambien conocido como **_"Thread"_**, es una unidad basica de ejecucion que permite que un programa realice multiples tareas de manera concurrente.

Los **_hilos_** permiten la ejecuccion de varias partes de un programa al mismo tiempo en forma de subprocesos sin que uno interfiera con el otro.

### Ciclo de vida de un hilo

<img src="img/ciclo de vide de los hilos.png" alt="Ciclo de vida" />

Por herencia

```java
public class Hilo extends Thread {
  public void run() {
      System.out.println("Hilo ejecutandose conclase Thread.");
  }
}
```

Mediante implementacion de inteface

```java
public class HiloRunnable implements Runnable {
  @Override
  public void run() {
      System.out.println("Hilo ejecutado usandorunnable.");
  }
}
```

Ejecutarlo en el metodo **_main()_**.

```java
public static void main(String[] args) {
    Hilo proceso = new Hilo();
    proceso.start();

    HiloRunnable runnable = new HiloRunnable();
    Thread hilo = new Thread(runnable);
    hilo.start();
}
```
