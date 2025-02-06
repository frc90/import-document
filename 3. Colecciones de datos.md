# Colecciones de datos en Java (Estructuras de datos)

**Índice**

1. [Listas](#id1)
2. [ArrayList](#id2)
3. [Colas](#id3)
4. [Pilas](#id4)
5. [Colas dobles](#id5)
6. [Set - Conjuntos - HashSet](#id6)
7. [TreeSet - LinkedHashSet](#id7)
8. [Map - HashMap](#id8)
9. [LinkedHashMap - TreeMap](#id9)
10. [Cellections](#id10)

## Descripcion

- Java tiene desde la version 1.2 todo un juego de clases e interfaces para guardar colecciones de objetos (No valido para tipos basicos) ( java.utils.\* ).
- Se tratan de estruturas dinamicas que pueden aumentar o disminuir de tamaño.

<div id='id1' />

## 1. Listas (interfaz **List**)

#### 1.1 Caracteristicas

- Las listas son colecciones de datos que permiten **_duplicados_**.

  - Se pueden crear con el metodo **asList()** de **Arrays**.

- Tienen orden y su acceso es **_posicional_** (como los **Arrays**).

  - El primero esta en la posicion **0**.
  - El metodo para obtener un elemento es **get()**.
  - El tamaño o largo lo obtenemos con **zise()**.

- Los elementos se pueden **_recorrer_** (iterar).

  - Bucle **for** y **foreach**.
  - Iterator normal: **iterator**.
  - Iterator extendido: **listIterator**.

- Se pueden buscar elemento.

  - Primera ocurrencia: **indexOf()**.
  - Ultima ocurrencia: **lastIndexOf()**.

- Operaciones con sublistas

  - Crear: **subList()**
  - Borrar: **clear()**

#### 1.2 Declaracion y uso.

```java
List<String> lista = Arrays.asList("Lunes", "Martes", "Miercoles", "Viernes", "Sabado", "Domingo");
List<Integer> listaDeEnteros = Arrays.asList(1, 2, 3, 4, 5, 6, 7);
List<Double> listaDeDoubles = Arrays.asList(1.5, 2.46, 45.5, 4.01, 5.20);

// primer elemento
lista.get(0);

// Recorrer con bucle foreach
for (String s : lista) {
    System.out.print(s + " ");
}

// Recorrer con bucle iterador
Iterator<String> it = lista.iterator();
while (it.hasNext()) {
    System.out.print(it.next() + " ");
}

// // Recorrer con iterador extendido
// Iterator<String> it = lista.listIterator(lista.zise());
// while(it.hasPrevious()){
//   System.out.print(it.previous() + " ");
// }

// sublista por indice de posicion
List<Integer> subListInt = listaDeEnteros.subList(2, 5);

ejercicio1();

// Ejercicio
public static void ejercicio1(){
  Scanner sc = new Scanner(System.in);
  System.out.println("Introdusca una lista de nombre con espacios: ");
  List<String> list = Arrays.asList(sc.nextLine().split(" "));
  System.out.println("Introdusca un nombre a buscar: ");
  String name = sc.next();
  if (list.contains(name)) {
      System.out.println("El nombre esta en la posicion: " + list.indexOf(name));
  }else {
      System.out.println("El nombre no se encuentra en la lista");
  }
}

```

<div id='id2' />

## 2. ArrayList

#### 2.1 Caracteristicas

- ArrayList esta en la jerarquia de **Collection**, y es una de las implementaciones mas habituales de **List**.

- El acceso por idices es muy eficiente y es la mejor opcion para las listas dinamicas.

- Tiene todos los metodos vistos en **List** y ademas algunas de sus operaciones basicas son:
  - **add(e)**: Añade al final de la lista.
  - **add(pos, e)**: Añade el elemento a las posicion **_pos_**.
  - **set(pos, e)**: Cambia el elemento de la posicion **_pos_** por el elemento **_e_**.
  - **remove(pos)**: Borra el elemento que se encuentra en la posicion **_pos_**.
- Otros metodos:
  - **addAll(c)**: Añade una coleccion a la lista.
  - **toArray()**: Convierte la lista en un array tradicional.
  - **remove(e)**: Elimina la primera aparicion de **_e_** en la lista.
  - **contains(e)**: Pregunta si el elemento esta en la coleccion.

#### 2.2 Declaracion y uso.

```java
List<String> lista = Arrays.asList("Lunes", "Martes", "Miercoles", "Viernes", "Sabado", "Domingo");
System.out.println("lista = " + lista);

Collection<String> lista1 = new ArrayList<>(lista);
System.out.println("lista1 = " + lista1);

ArrayList<Character> listChar = new ArrayList<>();
listChar.add('a');
listChar.add('e');
listChar.add('i');
listChar.add('o');
listChar.add('u');
System.out.println("listChar = " + listChar);

listChar.set(2, 'x');
System.out.println("listChar = " + listChar);

listChar.remove(3);
System.out.println("listChar = " + listChar);

listChar.add(3, 'c');
System.out.println("listChar = " + listChar);

listChar.add(4, 'r');
System.out.println("listChar = " + listChar);

ArrayList<Integer> integerArrayList = new ArrayList<>();
integerArrayList.addAll(Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9));
System.out.println("Lista inicial: \t\t\t" + integerArrayList);

Object[] listArr = new Integer[lista.size()];
listArr = integerArrayList.toArray();

List<Integer> subListInteger = integerArrayList.subList(1, 6);
subListInteger.clear();
System.out.println("integerArrayList = " + integerArrayList);

integerArrayList.remove((Object) 1);
System.out.println("integerArrayList = " + integerArrayList);

System.out.println("integerArrayList.contains((Object) 9) = " + integerArrayList.contains((Object) 9));;
ejercicio();

// Ejercicipublic static void ejercicio(){
Scanner sc = new Scanner(System.in);
System.out.println("Introdusca el primer valor: ");
int initialValue = sc.nextInt();
System.out.println("Introdusca el segundo valor: ");
int finalValue = sc.nextInt();
System.out.println("Multiplos a localizar: ");
int multi = sc.nextInt();
ArrayList<Integer> arrayList = new ArrayList<>();

for (int i = initialValue; i <= finalValue; i++) {
    arrayList.add(i);
}

for (int i = 0; i < arrayList.size(); i++) {
    if (arrayList.get(i) % multi != 0) {
        arrayList.set(i, 0);
    }
}
System.out.println("arrayList: \t\t" + arrayList)}
```

<div id='id3' />

## 3. Colas

### **Queve**:

Se conoce como **cola** a una coleccion de tipo lista, diseñada para insertar al final y eliminar al inicio.
Por lo general las colas siguen un patron que en computacion se conoce como **FIFO** (siglas en ingles - "First in - First out", primero en entrar, primero en salir). Como la cola de un supermercado.

Como todas las listas, por defecto se agregan al final de la lista. La novedad en este tipo de estructura, es que se saca por el principio.

Para las colas se pueden utilizar las **ArraysDeque**, **LinkedList** y **PriorityQueve**

### Particularidad

<table>
  <tr>
    <th></th>
    <th>Lanza una excepcion</th>
    <th>Devuelve valor especial</th>
  </tr>
  <tr>
    <td><i>Insercion</i></td>
    <td>add(e)</td>
    <td>offer(e)</td>
  </tr>
  <tr>
    <td><i>Extraer</i></td>
    <td>remove()</td>
    <td>poll()</td>
  </tr>
  <tr>
    <td><i>Examinar</i></td>
    <td>element()</td>
    <td>peek()</td>
  </tr>
</table>

#### 3.2 Declaracion y uso.

```java
Queue<String> month = new ArrayDeque<>();

month.add("Enero");
month.add("Febrero");
month.add("Marzo");
month.add("Abril");
month.add("Mayo");
month.add("Junio");
month.add("Julio");
month.offer("Agosto");
month.offer("Septiembre");
month.offer("Octubre");
month.offer("Noviembre");
month.offer("Diciembre");

while(!month.isEmpty()){
    System.out.println("month: " + month.remove());
}

Queue<String> myCollection = new LinkedList<>();
myCollection.add("Collection");
myCollection.add("List");
myCollection.add("Set");
myCollection.add("SortedSet");
myCollection.add("Map");

String elto;
Iterator<String> it = myCollection.iterator();
while (it.hasNext()){
    elto = it.next();
    if (elto.charAt(0) == 'S') {
        it.remove();
    }else{
        System.out.println("elto: " + elto);
    }
}

Queue<String> pasajeros = new PriorityQueue<>();
pasajeros.add(String.valueOf(new Billete("ELena", 1)));
pasajeros.add(String.valueOf(new Billete("Pedro", 2)));
pasajeros.add(String.valueOf(new Billete("Marcos", 3)));
pasajeros.add(String.valueOf(new Billete("Jesus", 1)));
pasajeros.add(String.valueOf(new Billete("Luisa", 6)));
pasajeros.add(String.valueOf(new Billete("Ester", 5)));

System.out.println("El primero en embarcar es: " + pasajeros.poll());
System.out.println("El resto de pasajero: ");
while (!pasajeros.isEmpty()){
    System.out.println(pasajeros.remove());
}

static class Billete implements Comparable<Billete>{
  private String nombre;
  private int prioridad;

  public Billete(String nombre, int prioridad) {
      this.nombre = nombre;
      this.prioridad = prioridad;
  }

  public String getNombre() {
      return nombre;
  }

  public void setNombre(String nombre) {
      this.nombre = nombre;
  }

  public int getPrioridad() {
      return prioridad;
  }

  public void setPrioridad(int prioridad) {
      this.prioridad = prioridad;
  }

  @Override
  public String toString(){
      return "Billete [nombre="+ this.nombre +" ,\tprioridad="+ this.prioridad +"]";
  }

  @Override
  public int compareTo(Billete o) {
      return -Integer.compare(prioridad, o.getPrioridad());
  }
}
```

<div id='id4' />

## 4. Pilas

### **Stack**:

Se conoce como una **pila** a una coleccion de tipo lista diseñada para insertar (apilar) y eliminar (desapilar) del final. Es lo que hacemos cuando amontonamos libros.

Las pilas siguen un patron computacional se conoce como **LIFO** ("Last in - First out", ultimo en estrar primero en salir).

### Operaciones

<table>
  <tr>
    <th></th>
    <th>Metodo Stack< E ></th>
    <th>Equivalente en Dequeve< E ></th>
  </tr>
  <tr>
    <td><i>Insercion</i></td>
    <td>push(e)</td>
    <td>
      addFirst(e),
      offerFrist(e)
    </td>
  </tr>
  <tr>
    <td><i>Extraer</i></td>
    <td>pop()</td>
    <td>
      removeFirst(),
      pollFrist()
    </td>
  </tr>
  <tr>
    <td><i>Examinar</i></td>
    <td>peek()</td>
    <td>
      getFirst(),
      peekFirst()
    </td>
  </tr>
</table>

### Caracteristicas:

- Las operaciones mas abituales son **push()**, **pop()** y **peek()**(esta ultima devuelve el elemento de la cima sin retirarlo).

- Para su implementacion se utiliza la clase **Stack** o la clase **ArrayDequeve** (que puede comportarse como pila o cola).

- **NOTA:** Segun la documentacion de Java se recomiendo utilizar ArrayDequeve.

#### 4.2 Declaracion y uso.

```java
Stack<String> pila = new Stack<>();

pila.push("Primero");
pila.push("Segundo");
pila.push("Tercero");
pila.push("Cuarto");
pila.push("Quinto");

while (!pila.isEmpty()){
    System.out.println(pila.pop());
}

Deque<String> pila1 = new ArrayDeque<>();
pila1.addFirst("Prinero");
pila1.addFirst("Segundo");
pila1.offerFirst("Tercero");
pila1.offerFirst("Cuarto");
pila1.push("Quinto");

System.out.println(pila1.pollFirst());
System.out.println(pila1.pollFirst());
System.out.println(pila1.removeFirst());
System.out.println(pila1.removeFirst());
System.out.println(pila1.pop());
```

<div id='id5' />

## 5. Colas dobles

### **doble-ended queve**:

Representa a una "**doble-ended queve**", es decir, una cola en la que los elementos pueden añadrise no solo al final, sino tambien "empujrse" al principio.

Pueden funcionar como una **cola** o como una **lista**.

### Operaciones:

- Incorpora las terminaciones **last** y **first** para realizar las operaciones de pila y cola.
- Se pueden realizar todas las combinaciones con los metodos **add()**, **offer()**, **remove()**, **poll()**, **get()** y **peek()**.
- Para su implementacion podemos utilizar **ArrayDequeve** y **LinkedList**.

#### Dequeve como **cola**

<table>
  <tr>
    <th></th>
    <th>Metodo Queve< E ></th>
    <th>Equivalente en Dequeve< E ></th>
  </tr>
  <tr>
    <td><i>Insercion</i></td>
    <td>
      add(e),
      offer(e)
    </td>
    <td>
      addLast(e),
      offerLast(e)
    </td>
  </tr>
  <tr>
    <td><i>Extraer</i></td>
    <td>
      remove(),
      poll()
    </td>
    <td>
      removeFirst(),
      pollFrist()
    </td>
  </tr>
  <tr>
    <td><i>Examinar</i></td>
    <td>
      element(), 
      peek()
    </td>
    <td>
      getFirst(),
      peekFirst()
    </td>
  </tr>
</table>

#### Dequeve como **pila**

<table>
  <tr>
    <th></th>
    <th>Metodo Stack< E ></th>
    <th>Equivalente en Dequeve< E ></th>
  </tr>
  <tr>
    <td><i>Insercion</i></td>
    <td>push(e)</td>
    <td>
      addFirst(e),
      offerFrist(e)
    </td>
  </tr>
  <tr>
    <td><i>Extraer</i></td>
    <td>pop()</td>
    <td>
      removeFirst(),
      pollFrist()
    </td>
  </tr>
  <tr>
    <td><i>Examinar</i></td>
    <td>peek()</td>
    <td>
      getFirst(),
      peekFirst()
    </td>
  </tr>
</table>

#### 5.2 Declaracion y uso.

```java
int[] initialList = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
Deque<Integer> finalList = new ArrayDeque<>();

for (Integer n : initialList) {
    if (n % 2 == 0) {
        finalList.addFirst(n);
    } else {
        finalList.addLast(n);
    }
}
System.out.println("Resultado final: " + finalList);


List<Integer> num = Arrays.asList(3, 4, 5, 6, 7, 8);
Deque<Integer> list = new LinkedList<>(num);
System.out.println("Inicial: \t" + list);
list.addFirst(2);
System.out.println("addFirst 2: \t" + list);
list.addLast(9);
System.out.println("addLast 9: \t" + list);
list.offerFirst(1);
System.out.println("offerFirst 1: \t" + list);
list.offerLast(10);
System.out.println("offerLast 10: \t" + list);

System.out.println("getFirst(): \t" + list.getFirst());
System.out.println("peekFirst(): \t" + list.peekFirst());
System.out.println("getLast(): \t" + list.getLast());
System.out.println("peekLast(): \t" + list.peekLast());
```

<div id='id6' />

## 6. Set - Conjuntos - HashSet

Un **Set** es una Collection de tipo "conjunto" que tiene 2 caracteristicas muy importantes:

- No hay repetidos.
- No Importa el orden(No hay acceso posicional).
- Dos conjuntos son iguales (**equals**) si contienen los mismos elementos.
- La ventaja principal de utilizar Sets es que preguntar si un elemento esta contenido mediante **contains()** es muy eficiente.

### **HashSet**:

- Es la implementacion mas frecuente de un conjunto y tambien la que mejor rendimiento tiene.
- El orden de sus elementos se debe al **codigo hash**(funcion resumen).
- Se mejora el rendimiento si se establece una capacidad inicial no muy eleveada.
- Todos sus metodos son heredados de **Collections**, por ejemplo:
  - **removeAll()**: Elimina todos los elementos que tiene el conjunto.
  - **retainAll()**: Hace lo contrario que la anterior, mantiene los elementos de un conjunto, eliminando al resto.

#### 6.2 Declaracion y uso.

```java
Set<String> conjuntoB = Set.of("Ana", "Luis", "Ines", "Beto");
Set<String> conjuntoB2 = Set.of("Beto", "Luis", "Ines", "Ana");
Set<String> conjuntoN = Set.of("Pedro", "Ana", "Beto");

System.out.println("Conjunto B: \t" + conjuntoB);
System.out.println("Conjunto B2: \t" + conjuntoB2);
System.out.println("Conjunto N: \t" + conjuntoN);

System.out.println("Luis en B: \t" + conjuntoB.contains("Luis"));
System.out.println("Luis en N: \t" + conjuntoN.contains("Luis"));

System.out.println("B igual a B2: \t" + conjuntoB.equals(conjuntoB2));
System.out.println("B igual a N: \t" + conjuntoB.equals(conjuntoN));

Collection<String> fruta = new HashSet<>();
fruta.addAll(Arrays.asList("Platano", "Manzana", "Naranja", "Pera"));
System.out.println("Orden de la fruta: " + fruta);

Set<String> conjuntoC = new HashSet<>(conjuntoB);
HashSet<String> conjuntoF = new HashSet<>(5);
conjuntoF.add("Ana");
conjuntoF.add("Beto");
conjuntoF.add("Pedro");
conjuntoF.add("Pedro");

conjuntoC.retainAll(conjuntoF);
System.out.println("Conjunto de interseccion C y F " + conjuntoF);
```

<div id='id7' />

## 7. TreeSet - LinkedHashSet

- Cuando necesitamos que los elementos del conjunto esten ordenados, debemos utilizar **TreeSet**.
- **TreeSet** construye un **arbol binario** ordenado con los objetos que se van agregando al conjunto. Esto tiene un coste, ya que es mas lento que el **HashSet**.
- Con numeros o cadenas de caracteres el orden es natural (numeros o alfabeticos, respectivamente).
- Con objetos es necesario implementar la intefaz **_comparable_**.

### Orden de insercion

- Con **LinkedHashSet** se mantiene el orden de insercion, utilizando para ello una tabla hash con una lista doblemente enlazada.
- Tiene mejor rendimiento que el **TreeSet**, y peor que el **HashSet**.
- Puede almacenar nulos (**null**) y tambien puede borrar elementos(**remove**).
- El metodo **containsAll()** sirve para saber si un conjunto es subconjunto de otro.

#### 7.2 Declaracion y uso.

```java
SortedSet<Integer> num = new TreeSet<>();

num.add(15);
num.add(18);
num.add(6);
num.add(21);
num.add(4);
num.add(10);
num.add(8);

String listado = "Beto Luis Ines Ana";
System.out.println("listado de nombres: " + listado);
Set<String> conjunto = Set.of(listado.split(" "));
TreeSet<String> nombres =  new TreeSet<>(conjunto);

String primero = nombres.first();
String ultimo = nombres.last();
System.out.println("El primer nombre: " + primero);
System.out.println("El siguiente es: " + nombres.higher(primero));
System.out.println("El ultimo nombre: " + ultimo);
System.out.println("El penultimo nombre: " + nombres.lower(ultimo));

String [] listNames = {"Beto", "Luis", "Ines", "Ana"};
System.out.println("listado de nombres \t" + Arrays.toString(listNames));

LinkedHashSet<String> names = new LinkedHashSet<>(5);
names.add(null);
System.out.println("names: \t\t" + names);
names.remove(null);

names.addAll(Arrays.asList(listNames));

System.out.println("Nombres ordenados por inscripcion: \t" + names);
System.out.println("Contiene a ana y beto: \t" + names.containsAll(Set.of("Ana", "Beto")));

TreeSet<String> ordenados = new TreeSet<>(names);
System.out.println("Noombres ordenados alfabeticamente: \t\t" + ordenados);
```

<div id='id8' />

## 8. Map - HashMap

Un **Map** representa lo que en otros lenguajes se conoce como **diccionario** y que suele asociar a la idea de "tabla hash".

Un map puede asemejarse a una tabla de bases de datos con dos columnas, la primera seria la **clave** o "key" y la segunda el **valor** o "value".

Las claves forman un cojunto en el sentido java: son un **Set**, no puede haber suplicados.

La jerarquia de map no hereda de collection.

Igual que Collection, no puede almacenar tipos basicos, por lo que hay que utilizar emvoltorios.

La implementacion mas habitual es **HashMap**.

#### Operaciones:

- **HashMap** es la implementacion que tiene mejor rendimiento.
- Se elige cuando el orden de los pares no es importante.
- Los metodos mas importantes de **HashMap**:
  - **get(k clave)**
    - Obtiene el valor correspondiente a una clave. Devuelve **null** si no existe esa clave en el map
  - **put(k calve, v valor)**
    - Añade un par clave-valor al map. Si ya habia un valor para esa clave se reemplaza.
  - **keySet()**
    - Devuelve todas las claves (devuelve un **Set**, sin duplicados)
  - **values()**
    - Devuelve todos los valores(que puedan estar duplicados, por lo tanto esta funcion devuelve una **Collection**).
  - **entrySet()**
    - Devuelve todos los pares clave-valor (devuelve un conjunto de objetos **Map.Entry**, cada uno de los cuales devuelve la clave y el valor de los metodos **getkey()** y **getValue()** respectivamente).

#### 8.2 Declaracion y uso.

```java
Map<String, Integer> notas = new HashMap<>();
notas.put("Antonio", 7);
notas.put("Pedro", 9);
notas.put("Luis", 10);
notas.put("Marco", 7);
notas.put("Maria", 4);
notas.put("Ana", 10);

System.out.println("La nota de Pedro es: " + notas.get("Pedro"));
System.out.println("Esta Maria?: " + notas.containsKey("Maria"));

double media = 0;
for(Integer nota : notas.values()){
    media += nota;
}
System.out.printf("La nota media es: %1.2f\n", media / notas.size());

for (Map.Entry<String, Integer> par: notas.entrySet()){
    System.out.println(par.getKey() + " -> " + par.getValue());
}
```
