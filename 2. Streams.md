# Streams & Lambdas

**Índice**

1. [Lambdas](#id1)
2. [Streams](#id2)

# 1. Lambdas

<div id='id1' />

**Lambda**

Las funciones lambdas es un término adoptado de la programación funcional y corresponden con funciones de Java que normalmente son anónimas y se escriben en línea allí donde se usan. Como cualquier función recibe cero o más argumentos y devuelven uno o ningún valor de retorno.

**Consumer:**

- Es una funciona **lambda** que nos provee java.
- Recive un valor y no retorna nada

```java
Consumer<String> consumer = (param) -> {
    System.out.println(param);
};
Consumer<String> consumer = (param) -> System.out.println(param);
Consumer<String> consumer = System.out::println;
consumer.accept("Hello world");
```

**Biconsumer:**

- Es una funciona **lambda** que nos provee java.
- Recive dos valores y no retorna nada

```java
BiConsumer<String, String> biconsumer = (t, u) -> {
    System.out.println(t + u);
};
BiConsumer<String, String> biconsumer = (t, u) -> System.out.println(t + u);
biconsumer.accept("Hello ", "world");
```

**Supplier:**

- Es una funciona **lambda** que nos provee java.
- No recive ningun valor y pero retorna un resultado.

```java
Supplier<String> supplier = () -> {
    return "Hello World";
};
Supplier<String> supplier = () -> "Hello World";
System.out.println(supplier.get());
```

**Function:**

- Es una funciona **lambda** que nos provee java.
- Recive un valor y retorna un resultado.

```java
Function<Integer, String> function = (num) -> {
    return "The number is: " + num;
};
Function<Integer, String> function = (num) -> "The number is: " + num;
System.out.println(function.apply(10));
```

**BiFunction:**

- Es una funciona **lambda** que nos provee java.
- Recive dos valores y retorna un resultado.

```java
BiFunction<Integer, Integer, Integer> biFunction = (a, b) -> {
    return a * b;
};
BiFunction<Integer, Integer, Integer> biFunction = (a, b) -> a * b;
System.out.println(biFunction.apply(1, 2));
```

**Predicate:**

- Es una funciona **lambda** que nos provee java.
- Recive un valor y retorna un boolean.

```java
Predicate<String> predicate = (str) -> {
    return str.length() > 5;
};
System.out.println(predicate.test("Hello world!"));
```

**BiPredicate:**

- Es una funciona **lambda** que nos provee java.
- Recive dos valores y retorna un boolean.

```java
BiPredicate<Integer, Integer> biPredicate = (a, b) -> {
    return a > b;
};
BiPredicate<Integer, Integer> biPredicate = (a, b) -> a > b;
System.out.println(biPredicate.test(2, 3));
```

**BinaryOperator:**

- Es una funciona **lambda** que nos provee java.
- Recive dos valores del mismo tipo y retorna un valor del mismo tipo.

```java
BinaryOperator<Integer> binaryOperator = (a, b) -> {
    return a + b;
};
BinaryOperator<Integer> binaryOperator = (a, b) -> a + b;
System.out.println(binaryOperator.apply(20, 3));
```

**UnaryOperator:**

- Es una funciona **lambda** que nos provee java.
- Recive un valores del mismo tipo y retorna un valor del mismo tipo.

```java
UnaryOperator<Integer> supplier = (num) -> {
    return num * num;
};
UnaryOperator<Integer> supplier = (num) -> num * num;
System.out.println(supplier.apply(78));
```

**Runnable:**

- Es una funciona **lambda** que nos provee java.
- No recive parametro y no retorna nada, solo ejecuta tarea.

```java
Runnable runnable = () -> {
    System.out.println("Executing Runnable....");
};
Runnable runnable = () -> System.out.println("Executing Runnable....");
runnable.run();
```

**Callable:**

- Es una funciona **lambda** que nos provee java.
- No recive valores, retorna un resultado y puede lanzar una exeptcion.

```java
Callable<String> callable = () -> {
    return "Result ...";
};
Callable<String> callable = () -> "Result ...";
try{
    System.out.println(callable.call());
} catch (Exception e) {
    throw new RuntimeException(e);
}
```

**NOTA:** Los **Callable**, **Runnable** son los que se utilizan para las concurrencia y trabajan con hilos, programacion multi-hilos.

# 2. Streams

<div id='id2' />

Un **Stream** se describe como una tuberia que transporta datos que se generan a patir de una **_lista_** (de izquierda a derecha o arriba abajo). Basicamente **_operaciones_** que van a porcesar los elementos de la **_lista_**.

## Ojetivo

**_Tranformar datos_** como si fuese una tuberia, entra un grupo de datos como **_Collection_** y se hace cualquier operacion que tenga **_stream_** y devuelve otra **_Collection_** con los datos restantes.

**nota:** Cuando se trabajar con expresiones lambdas hay que tener en cuenta que hay **_Operadores_** que son lo que nos ayudan a trabajar con la informacion.

**Operadores**

- **Finales** -> Termina el flujo
- **No finales** -> No termina el flujo (Se pueden hacer mas operaciones)

---

### **stream().forEach()**

Sirve para recorrer todos los elementos de una colleccion. Hace una accion por cada elemento.

**stream().forEach(Consumer<? super String> consumer)** -> Operador Final

Recive un **Consumer**

```java
List<String> names = Arrays.asList("Ana","Luis", "Maria", "Pedro", "Juan", "Carla");
names.stream().forEach((name) -> {
    System.out.println(name);
});
names.stream().forEach(System.out::println);
```

---

### **stream().filter()**

**filter()**: Filtra los elementos que cumplen una condicion y devuelve una coleccion.

**stream().filter(Predicate<? super String> predicate)** -> Operador no final

Recive un **Predicate**

```java
List<String> names = Arrays.asList("Ana","Luis", "Maria", "Pedro", "Juan", "Carla");
names.stream().filter(name -> {
    return name.length() > 4;
}).forEach(System.out::println);

List<String> nList = names.stream().filter(name -> {
    return name.length() > 4;
}).toList();
nList.forEach(System.out::println);
```

---

### **stream().map()**

**map()**: Tranforma los elementos aplicando alguna funcion. Sirve para modificar elementos.

**stream().map(Function<? super String, ?> mapper)** -> Operador no final

Recive un **Function**

```java
List<String> names = Arrays.asList("Ana","Luis", "Maria", "Pedro", "Juan", "Carla");
names.stream().map(name -> {
    return name.toUpperCase();
}).forEach(System.out::println);

names.stream().map(name -> name.toUpperCase()).forEach(System.out::println);
```

---

### **stream().sorted()**

**sorted()**: Ordena nos elementos de un stream

**stream().sorted(Comparator<? super String> comparator)** -> Operador no final

Recive un **Comparator**

```java
List<String> names = Arrays.asList("Ana","Luis", "Maria", "Pedro", "Juan", "Carla");
names.stream().sorted().forEach(System.out::println);
```

---

### **stream().reduce()**

**reduce()**: Combina todos los elementos en un solo valor.

**stream().reduce(BinaryOperator<? super String> acumulator)** -> Operador final

Recive un **BinaryOperator**

```java
List<String> names = Arrays.asList("Ana","Luis", "Maria", "Pedro", "Juan", "Carla");
String result = names.stream().reduce("Resultado: ", (a, b) -> {
    return a + " " + b;
});
System.out.println(result);
```

---

### **stream().collect()**

**collect()**: Recoje todos los elementos en una coleccion.

**stream().collect(Collector<? super String, A, R> collector)** -> Operador final

Recive un **Collector**

```java
List<String> names = Arrays.asList("Ana","Luis", "Maria", "Pedro", "Juan", "Carla");
List<String> list =  names.stream().map(name -> name.toUpperCase()).collect(Collectors.toList());
list.forEach(System.out::println);
```

**NOTA:** **_collect(Collectors.toList())_** esta disponible en versiones anteriores a las 17. A partir de la misma se puede sustituir directamente por **_toList()_**.

---

### **stream().distinct()**

**distinct()**: Elimina los elementos duplicados.

**stream().distinct()** -> Operador no final

No recive parametros.

```java
List<String> names = Arrays.asList("Ana","Luis", "Maria", "Pedro", "Juan", "Carla");
names.stream().distinct().forEach(System.out::println);
```

---

### **stream().limit()**

**limit()**: Limita el numero de elementos procesados.

**stream().limit(Long ?)** -> Operador no final

Recive un **Long**

```java
List<String> names = Arrays.asList("Ana","Luis", "Maria", "Pedro", "Juan", "Carla");
names.stream().limit(3).forEach(System.out::println);
```

---

### **stream().anyMatch()**

**anyMatch()**: Verifica si algun elemento de la coleccion cumple la condicion.

**stream().anyMatch(Predicate<? super String> predicate)** -> Operador final

Recive un **Predicate**

```java
List<String> names = Arrays.asList("Ana","Luis", "Maria", "Pedro", "Juan", "Carla");
boolean result = names.stream().anyMatch(name -> {
    return name.startsWith("A");
});
System.out.println(result);
```

---

### **stream().allMatch()**

**allMatch()**: Verifica si todos los elemento de la coleccion cumple la condicion.

**stream().allMatch(Predicate<? super String> predicate)** -> Operador final

Recive un **Predicate**

```java
List<String> names = Arrays.asList("Ana","Luis", "Maria", "Pedro", "Juan", "Carla");
boolean result = names.stream().allMatch(name -> {
    return name.startsWith("A");
});
System.out.println(result);
```

---

### **stream().noneMatch()**

**noneMatch()**: Verifica si ningun elemento de la coleccion cumple la condicion.

**stream().noneMatch(Predicate<? super String> predicate)** -> Operador final

Recive un **Predicate**

```java
List<String> names = Arrays.asList("Ana","Luis", "Maria", "Pedro", "Juan", "Carla");
boolean result = names.stream().noneMatch(name -> {
    return name.startsWith("A");
});
System.out.println(result);
```
