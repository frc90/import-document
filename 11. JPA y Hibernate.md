# Docker

## Índice

1. [Que es JPA](#id1)
2. [Relasiones entre Entities - (Clases)](#id2)
   - [2.1 Relasion de 1 a 1](#id2.1)
   - [2.2 Relasion de 1 a n](#id2.2)
   - [2.3 Relasion de n a m](#id2.3)

<div id='id1' />

## 1. Que es JPA

**_JPA_** (**_Java Persistence API_**) es una especificación de Java que define cómo acceder, conservar y administrar datos entre una base de datos relacional y objetos o clases. **_Hibernate_** es una herramienta de mapeo objeto-relacional (**_ORM_**) que implementa la especificación de **_JPA_**, pero con funcionalidades adicionales:

- **_JPA_**

Define un conjunto de interfaces y anotaciones para el mapeo objeto-relacional. **_JPA_** crea, lee y elimina réplicas de clases de entidad vinculadas mediante la interfaz _EntityManager_.

- **_Hibernate_**

Facilita el mapeo de atributos entre una base de datos relacional y el modelo de objetos de una aplicación. **_Hibernate_** realiza operaciones de base de datos y utiliza el lenguaje de consulta orientado a objetos _Hibernate Query Language_ (_HQL_). **_Hibernate_** también puede trabajar con bases de datos NoSQL, como MongoDB.

**_JPA_** se puede utilizar con **_Hibernate_** como proveedor subyacente, lo que permite cambiar fácilmente entre diferentes implementaciones de **_JPA_**.

Entidad (**_Entity_**)

**nota:** Una entidad con las anotaciones correspondientes nos dice que sera una **tabla** en la base de datos (**DB**)

**eg.:**

```java
@Entity
@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class Club {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(
            nullable = false,
            unique = true
    )
    private String name;
}
```

**nota:** Si se quiere expecificar el nombre de la tabla se le pone una anotacion extra **_@Table_**

**eg.:**

```java
@Entity
...
@Table(name = "club_tbl") // nombre de la tabla
public class Club {
    ...
}
```

**nota:** Java trabaja las propiedades de las clases como **_Camelcase_**, cuando se pasa a la **DB** pasaria con un quion bajo.

**eg.:**

```java
@Entity
...
public class Club {
    @Column(name = "last_name") // buena practica
    private String lastName; // last_name
}
```

<div id='id2' />

## 2. Relasiones entre Entities - (Clases)

Existen muchos tipos de relasiones en **_JPA_**

- **Relasiones** de 1 a 1
- **Relasiones** de 1 a **_n_**
- **Relasiones** de **_n_** a 1
- **Relasiones** de **_n_** a **_m_**

Las relasiones entre entidades va a ser lo mismo que las relasiones entre tablas para la base de datos, lo que va hacer **_JPA_** es mapear esas entidades y relasiones a travez de las clases.

### 2.1 Relasion de 1 a 1

<div id='id2.1' />

Vamos a ver la relasion de 1 a 1. Para este ejemplo seria:

> Un equipo (**Club**) tiene un entrenador (**Coach**) y un entrenador (**Coach**) pertenece a un equipo (**Club**)

Lo que se hace en **SQL** seria: La clave **primaria** pasa a ser clave **foranea** de una de las tablas. Cual de las 2 deberia tener la clave **foranea** de la otra.

- Depende de las necesidades del negocio.
- Para este caso va a servir mas tener la clave foranea en **Club**

Porque cuando se consulte a la **BD** por el **Club** va a interesar saber cual es el **Coach** asignado a ese club.

**nota:** Tambien se puede tener en **Coach** y funcionaria igualmente. Depende de las necesidades del problema.

**eb.:**

```java
@Entity
...
public class Club {
    ...
    @OneToOne(
            targetEntity = Coach.class,
            cascade = CascadeType.PERSIST
    )
    private Coach coach;
}
```

Propiedades de **_@OneToOne_**

- targetEntity = Club.class -->> la entidad con la relasion de la clave foranea

- cascade = -->> comportaminto en cascada
  - CascadeType.REMOVE
  - CascadeType.REFRESH
  - CascadeType.MERGE
  - CascadeType.DETACH
  - CascadeType.ALL
  - CascadeType.PERSIST

**PERSIST**: si guardo en la **DB** un **Club** tambien va a guarda el **Coach**. Se harian 2 **_insert_**

**REMOVE**: si se elimina de la **DB** un **Club** tambien va a eliminar el **Coach**. Se harian 2 **_delete_**

**MERGE**: si se actualiza en la **DB** un **Club** tambien va a actualizar el **Coach**. Se harian 2 **_update_**

Si se quiere cambiar el nombre de la clave foranea se pondria esta anotacion **@JoinColumn**.

**eg.:**

```java
@Entity
...
public class Club {
    ...
    @OneToOne(
            targetEntity = Club.class,
            cascade = CascadeType.PERSIST
    )
    @JoinColumn(name = "id_coach")
    private Coach coach;
}
```

### 2.2 Relasion de 1 a n

<div id='id2.2' />

Vamos a ver la relasion de 1 a n. Para este ejemplo seria:

> Un equipo (**Club**) puede tener **_1_** o **_n_** jugadores(**Player**) y un jugador (**Player**) pertenece a un solo equipo (**Club**)

Lo que se plante en **SQL** seria: La clave **primaria** de la tabla **_1_** pasa a ser clave **foranea** de tabla **_n_**.

**eg.:**

Clase **Club** de 1 a n, extremo 1

```java
...
public class Club {
    ...
    @OneToMany(
            targetEntity = Player.class,
            fetch = FetchType.LAZY,
            mappedBy = "club"
    )
    private List<Player> players;
}
```

Clase **Player** de n a 1, extremo n

```java
...
public class Player {
    ...
    @ManyToOne(targetEntity = Club.class)
    private Club club;
}
```

**Club** va a tener esta anotacion **_@OneToMany_**

- targerEntity = Player.class
- fetch =
  - FetchType.EAGER
  - FetchType.LAZY

> **LAZY** va a obtener la plantilla de player cuando lo solicite yo

> **EAGER** va a obtener la plantilla de player y los guardara en memoria

- mappedBy = "club"

**nota:** Hasta aca todo bien pero este ejemplo crea una tabla intermedia con ambas **_claves primarias_**, esta **_mal_**. Para evitar crear una tabla intermedia, lo que hace es mapear el **_atributo_** en **_mappedBy_** con la propiedad de la clase **_n_**.

**Player** va a tener esta anotacion **_@ManyToOne_**

- targetEntity = Club.class -> relasion correspondiente

**eg.: 2**

```java
...
public class Club {
    ...
    @ManyToOne(targetEntity = FootballAssociation.class)
    private FootballAssociation footballAssociation;
}
```

Clase **Player** de n a 1, extremo n

```java
...
@Entity
...
public class FootballAssociation {
    ...
    @OneToMany(
            targetEntity = Club.class,
            mappedBy = "footballAssociation",
            fetch = FetchType.LAZY
    )
    private List<Club> clubs;
}
```

### 2.3 Relasion de n a m

<div id='id2.3' />

Lo que se plantea en SQL seria: se tiene que crear una tabla intermedia la cual tenga las claves foranes de las tablas (**_en este caso entidades_**) correspondientes a las relasion.

**eg.:**

```java
...
public class Club {
    ...
    @ManyToMany(
            targetEntity = FootballCompetition.class,
            fetch = FetchType.LAZY
    )
    private List<FootballCompetition> footballCompetitions;

}
```

Esto lo que hace es que crea la tabla (**_club_football_competitions_**) intermedia con los id de cada extremo de cada relasion. Esta seria la forma correcta de hacerlo, dependiendo el problema.

Tambien hay otra manera de poner las relasiones manualmente.
**eg.2:**

```java
...
public class Club {
    ...
    @ManyToMany(
            targetEntity = FootballCompetition.class,
            fetch = FetchType.LAZY
    )
    @JoinTable(
            name = "club_competition",
            joinColumns = @JoinColumn(name = "club_id"),
            inverseJoinColumns = @JoinColumn(name = "competition_id")
    )
    private List<FootballCompetition> footballCompetitions;

}
```

Para hacerlo manualmente, tendremos en cuenta determinadas propiedades que van dentro de la anotacion principal **_@JoinTable_**

- @JoinTable() -> anotacion para crear la relasion muchos a muchos
- name = "club*competition" -> nombre de la tabla en la \*\*\_DB*\*\*
- joinColumns = @JoinColumn(name = "club_id") -> crea el campo con el id de referencia de la tabla que se esta creando la relasion
- inverseJoinColumns = @JoinColumn(name = "competition_id") -> crea el campo con el id de referencia de la segunda tabla de la relasion

```java
...
@Entity
...
public class FootballCompetition {
    ...
    @ManyToMany(
            targetEntity = Club.class,
            fetch = FetchType.LAZY
    )
    private List<Club> clubs;
}
```

**nota:** Si se Ponen las 2 relasiones en ambas clases (**Entity**) lo que hace es que crea la tabla (**_club_football_competitions_**) intermedia con los id de cada extremo de cada relasion y crea la tabla (**_football_competitions_club_**). **ESTO NO ES LO CORRECTO**.
