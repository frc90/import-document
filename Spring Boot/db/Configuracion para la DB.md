## Indice
- [application.properties](#id1)
- [application.yml](#id2)
- [gradle](#id3)
- [JpaConfig.java](#id4)

-----

### 1\. Configuración Básica de `application.properties` y `application.yml`

Spring Boot es muy flexible y soporta ambos formatos para la configuración. La principal diferencia es la sintaxis: `application.properties` usa un formato clave-valor plano, mientras que `application.yml` usa YAML, que es más jerárquico y a menudo más legible para configuraciones complejas.

#### **1.1. Configuración Básica con `application.properties`**

El archivo `application.properties` se encuentra típicamente en `src/main/resources`.

**Ejemplo Básico:**

```properties
# Nombre de la aplicación (opcional, pero útil)
spring.application.name=mi-aplicacion-springboot

# Puerto del servidor web integrado de Spring Boot (por defecto es 8080)
server.port=8080

# Configuración básica de logging
logging.level.root=INFO
logging.level.com.example.miaplicacion=DEBUG
```

#### **1.2. Configuración Básica con `application.yml`**

El archivo `application.yml` también se encuentra en `src/main/resources`.

**Ejemplo Básico:**

```yaml
# Nombre de la aplicación
spring:
  application:
    name: mi-aplicacion-springboot

# Puerto del servidor web integrado
server:
  port: 8080

# Configuración básica de logging
logging:
  level:
    root: INFO
    com.example.miaplicacion: DEBUG
```

Como puedes ver, `application.yml` utiliza indentación para representar la jerarquía, lo que puede ser más limpio para configuraciones anidadas.

-----

### 2\. Configuración para Diferentes Tipos de Bases de Datos

Spring Boot facilita enormemente la configuración de bases de datos. Necesitarás el *driver* JDBC correspondiente en tus dependencias de Maven o Gradle.

Vamos a ver ejemplos para algunas bases de datos populares.

#### **2.1. Configuración para H2 (Base de Datos en Memoria o Basada en Archivo)**

H2 es excelente para desarrollo local o pruebas debido a su facilidad de uso.

**Dependencia (pom.xml para Maven):**

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```

**`application.properties` para H2 (en memoria):**

```properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.h2.console.enabled=true # Habilita la consola H2 (útil para desarrollo)
spring.h2.console.path=/h2-console
```

**`application.yml` para H2 (en memoria):**

```yaml
spring:
  datasource:
    url: jdbc:h2:mem:testdb
    driverClassName: org.h2.Driver
    username: sa
    password:
  jpa:
    database-platform: org.hibernate.dialect.H2Dialect
  h2:
    console:
      enabled: true
      path: /h2-console
```

Para H2 basado en archivo (persistencia entre reinicios): cambia `jdbc:h2:mem:testdb` a `jdbc:h2:file:./data/testdb`.

#### **2.2. Configuración para PostgreSQL**

**Dependencia (pom.xml para Maven):**

```xml
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
```

**`application.properties` para PostgreSQL:**

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/mydatabase
spring.datasource.username=myuser
spring.datasource.password=mypassword
spring.datasource.driverClassName=org.postgresql.Driver
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.hibernate.ddl-auto=update # o create, create-drop, none
```

**`application.yml` para PostgreSQL:**

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/mydatabase
    username: myuser
    password: mypassword
    driverClassName: org.postgresql.Driver
  jpa:
    database-platform: org.hibernate.dialect.PostgreSQLDialect
    hibernate:
      ddl-auto: update # o create, create-drop, none
```

  * `spring.jpa.hibernate.ddl-auto`: Controla cómo Hibernate interactúa con el esquema de la base de datos al iniciar la aplicación.
      * `none`: No hace nada.
      * `update`: Actualiza el esquema existente.
      * `create`: Crea el esquema, eliminando el existente si lo hay.
      * `create-drop`: Crea el esquema al iniciar y lo elimina al cerrar.

#### **2.3. Configuración para MySQL**

**Dependencia (pom.xml para Maven):**

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.28</version> <scope>runtime</scope>
</dependency>
```

**`application.properties` para MySQL:**

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/mydatabase?useSSL=false&serverTimezone=UTC
spring.datasource.username=myuser
spring.datasource.password=mypassword
spring.datasource.driverClassName=com.mysql.cj.jdbc.Driver
spring.jpa.database-platform=org.hibernate.dialect.MySQL8Dialect
spring.jpa.hibernate.ddl-auto=update
```

**`application.yml` para MySQL:**

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydatabase?useSSL=false&serverTimezone=UTC
    username: myuser
    password: mypassword
    driverClassName: com.mysql.cj.jdbc.Driver
  jpa:
    database-platform: org.hibernate.dialect.MySQL8Dialect
    hibernate:
      ddl-auto: update
```

  * **`?useSSL=false&serverTimezone=UTC`**: Son parámetros importantes para la conexión a MySQL, especialmente en versiones recientes del conector. `useSSL=false` es común para entornos de desarrollo si no estás configurando SSL. `serverTimezone=UTC` ayuda con problemas de zona horaria.

-----

### 3\. Configuración con Múltiples Ambientes de Desarrollo (Perfiles)

Spring Boot utiliza el concepto de "perfiles" para permitirte tener configuraciones específicas para diferentes entornos (local, dev, test, prod, etc.). Esto es una de las características más potentes.

#### **Cómo funciona:**

1.  Creas archivos de configuración específicos para cada perfil con el formato `application-{profile}.properties` o `application-{profile}.yml`.
2.  Le indicas a Spring Boot qué perfil activar.

**Ejemplo de Estructura de Archivos:**

```
src/main/resources/
├── application.properties  (Configuración por defecto, aplicada a todos los perfiles a menos que se sobreescriba)
├── application-local.properties
├── application-dev.properties
├── application-prod.properties
└── application.yml         (Alternativa a application.properties)
├── application-local.yml
├── application-dev.yml
└── application-prod.yml
```

#### **3.1. Ejemplo con `application.properties` y Perfiles**

**`src/main/resources/application.properties` (Configuración por defecto):**

```properties
spring.application.name=mi-aplicacion-perfiles
server.port=8080 # Puerto por defecto

# Configuración de base de datos H2 en memoria por defecto
spring.datasource.url=jdbc:h2:mem:defaultdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.hibernate.ddl-auto=update
spring.h2.console.enabled=true
```

**`src/main/resources/application-local.properties` (Para desarrollo local con H2 persistente):**

```properties
server.port=8081 # Puerto diferente para evitar conflictos
spring.datasource.url=jdbc:h2:file:./data/localdb;DB_CLOSE_ON_EXIT=FALSE
spring.h2.console.path=/h2-console-local
logging.level.root=DEBUG # Más logging en local
```

**`src/main/resources/application-dev.properties` (Para entorno de desarrollo con PostgreSQL):**

```properties
server.port=8080
spring.datasource.url=jdbc:postgresql://dev-db.example.com:5432/dev_db
spring.datasource.username=devuser
spring.datasource.password=devpassword123
spring.datasource.driverClassName=org.postgresql.Driver
spring.jpa.hibernate.ddl-auto=update
logging.level.root=INFO
```

**`src/main/resources/application-prod.properties` (Para entorno de producción con PostgreSQL):**

```properties
server.port=8080
spring.datasource.url=jdbc:postgresql://prod-db.example.com:5432/prod_db
spring.datasource.username=produser
spring.datasource.password=${DB_PASSWORD_PROD} # Usando variable de entorno para seguridad
spring.datasource.driverClassName=org.postgresql.Driver
spring.jpa.hibernate.ddl-auto=none # No queremos que Hibernate modifique el esquema en prod
logging.level.root=ERROR # Solo errores críticos en prod
```

#### **3.2. Ejemplo con `application.yml` y Perfiles**

La sintaxis YAML para perfiles es muy elegante. Puedes definir perfiles dentro de un solo archivo `application.yml` o usar archivos separados como con `.properties`. Recomiendo archivos separados para mayor claridad en proyectos grandes.

**`src/main/resources/application.yml` (Configuración por defecto y perfiles en un solo archivo):**

```yaml
spring:
  application:
    name: mi-aplicacion-perfiles-yml
  profiles:
    active: local # Puedes activar un perfil por defecto aquí, aunque es mejor hacerlo por línea de comandos o variable de entorno
server:
  port: 8080

---
spring:
  config:
    activate:
      on-profile: local
server:
  port: 8081
logging:
  level:
    root: DEBUG
spring:
  datasource:
    url: jdbc:h2:file:./data/localdb;DB_CLOSE_ON_EXIT=FALSE
    driverClassName: org.h2.Driver
    username: sa
    password:
  h2:
    console:
      enabled: true
      path: /h2-console-local

---
spring:
  config:
    activate:
      on-profile: dev
server:
  port: 8080
logging:
  level:
    root: INFO
spring:
  datasource:
    url: jdbc:postgresql://dev-db.example.com:5432/dev_db
    username: devuser
    password: devpassword123
    driverClassName: org.postgresql.Driver
  jpa:
    hibernate:
      ddl-auto: update

---
spring:
  config:
    activate:
      on-profile: prod
server:
  port: 8080
logging:
  level:
    root: ERROR
spring:
  datasource:
    url: jdbc:postgresql://prod-db.example.com:5432/prod_db
    username: produser
    password: ${DB_PASSWORD_PROD}
    driverClassName: org.postgresql.Driver
  jpa:
    hibernate:
      ddl-auto: none
```

  * La clave `---` en YAML separa documentos, lo que permite definir configuraciones para diferentes perfiles en el mismo archivo.
  * `spring.config.activate.on-profile: [profile_name]` es la forma de indicar a qué perfil pertenece la configuración siguiente.

#### **3.3. Cómo Activar un Perfil**

Puedes activar un perfil de varias maneras:

  * **Línea de comandos (al ejecutar el JAR):**
    ```bash
    java -jar mi-aplicacion.jar --spring.profiles.active=dev
    ```
  * **Variable de entorno:**
    ```bash
    export SPRING_PROFILES_ACTIVE=prod
    java -jar mi-aplicacion.jar
    ```
  * **En `application.properties` o `application.yml` (usado con menos frecuencia, más para un perfil por defecto):**
    ```properties
    spring.profiles.active=local
    ```
    o
    ```yaml
    spring:
      profiles:
        active: local
    ```

**Pensamiento Crítico:** Siempre es una buena práctica no activar perfiles específicos en el `application.properties` principal, sino a través de variables de entorno o parámetros de línea de comandos. Esto le da más flexibilidad al entorno de despliegue para decidir qué configuración usar sin modificar el JAR.

-----

### 4\. Configuración con Variables de Entorno

Una práctica esencial para la seguridad y la flexibilidad es usar variables de entorno para datos sensibles (como contraseñas de bases de datos) o configuraciones que cambian frecuentemente entre despliegues.

Spring Boot tiene un orden de precedencia en la carga de propiedades, y las variables de entorno tienen una alta precedencia.

#### **Cómo funciona:**

Si tienes una propiedad en tu `application.properties` o `application.yml` como `${MI_VARIABLE}`, Spring Boot intentará resolverla de las siguientes fuentes (entre otras, en orden de precedencia):

1.  Argumentos de línea de comandos.
2.  Variables de entorno del sistema operativo.
3.  Propiedades del sistema Java (`System.getProperties()`).
4.  Archivos `application-{profile}.properties` / `application-{profile}.yml`.
5.  Archivos `application.properties` / `application.yml`.

**Ejemplo de Uso:**

Digamos que en tu `application-prod.properties` (o `application-prod.yml`) tienes:

```properties
spring.datasource.password=${DB_PASSWORD_PROD}
```

**Para ejecutar tu aplicación con esta variable:**

  * **En Linux/macOS:**
    ```bash
    export DB_PASSWORD_PROD="passwordSeguraDeProd"
    java -jar mi-aplicacion.jar --spring.profiles.active=prod
    ```
  * **En Windows (CMD):**
    ```cmd
    set DB_PASSWORD_PROD="passwordSeguraDeProd"
    java -jar mi-aplicacion.jar --spring.profiles.active=prod
    ```
  * **En Windows (PowerShell):**
    ```powershell
    $env:DB_PASSWORD_PROD="passwordSeguraDeProd"
    java -jar mi-aplicacion.jar --spring.profiles.active=prod
    ```

**Ventajas:**

  * **Seguridad:** Evita que las credenciales sensibles queden codificadas en tus archivos de configuración o en el repositorio de código.
  * **Flexibilidad:** Permite cambiar la configuración sin recompilar o modificar el JAR de la aplicación.
  * **DevOps:** Facilita la integración con herramientas de CI/CD y plataformas de despliegue que gestionan variables de entorno.

**Pensamiento Crítico y Seguridad:** Siempre utiliza variables de entorno para contraseñas, claves API y otros secretos en entornos que no sean de desarrollo local. Nunca los hardcodees en tus archivos de configuración de producción.

-----

### 5\. Configuración con Docker

Docker es fundamental para empaquetar y ejecutar aplicaciones de manera consistente en cualquier entorno. Integrar las configuraciones de Spring Boot con Docker es sencillo y aprovecha los conceptos que ya hemos visto.

#### **Pasos clave para Dockerizar una aplicación Spring Boot:**

1.  **Crear un JAR ejecutable:** Asegúrate de que tu proyecto Spring Boot pueda ser compilado en un JAR ejecutable. Maven o Gradle lo hacen por defecto.

    ```bash
    # Para Maven
    mvn clean package

    # Para Gradle
    ./gradlew clean build
    ```

    Esto generará un archivo `target/your-app-name.jar` (Maven) o `build/libs/your-app-name.jar` (Gradle).

2.  **Crear un `Dockerfile`:** Este archivo define cómo construir tu imagen Docker.

3.  **Construir la imagen Docker.**

4.  **Ejecutar el contenedor Docker.**

#### **Ejemplo Paso a Paso para Docker con Perfiles y Variables de Entorno**

Vamos a considerar que tenemos los perfiles `dev` y `prod` configurados como vimos anteriormente, utilizando PostgreSQL y variables de entorno para la contraseña de producción.

**5.1. `Dockerfile` Básico:**

Crea un archivo llamado `Dockerfile` en la raíz de tu proyecto.

```dockerfile
# Usa una imagen base oficial de OpenJDK para Java 17 (ajusta según tu versión de Java)
FROM openjdk:17-jdk-slim

# Establece el directorio de trabajo dentro del contenedor
WORKDIR /app

# Copia el JAR ejecutable de tu aplicación al contenedor
# Asume que el JAR se llama 'mi-aplicacion.jar'
COPY target/mi-aplicacion.jar mi-aplicacion.jar

# Expone el puerto en el que la aplicación Spring Boot se ejecutará
EXPOSE 8080

# Comando para ejecutar la aplicación cuando el contenedor se inicie
# Por defecto, ejecutará el perfil "default" si no se especifica otro
CMD ["java", "-jar", "mi-aplicacion.jar"]
```

**5.2. Construir la Imagen Docker:**

En la terminal, desde la raíz de tu proyecto (donde está el `Dockerfile` y la carpeta `target` con el JAR):

```bash
docker build -t mi-aplicacion-springboot .
```

  * `-t mi-aplicacion-springboot`: Asigna un nombre (tag) a tu imagen.
  * `.`: Indica que el `Dockerfile` está en el directorio actual.

**5.3. Ejecutar el Contenedor Docker:**

Aquí es donde entra la magia de los perfiles y las variables de entorno en Docker.

  * **Ejecutar en modo `dev` (con PostgreSQL de desarrollo):**

    ```bash
    docker run -p 8080:8080 -e SPRING_PROFILES_ACTIVE=dev mi-aplicacion-springboot
    ```

      * `-p 8080:8080`: Mapea el puerto 8080 del contenedor al puerto 8080 de tu máquina local.
      * `-e SPRING_PROFILES_ACTIVE=dev`: Establece la variable de entorno `SPRING_PROFILES_ACTIVE` dentro del contenedor a `dev`, activando el perfil `dev` de tu aplicación.

  * **Ejecutar en modo `prod` (con PostgreSQL de producción y variable de entorno para la contraseña):**

    ```bash
    docker run -p 8080:8080 -e SPRING_PROFILES_ACTIVE=prod -e DB_PASSWORD_PROD="passwordSeguraDeProd" mi-aplicacion-springboot
    ```

      * `-e DB_PASSWORD_PROD="passwordSeguraDeProd"`: Pasa la variable de entorno `DB_PASSWORD_PROD` con el valor real de la contraseña al contenedor.

**Consideraciones Adicionales para Docker:**

  * **`alpine` o `slim` images:** Usa imágenes base más pequeñas (como `openjdk:17-jdk-slim` o `openjdk:17-jdk-alpine`) para reducir el tamaño de tus imágenes Docker y mejorar los tiempos de despliegue y el consumo de recursos.

  * **`.dockerignore`:** Similar a `.gitignore`, crea un archivo `.dockerignore` para excluir archivos y directorios innecesarios (como `target/`, `.git/`, `README.md`) de tu contexto de construcción Docker, lo que acelera las construcciones.

    ```
    target/
    build/
    .mvn/
    .gradle/
    .git/
    .gitignore
    Dockerfile
    README.md
    *.iml
    .idea/
    ```

  * **Docker Compose:** Para entornos más complejos con múltiples servicios (aplicación, base de datos, Redis, etc.), **Docker Compose** es la herramienta ideal. Te permite definir y ejecutar aplicaciones multi-contenedor con un solo comando.

    **Ejemplo de `docker-compose.yml` para una app Spring Boot y PostgreSQL:**

    ```yaml
    version: '3.8'

    services:
      db:
        image: postgres:13-alpine
        restart: always
        environment:
          POSTGRES_DB: mydatabase_dev
          POSTGRES_USER: devuser
          POSTGRES_PASSWORD: devpassword123
        ports:
          - "5432:5432" # Solo para desarrollo local, no recomendado en producción para la DB
        volumes:
          - pgdata:/var/lib/postgresql/data

      app:
        build: . # Construye la imagen de la app desde el Dockerfile en el directorio actual
        ports:
          - "8080:8080"
        environment:
          SPRING_PROFILES_ACTIVE: dev # Activa el perfil dev
          # Para prod, esto cambiaría:
          # DB_PASSWORD_PROD: "tu_secreto_de_produccion"
        depends_on:
          - db # Asegura que la DB se inicie antes que la app
        restart: on-failure

    volumes:
      pgdata:
    ```

    Luego, puedes levantar todo con `docker-compose up -d`.

-----

### Resumen de Mejores Prácticas:

  * **Perfiles para Entornos:** Utiliza perfiles de Spring Boot para diferenciar configuraciones entre entornos.
  * **Variables de Entorno para Secretos:** Nunca hardcodees credenciales o información sensible. Usa variables de entorno.
  * **YAML para Claridad:** Considera `application.yml` para configuraciones más legibles y jerárquicas.
  * **Dockerizar tus Apps:** Empaqueta tus aplicaciones en contenedores Docker para garantizar la portabilidad y consistencia.
  * **Docker Compose para Multi-Servicio:** Usa Docker Compose para orquestar aplicaciones con múltiples servicios (base de datos, caché, etc.) en desarrollo.
  * **Optimización de Imagen Docker:** Utiliza imágenes base pequeñas (`-slim`, `-alpine`) y un `.dockerignore` para reducir el tamaño de tus imágenes.
  * **`ddl-auto=none` en Producción:** Evita que Hibernate modifique automáticamente el esquema de tu base de datos en producción. Usa migraciones de base de datos (Flyway, Liquibase) para gestionar cambios en el esquema de manera controlada.

Espero que esta explicación detallada te sea de gran utilidad. ¡Estoy listo para tu próxima pregunta o problema de desarrollo\!

----
<div id="id1"/>

ubicasion `src/main/resources` se crean un archivo puede ser:

`application.properties`

```properties
# H2 DB
spring.datasource.url=jdbc:h2:file:C:/temp/test
spring.datasource.username=sa
spring.datasource.password=
spring.datasource.driverClassName=org.h2.Driver
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect

# MySQL
spring.datasource.url=jdbc:mysql://localhost:3306/test
spring.datasource.username=dbuser
spring.datasource.password=dbpass
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect

# Oracle
spring.datasource.url=jdbc:oracle:thin:@localhost:1521:orcl
spring.datasource.username=dbuser
spring.datasource.password=dbpass
spring.datasource.driver-class-name=oracle.jdbc.OracleDriver
spring.jpa.database-platform=org.hibernate.dialect.Oracle10gDialect

# SQL Server
spring.datasource.url=jdbc:sqlserver://localhost;databaseName=springbootdb
spring.datasource.username=dbuser
spring.datasource.password=dbpass
spring.datasource.driverClassName=com.microsoft.sqlserver.jdbc.SQLServerDriver
spring.jpa.hibernate.dialect=org.hibernate.dialect.SQLServer2012Dialect


# Habilitar transacciones JTA en Spring Boot
spring.jta.enabled=true


# Configuración de JNDI - JTA
spring.datasource.jndi-name=java:/MySqlDS


#JBoss defined datasource using JNDI
spring.datasource.jndi-name = java:jboss/datasources/testDB
```

<div id="id2"/> 

`application.yml`

```yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/nombre_de_tu_base_de_datos?useSSL=false&serverTimezone=UTC
    username: tu_usuario
    password: tu_contraseña
    driver-class-name: com.mysql.cj.jdbc.Driver
```

```xml
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>8.0.33</version> 
</dependency>

<dependency>
  <groupId>org.postgresql</groupId>
  <artifactId>postgresql</artifactId>
  <scope>runtime</scope>
</dependency>
```

<div id="id3"/>

`gradle`


mysql

```gradle
implementation 'mysql:mysql-connector-java:8.0.33' // Usa una versión compatible
```

postgresql

```xgradleml
runtimeOnly 'org.postgresql:postgresql'
```

<div id="id4"/>

`JpaConfig.java`

```java
@Configuration
public class JpaConfig {
    @Bean(name = "h2DataSource")
    public DataSource h2DataSource()
    {
        DataSourceBuilder dataSourceBuilder = DataSourceBuilder.create();
        dataSourceBuilder.driverClassName("org.h2.Driver");
        dataSourceBuilder.url("jdbc:h2:file:C:/temp/test");
        dataSourceBuilder.username("sa");
        dataSourceBuilder.password("");
        return dataSourceBuilder.build();
    }
    @Bean(name = "mysqlDataSource")
    @Primary
    public DataSource mysqlDataSource()
    {
        DataSourceBuilder dataSourceBuilder = DataSourceBuilder.create();
        dataSourceBuilder.url("jdbc:mysql://localhost/testdb");
        dataSourceBuilder.username("dbuser");
        dataSourceBuilder.password("dbpass");
        return dataSourceBuilder.build();
    }
}
```