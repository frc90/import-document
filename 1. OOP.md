# OOP (Programacion orientada a objetos)

La programacion orientada da objetos **(OOP)** es un paradigma de **_programacion imperativo_**.
Basado en que:

- Todo es un "**objeto**".
- La solucion se basa en la interaccion entre ellos.
- Se intenta modelar un subconjunto de la realidad.

> **Resumen**: La solucion en el paradigma orientado a objetos va a ser dado por la interaccion de los objetos y la forma que se comunican entre si

## Paradigma

Conjunto de reglas y formas de razonamiento que conforman una filosofia para la produccion de programas.

## Que es un **Objeto**

Es una entidad que dispone de estado, comportamiento e identidad.

- Propiedades: Llamadas atributos (variables).
- Estado de un objeto: viene dado por los valores de sus atributos.
- Comportamiento: Son las acciones u operaciones que puede realizar a travez de sus metodos.
- Identidad de un **objeto**: Propiedades que permite distinguirlos de otros. Generalmente se trata del espacio que ocupa en memoria.

> **Resumen**: Todo puede ser un objeto

## Diferencia entre Clase vs Objeto

Nota: La unica manera de crear objetos es previamente haber definido una clase.

### Que es una clase:

Molde o plantilla que permite crear objectos a partir de ello.

Crear una clase significa crear un molde o plantilla que nos va a permitir crear objetos a partir del mismo.

Ejemplo: un plano seria la **clase** y una casa seria el **objeto**.

> **Resumen**: Una clase asegura la creacion de objetos con el mismo conjuntos de atributos y metodos. **Un Objeto es la instancia de una clase.**

# Pilares de la programacion orientada a objetos (OOP)

- Abstraccion
- Encapsulamiento
- Herencia
- Polimorfismo

## Encapsulamiento

1. Deberiamos tomar ciertos atributos y ciertos metodos que hagan a un tipo particular de objetos, embolverlos o encapsularlos en una unica unidad llamada clase.

2. Los objetos deberian de mantener su integridad, ocultar sus detalles de implementacion para que no se pueda cambiar sin pasar por su voluntad.

## Herencia

Es el mecanismo por el cual se logran aprovechar o reutilizar las propiedades y metodos de una o varias clases ya existentes. Reduce la repeticion de codigo.

## Polimorfismo

Ad-hoc:

- Por sobrecarga
- Por Coercion

Universal:

- Parametrico
- Por Herencia

#### Polimorfismo por sobrecarga:

A dentro de una misma clase podrian llegar haber metodos que se llamen igual pero con diferentes tipo y cantidad de parametros.

#### Polimorfismo por herencia:

Mecanismo que permite enviar mensajes sintacticamente iguales a objetos de tipos distintos, cada uno con su propio comportamiento.

#### Polimorfismo por coercion:

Es cuando se pasan tipos diferentes pero se fuerza a que se pase el mismo tipo de dato.
