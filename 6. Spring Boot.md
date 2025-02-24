# Spring Boot

**Índice**

1. [Introduccion a Spring y Spring Boot](#id1)
   - [1.1 Que es un Bean?](#id1.1)
   - [1.2 Como funciona Hibernate(ORM - Object Relational Mapping)](#id1.2)
   - [1.3 Que es JPA?](#id1.3)
2. [Configuracion de application.properties](#id2)
   - [2.1 Conectar la aplicación Spring Boot a MySQL con **_application.yml_**](#id2.1)
   - [2.2 Conectar la aplicación Spring Boot a PostgreSQL con **_application.yml_**](#id2.2)
   - [2.3 Usar perfiles en Spring Boot (MySQL y PostgreSQL)](#id2.3)
   - [2.4 Conectar la aplicación Spring Boot a PostgreSQL y MySQL](#id2.4)
   - [2.5 Perfiles](#id2.5)
3. [JPA,JDBC y SQL queries](#id3)

<div id='id1' />

## 1. Introduccion

<div id='id1.1' />

### 1.1 Que es un Bean?

Es objeto unico que tiene ciertas caracteristicas que puede ser inyectado en cualquier lugar que desees. Se hace a travez de las anotaciones.

<div id='id1.2' />

### 1.1 Como funciona Hibernate (ORM - Object Relational Mapping)

#### `Hibernate`

(**_ORM - Relational Mapping_**)(**_Mapeo Objeto-Relacional_**) Hibernate es una libreria para mapear clases de Java en tablas de una base de datos relasional, disminuyendo considerablemente la cantidad de codigo SQL para interactuar con la base de datos.

Para hacer uso de Hibernate solo tienes que declarar los metodos getter y setter para cada campo y el se encarga del resto.

`Que hace`

Hibernate mapea los atributos de una base de datos tradicional con el modelo de objetos de una aplicación. Esto se logra mediante archivos declarativos o anotaciones en los beans de las entidades.

`Para qué sirve`

Hibernate optimiza el flujo de trabajo evitando código repetitivo.

`Cómo funciona`

Hibernate utiliza un lenguaje de consulta potente (HQL) que se parece a SQL, pero es orientado a objetos.

`Quién lo creó`

Gavin King y colegas de Cirrus Technologies crearon Hibernate en 2001 como alternativa al estilo de programación de EJB2.

`Cómo se relaciona con JPA`

JPA es una especificación estándar de Java que define un conjunto de interfaces y anotaciones para el mapeo objeto-relacional. Hibernate es una implementación concreta de esta especificación.

Hibernate y Spring son dos herramientas complementarias que se pueden utilizar juntas o separadas para desarrollar aplicaciones empresariales.

---

<div id='id1.3' />

#### 1.3 Que es JPA?

JPA (Java Persistence API) es una API que permite gestionar la persistencia de datos y la correlación de objetos en Java. JPA es una especificación estándar que se basa en el modelo de programación Java y puede funcionar en entornos Java EE o Java SE.

JPA permite: Asignar objetos Java a tablas de bases de datos relacionales, Realizar consultas, Manejar transacciones, Recuperar objetos sin necesidad de grabar consultas SQL específicas.

JPA funciona mediante anotaciones o XML para correlacionar objetos con una o más tablas de una base de datos. Para usar JPA, se necesita definir una Unidad de Persistencia (Persistence Unit) en un archivo XML llamado persistence.xml.

Una implementación popular de JPA es Hibernate ORM, que ofrece funcionalidades adicionales y un conjunto completo de herramientas para trabajar con persistencia de datos en Java.

<div id='id2' />

## 2. Configuracion de application.properties o application.yml

<div id='id2.1' />

### 2.1 Conectar la aplicación Spring Boot a MySQL con **_application.yml_**

archivo **_application.yml_**

```yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/car-rental # 'localhost' porque la base de datos está en tu máquina local
    username: root
    password: rootpassword
    driver-class-name: com.mysql.cj.jdbc.Driver
```

<div id='id2.2' />

### 2.2 Conectar la aplicación Spring Boot a PostgreSQL con **_application.yml_**

```yml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/car-rental # 'localhost' porque la base de datos está en tu máquina local
    username: user
    password: password
    driver-class-name: org.postgresql.Driver
```

<div id='id2.3' />

### 2.3 Usar perfiles en Spring Boot (MySQL y PostgreSQL)

```yml
spring:
  profiles:
    active: mysql # Cambia a 'postgres' para usar PostgreSQL

# Perfil MySQL
---
spring:
  config:
    activate:
      on-profile: mysql
  datasource:
    url: jdbc:mysql://localhost:3306/car-rental
    username: root
    password: rootpassword
    driver-class-name: com.mysql.cj.jdbc.Driver

# Perfil PostgreSQL
---
spring:
  config:
    activate:
      on-profile: postgres
  datasource:
    url: jdbc:postgresql://localhost:5432/car-rental
    username: user
    password: password
    driver-class-name: org.postgresql.Driver
```

<div id='id2.4' />

### 2.4 Conectar la aplicación Spring Boot a PostgreSQL y MySQL

archivo **_application.properties_**

```properties
// PostgreSQL
spring.jpa.database=POSTGRESQL
spring.sql.init.platform=postgres
spring.datasource.url=jdbc:postgresql://localhost:5432/[name]
spring.datasource.username=[user]
spring.datasource.password=[password]
#create, create-drop, validate, update, none
spring.jpa.hibernate.ddl-auto=[option]
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

spring.jpa.defer-datasource-initialization=true
spring.sql.init.mode=always


spring.datasource.hikari.connection-timeout=20000
spring.datasource.hikari.maximum-pool-size=5

logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE


// SQL
spring.application.name=courses
spring.datasource.url=jdbc:mysql://localhost:3306/courses
spring.datasource.username=root
spring.datasource.password=

spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA (Hibernate) Properties
# Automatically creates/updates tables based on entities
spring.jpa.hibernate.ddl-auto=update
# To print SQL queries for debugging
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect


# http://127.0.0.1:8080/swagger-ui.html

# This will display the SQL statements in your IDE's console as Hibernate generates tables.
logging.level.org.hibernate.SQL=debug

logging.level.org.hibernate.orm.jdbc.bind=trace


// H2 en memoria
spring.h2.console.enabled=true
spring.datasource.url=jdbc:h2:mem:dbmallapp
spring.datasource.driver-class-name=org.h2.Driver
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.H2Dialect
```

<div id='id2.4.1' />

### 2.4.1 Crear app en docker con application.properties y MySQL (con variable globales)

archivo **_application.properties_**

```properties
spring.datasource.url=${DATABASE_URL}
spring.datasource.username=${DATABASE_USERNAME}
spring.datasource.password=${DATABASE_PASSWORD}

spring.application.name=car-rental
spring.jpa.hibernate.ddl-auto=update
server.port=8080
#spring.datasource.driver-class-name=${DATABASE_DRIVER}
#spring.jpa.database-platform=${DATABASE_PLATFORM}
```

archivo **_Dockerfile_**

```Dockerfile
FROM eclipse-temurin:17

LABEL author=frc90

ENV DATABASE_URL jdbc:mysql://mysql:3306/car-rental
ENV DATABASE_USERNAME user
ENV DATABASE_PASSWORD password
#ENV DATABASE_PLATFORM org.hibernate.dialect.MySQL57Dialect
#ENV DATABASE_DRIVER com.mysql.cj.jdbc.Driver

COPY target/car-rental-0.0.1-SNAPSHOT.jar app.jar

ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Crear el **_docker-compose.yml_**

```yml
version: "3.8"
services:
  app:
    container_name: car-rental-container
    image: car-rental:1.0.0
    build: .
    ports:
      - 8080:8080
    environment:
      - DATABASE_URL=jdbc:mysql://mysql:3306/car-rental
      - DATABASE_USERNAME=root
      - DATABASE_PASSWORD=password
    depends_on:
      - mysql
  mysql:
    image: mysql:8
    container_name: mysql
    ports:
      - "3306:3306"
    environment:
      MYSQL_DATABASE: car-rental
      MYSQL_USER: user
      MYSQL_PASSWORD: password
      MYSQL_ROOT_PASSWORD: password
```

### Crear el empaquetado de el .jar para crear la imagen y luego el contenedor

```bash
mvn clean package -DskipTests
```

Si no tenemos aun BD creada y nos queremos saltar las pruebas

```bash
mvn clean package
```

Si queremos correr con las pruebas de conexion a la base de datos.

#### Ejecutar el siguiente codigo en bash.

Detener todos los contenedores.

```bash
docker-compose down
```

Construir una nueva imagen a partir de la app de spring boot.

```bash
docker-compose build

```

Crear las imagenes.

```bash
docker-compose up -d
```

Levantar los contenedores, crearlos sino estan creados y levantarlos.

```bash
docker-compose up
```

<div id='id2.5' />

### 2.5 Perfiles

application.yml

```yml
server:
  port: 8080
spring:
  profiles:
    active: dev
---
spring:
  config:
    activate:
      on-profile: dev
  application:
    name: market-management
  datasource:
    password: ""
    url: jdbc:mysql://localhost:3306/market_management
    username: root
  jpa:
    hibernate:
      ddl-auto: update
    properties:
      hibernate:
        format_sql: true
    show-sql: true
---
spring:
  config:
    activate:
      on-profile: qa
  application:
    name: market-management
  datasource:
    password: ""
    url: jdbc:mysql://localhost:3306/market_management_qa
    username: root
  jpa:
    hibernate:
      ddl-auto: update
    properties:
      hibernate:
        format_sql: true
    show-sql: true
---
spring:
  config:
    activate:
      on-profile: prod
  application:
    name: market-management
  datasource:
    password: ""
    url: jdbc:mysql://localhost:3306/market_management_prod
    username: root
  jpa:
    hibernate:
      ddl-auto: update
    properties:
      hibernate:
        format_sql: true
    show-sql: true
```

<div id='id3' />

## 3. [JPA,JDBC y SQL queries]

```java
@Repository
public interface CustomerRepository extends JpaRepository<Customer, Long> {
  // JPA
  Optional<Customer> findCustomerByFirstName(String firstName);
  List<Customer> findCustomersByFirstNameContains(String contain);

  // JPQL
  @Query("select c from Customer c where c.email = ?1")
  Optional<Customer> getCustomerByEmailAddress(String email);

  @Query("select c.firstName from Customer c where c.email = :email")
  String getCustomerFirstNameByEmailAddress(String email);

  // Query nativo
  @Query(
          value = "select * from customer where email = :emailAddress",
          nativeQuery = true
  )
  Optional<Customer> getCustomerFirstNameByEmailAddressNative(@Param("emailAddress") String email);


  @Modifying
  @Transactional
  @Query(
          value = "update customer set first_name = ?1 where email = ?2",
          nativeQuery = true
  )
  void updateCustomerNameByEmail(String name, String email);
}
```
