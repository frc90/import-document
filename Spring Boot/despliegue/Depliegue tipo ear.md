
Para desplegar una aplicación Spring Boot como un ``.ear`` en WildFly 32, necesitas una estructura de proyecto multi-módulo Maven con al menos dos módulos:

Módulo WAR: tu app Spring Boot (packaging war).
Módulo EAR: el empaquetador (packaging ear), que incluye el WAR.
Spring Boot no está pensado para EAR, pero puedes hacerlo siguiendo estos pasos:


1. Estructura de carpetas recomendada

springboot-wildfly-ear-app/

```
│
├── webapp/         # Tu app Spring Boot (WAR)
│   └── pom.xml
│
└── ear/            # Empaquetador EAR
    └── pom.xml
└── pom.xml         # (parent)
```

2. POM del módulo WAR (pom.xml)
Debe tener <packaging>war</packaging>.
Hereda del parent.
No uses el plugin spring-boot-maven-plugin para generar el ejecutable, solo para dependencias.

3. POM del módulo EAR (pom.xml)

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>springboot-wildfly-ear-app</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <artifactId>springboot-wildfly-ear</artifactId>
    <packaging>ear</packaging>
    <dependencies>
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>springboot-wildfly-webapp</artifactId>
            <version>1.0-SNAPSHOT</version>
            <type>war</type>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-ear-plugin</artifactId>
                <version>3.3.0</version>
                <configuration>
                    <modules>
                        <webModule>
                            <groupId>com.example</groupId>
                            <artifactId>springboot-wildfly-webapp</artifactId>
                            <contextRoot>/</contextRoot>
                        </webModule>
                    </modules>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```


1. POM del parent (pom.xml raíz)

```xml
<project >
    <modules>
        <module>webapp</module>
        <module>ear</module>
    </modules>
</project>
```


1. Cambios en tu aplicación
- Hereda de SpringBootServletInitializer (ya lo tienes).
- No uses spring-boot-starter-tomcat como dependencia normal, debe ser <scope>provided</scope>.
- No uses spring-boot-maven-plugin para empaquetar el WAR (solo para dependencias).