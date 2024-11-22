# Clases Abstractas e Interfazes

**Índice**

1. [Clases Abstractas](#id1)
   - [Cuando se usan las clases abstractas?](#id1.1)
   - [Declaracion y uso](#id1.2)
2. [Interfazes](#id2)
   - [Cuando se usan las interfazes?](#id2.1)
   - [Declaracion y uso](#id2.2)

<div id='id1' />

# 1. Clases Abstractas

- Son un **_tipo particular de clases_** cuya principal caracteristicas es que no puede ser intanciada.
- Generalmente declara la existencia de metodos pero no su implementacion, convirtiendola en una **_clase padre_**.
- Al menos **_uno_** de sus **_metodos_** debe de ser **_abstracto_**(puede tener metodos no abstractos).
- Sus niveles de visualizacion deben de ser o **_public_** o **_protected_** (nunca private)
- Cuando se usan una de las clases abstractas, una clase **_no puede heredar de varias abstractas a las vez_**(como en el caso de la interfazes).
- Generalmente las clases abstractas indican el "**_ES/SER_**" de un objeto

<div id='id1.1' />

## 1.1 Cuando se usan las clases abstractas?

Cuando deseamos definir **_una abstraccion_** que englobe objetos de distintos tipos y queremos hacer uso del polimorfismo.

Ejemplo:

- **_Figura_** es una **_clase abstracta_** porque no tiene sentido calcular su area, pero si la de sus hijos (**_Cuadrado_** y **_Circulo_**).

- Si una subclase de **_Figura_** (**_Cuadrado_** o **_Circulo_**) no redefine el metodo **calcularArea()**, deberia tambien declararse como clase abstracta.

<div id='id1.2' />

## 1.2 Declaracion y uso.

```java
public abstract class Figura {
  protected double x;
  protected double y;

  protected Figura(double x, double y) {
      this.x = x;
      this.y = y;
  }

  protected Figura() {
  }

  public abstract double calcularArea();
}

public class Circulo extends Figura{
  private double radio;

  public Circulo() {
  }

  public Circulo(double x, double y, double radio) {
      super(x, y);
      this.radio = radio;
  }

  @Override
  public double calcularArea() {
      double pi = Math.PI;
      return  pi * (this.radio * this.radio);
  }
}

public class Cuadrado extends Figura {
  private double lado;

  public Cuadrado() {
  }

  public Cuadrado(double x, double y, double lado) {
      super(x, y);
      this.lado = lado;
  }

  @Override
  public double calcularArea() {
      return this.lado * this.lado;
  }
}
```

<div id='id2' />

# 2. Interfazes

- Son **_colecciones de metodos abstractos_** con propiedades (atrubutos) **_CONSTANTES_**.
- Una interfaz **_solamente puede extender o implementar otras interfazes_**(la cantidad que quiera);
- Da a conocer que **_se debe hacer_** (metodos) **_pero sin mostrar su implementacion_** (solo puede tener metodos abstractos).
- Solo pueden tener **_metodos_** con **_acceso publico_** (no pueden ser pritected o pirvate).
- Solo pueden tener "_variables_" **_public static final_** (constantes).
- La palabra "**_abstract_**" en la definicion del metodo no es obligatoria.
- Generalmente las interfazes indican el que "**_PUEDE HACER_**" de un objeto.

## 2.1 Cuando se usan las Interfazes?

Sino fuese necesario conocer la ubicacion de una figura (x, y), se podria eliminar por completo los atributos y convertir a Figura en una **_interface_**.

Ejemplo:

**_Figura_** nos da ahora metodos que todas las figuras pueden tener; mientras que **_Rotable_** y **_Dibujable_** metodos que solo algunas pueden implementar.

No se utiliza la palabra **_extends_**, sino la palabra **_implements_**.

## 2.2 Declaracion y uso.

```java
public interface Dibujable {
  void dibujar();
}

public interface Rotable {
  void rotar();
}

public interface Figura {
  double calcularAreal();
}

public class Circulo implements Dibujable, Figura, Rotable{
  private double radio;

  public Circulo() {
  }

  public Circulo(double radio) {
      this.radio = radio;
  }

  @Override
  public void dibujar() {
      System.out.println("Dibujando un Circulo....");
  }

  @Override
  public double calcularAreal() {
      return 0;
  }

  @Override
  public void rotar() {
      System.out.println("Rotando el circulo....");
  }
}

public class Cuadrado implements Dibujable, Figura{
  private double lado;

  public Cuadrado() {
  }

  public Cuadrado(double lado) {
      this.lado = lado;
  }

  @Override
  public void dibujar() {
      System.out.println("Dibujando un Cuadrado....");
  }

  @Override
  public double calcularAreal() {
      return 0;
  }
}
```
