# Configuracion para despliegue con wildfly


## Indice
- [1. Configuracion del archivo application.properties](#id1)
- [2. Dependencias archivo pom.xml](#id2)
- [3. Configuracion @SpringBootApplication](#id3)
- [4. Crear un archivo jboss-web.xml](#id4)
- [5. Configuracin del module.xml](#id5)
- [6. Configuracin del standalone.xml](#id6)



<div id="id1"/>

### 1. Configuracion del archivo application.properties

`application.properties`

```properties
server.port=8080
spring.main.banner-mode=off
spring.main.log-startup-info=false
logging.register-shutdown-hook=false

spring.datasource.jndi-name=java:jboss/datasources/[database_name]
spring.jpa.hibernate.ddl-auto=update
```

<div id="id2"/>

### 2. Dependencias archivo pom.xml

`pom.xml`

Ubicasion `pom.xml`

```xml
<packaging>war</packaging>
<name>demo1</name>
<description>Demo project for Spring Boot</description>
<url/>
<licenses>
    <license/>
</licenses>
<developers>
    <developer/>
</developers>
<scm>
    <connection/>
    <developerConnection/>
    <tag/>
    <url/>
</scm>
<properties>
    <java.version>17</java.version>
    <maven.test.skip>true</maven.test.skip>

    <deploy.wildfly.host>127.0.0.1</deploy.wildfly.host>
    <deploy.wildfly.port>9990</deploy.wildfly.port>
    <deploy.wildfly.username>admin</deploy.wildfly.username>
    <deploy.wildfly.password>wildfly</deploy.wildfly.password>
</properties>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-logging</artifactId>
            </exclusion>
        </exclusions>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <!-- for wildfly deployment -->
        <exclusions>
            <exclusion>
                <groupId>ch.qos.logback</groupId>
                <artifactId>logback-classic</artifactId>
            </exclusion>

            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-tomcat</artifactId>
            </exclusion>
            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-logging</artifactId>
            </exclusion>
        </exclusions>
        <!-- end wildfly deploy -->
    </dependency>

    <!-- for wildfly deployment -->
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>4.0.1</version>
        <scope>provided</scope>
    </dependency>
    <!-- end wildfly deploy -->

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>

        <exclusions>
            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-logging</artifactId>
            </exclusion>
        </exclusions>
    </dependency>

    <!-- Swagger -->
    <dependency>
        <groupId>org.springdoc</groupId>
        <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
        <version>2.8.5</version>
        <exclusions>
            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-logging</artifactId>
            </exclusion>
        </exclusions>
    </dependency>

    <!-- Mapper -->
    <dependency>
        <groupId>org.modelmapper</groupId>
        <artifactId>modelmapper</artifactId>
        <version>3.1.1</version>
        <scope>compile</scope>
    </dependency>
</dependencies>

<plugins>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <configuration>
            <annotationProcessorPaths>
                <path>
                    <groupId>org.projectlombok</groupId>
                    <artifactId>lombok</artifactId>
                </path>
            </annotationProcessorPaths>
        </configuration>
    </plugin>
    <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
            <excludes>
                <exclude>
                    <groupId>org.projectlombok</groupId>
                    <artifactId>lombok</artifactId>
                </exclude>
            </excludes>
        </configuration>
    </plugin>

    <!-- wildfly -->
    <plugin>
        <groupId>org.wildfly.plugins</groupId>
        <artifactId>wildfly-maven-plugin</artifactId>
        <version>4.0.0.Final</version>

        <executions>
            <execution>
                <phase>install</phase>
                <goals>
                    <goal>deploy</goal>
                </goals>
            </execution>
        </executions>

        <configuration>
            <filename>${project.build.finalName}.war</filename>
            <hostname>${deploy.wildfly.host}</hostname>
            <port>${deploy.wildfly.port}</port>
            <username>${deploy.wildfly.username}</username>
            <password>${deploy.wildfly.password}</password>
        </configuration>
    </plugin>
</plugins>
```

<div id="id3"/>

### 3. Configuracion @SpringBootApplication

`@SpringBootApplication`

En el punto de inicio de la app, extiende de `SpringBootServletInitializer`

```java
@SpringBootApplication
public class Demo1Application extends SpringBootServletInitializer {

	@Override
	protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
		return builder.sources(Demo1Application.class);
	}

	public static void main(String[] args) {
		SpringApplication.run(Demo1Application.class, args);
	}
}
```

<div id="id4"/>

### 4. Crear un archivo jboss-web.xml

`jboss-web.xml`

crear el archivo `src/main/webapp/WEB-INF/jboss-web.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<jboss-web
        xmlns="http://www.jboss.com/xml/ns/javaee"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.jboss.com/xml/ns/javaee
        http://www.jboss.org/j2ee/schema/jboss-web_5_1.xsd">
    <context-root>/</context-root>
</jboss-web>
```

<div id="id5"/>

### 5. Configuracin del module.xml

`module.xml`

Importante en la direccion `wildfly-36.0.1.Final\modules\system\layers\base\org` se crea `postgresql\main` y dentro se copia el archivo de `postgresql-42.7.5.jar`
de postgrest, importante, tiene que tener el mismo nombre el archivo y la configuracion en el resources>resource-root

```xml
<module xmlns="urn:jboss:module:1.9" name="org.postgresql">
    <resources>
        <resource-root path="postgresql-42.7.5.jar"/>
    </resources>
    <dependencies>
        <module name="javax.api"/>
        <module name="javax.transaction.api"/>
        <module name="javax.api"/>
    </dependencies>
</module>
```

<div id="id6"/>

### 6. Configuracin del standalone.xml

`standalone.xml`

En la direccion `wildfly-36.0.1.Final\standalone\configuration` en archivo `standalone.xml` en el datasource copiar la conexion con la base de datos 

```xml
<datasource jta="true" jndi-name="java:jboss/datasources/testing" pool-name="TestingPool" enabled="true" use-java-context="true" use-ccm="true">
    <connection-url>jdbc:postgresql://localhost:5432/testing</connection-url>
    <driver>postgresql</driver>
    <security user-name="postgres" password="postgres"/>
</datasource>
```
