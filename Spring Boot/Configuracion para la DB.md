## Indice
- [application.properties](#id1)
- [application.yml](#id2)
- [gradle](#id3)
- [JpaConfig.java](#id4)



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
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.jpa.database-platform=org.hibernate.dialect.MySQL5InnoDBDialect

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