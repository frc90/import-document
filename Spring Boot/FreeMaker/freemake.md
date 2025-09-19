# 📋 Composición de Plantillas con FreeMarker

## 💡 El Problema y la Solución

El problema que presentas es cómo componer un **correo electrónico** o **SMS** completo a partir de partes reutilizables (**header**, **body**, **footer**).  
En tu diagrama de base de datos, cada registro en la tabla `templates` representa una plantilla completa, lo cual dificulta la reutilización.

La **solución** es utilizar el **motor de plantillas FreeMarker** para manejar la composición directamente en el código.  
Guardaremos las plantillas **parciales** (`header`, `footer`) y una plantilla **maestra** en la base de datos.  
Esta plantilla maestra contendrá etiquetas especiales de FreeMarker para **incluir** las plantillas parciales.

---

## 🛠️ Implementación en Spring Boot

Para implementar esto, necesitamos **3 elementos principales**:

1. **Datos en la Base de Datos**:  
   Las plantillas se guardan con su contenido FreeMarker.
2. **Un `TemplateService`**:  
   El servicio principal que recupera la plantilla maestra de la base de datos y la procesa con FreeMarker.
3. **Un `DatabaseTemplateLoader`**:  
   Un componente personalizado que le enseña a FreeMarker a buscar las plantillas incluidas (`<#include ...>`) directamente en la base de datos, en lugar de en archivos del sistema.

---

## 1. Estructura de las Plantillas en la Base de Datos

### Plantilla **header**
```txt
name: header
template_content: <html><head>...</head><body><header>...</header>


### Plantilla **footer**

```txt
name: footer
template_content: <footer>...</body></html>
```

### Plantilla **pedido\_enviado\_final** *(plantilla maestra)*

```txt
name: pedido_enviado_final
template_content:

<#include "header.ftl">

<p>¡Hola ${recipient_name}!</p>
<p>Tu pedido ${order_number} ha sido enviado.</p>

<#include "footer.ftl">
```

> **Nota:** La sintaxis `<#include "nombre.ftl">` le dice a FreeMarker que busque otra plantilla llamada `nombre.ftl`.
> El `.ftl` es un convenio que manejaremos en el código.

---

## 2. Código Java

### **TemplateService.java**

```java
package uy.com.antel.modulo_comunicacion.infrastructure.services;

import freemarker.template.Configuration;
import freemarker.template.Template;
import freemarker.template.TemplateException;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import uy.com.antel.modulo_comunicacion.domain.entities.TemplateEntity;
import uy.com.antel.modulo_comunicacion.domain.exceptions.ResourceNotFoundException;
import uy.com.antel.modulo_comunicacion.infrastructure.repositories.TemplateRepository;

import java.io.IOException;
import java.io.StringReader;
import java.io.StringWriter;
import java.util.Map;
import java.util.Optional;

@Service
public class TemplateService {

    private final TemplateRepository templateRepository;
    private final Configuration freeMarkerConfiguration;

    @Autowired
    public TemplateService(TemplateRepository templateRepository, Configuration freeMarkerConfiguration) {
        this.templateRepository = templateRepository;
        this.freeMarkerConfiguration = freeMarkerConfiguration;
    }

    /**
     * Procesa una plantilla final por su nombre y un mapa de variables,
     * ensamblando todas las partes de la plantilla usando FreeMarker.
     *
     * @param templateName El nombre de la plantilla final (ej. "pedido_enviado_final").
     * @param variables Un mapa con las variables para la plantilla (ej. "recipient_name", "order_number").
     * @return El contenido final del mensaje listo para ser enviado (HTML o texto).
     * @throws ResourceNotFoundException si la plantilla no se encuentra.
     * @throws IOException si ocurre un error al leer la plantilla.
     * @throws TemplateException si ocurre un error al procesar la plantilla.
     */
    public String getComposedTemplate(String templateName, Map<String, Object> variables)
            throws IOException, TemplateException {
        // Buscar la plantilla principal por su nombre.
        Optional<TemplateEntity> mainTemplateOpt = templateRepository.findByName(templateName);
        if (mainTemplateOpt.isEmpty()) {
            throw new ResourceNotFoundException("Template not found with name: " + templateName);
        }
        TemplateEntity mainTemplate = mainTemplateOpt.get();

        // Crear una configuración de FreeMarker en tiempo de ejecución.
        Configuration customConfig = new Configuration(Configuration.VERSION_2_3_32);
        customConfig.setTemplateLoader(new DatabaseTemplateLoader(templateRepository));

        // Analizar la plantilla principal (que contiene las etiquetas de inclusión)
        Template template = new Template(mainTemplate.getName(),
                new StringReader(mainTemplate.getTemplateContent()), customConfig);

        // Procesar la plantilla con las variables
        StringWriter writer = new StringWriter();
        template.process(variables, writer);

        return writer.toString();
    }
}
```

---

### **DatabaseTemplateLoader.java**

```java
package uy.com.antel.modulo_comunicacion.infrastructure.services;

import freemarker.cache.TemplateLoader;
import uy.com.antel.modulo_comunicacion.domain.entities.TemplateEntity;
import uy.com.antel.modulo_comunicacion.infrastructure.repositories.TemplateRepository;

import java.io.IOException;
import java.io.Reader;
import java.io.StringReader;

/**
 * Un cargador de plantillas personalizado para FreeMarker que recupera las plantillas de la base de datos.
 * Esto permite que <#include "nombre.ftl"> funcione.
 */
public class DatabaseTemplateLoader implements TemplateLoader {

    private final TemplateRepository templateRepository;

    public DatabaseTemplateLoader(TemplateRepository templateRepository) {
        this.templateRepository = templateRepository;
    }

    @Override
    public Object findTemplateSource(String name) throws IOException {
        // FreeMarker espera que el nombre de la plantilla termine en .ftl.
        // Lo removemos para buscar en la base de datos por el nombre limpio.
        if (name.endsWith(".ftl")) {
            name = name.substring(0, name.length() - 4);
        }
        // Buscar la plantilla por su nombre en la base de datos
        return templateRepository.findByName(name).orElse(null);
    }

    @Override
    public long getLastModified(Object templateSource) {
        // En un entorno de producción, podrías usar un campo de última modificación.
        // Para este ejemplo, simplemente devolvemos la hora actual.
        return System.currentTimeMillis();
    }

    @Override
    public Reader getReader(Object templateSource, String encoding) throws IOException {
        // El templateSource es el objeto TemplateEntity que recuperamos antes.
        TemplateEntity templateEntity = (TemplateEntity) templateSource;
        return new StringReader(templateEntity.getTemplateContent());
    }

    @Override
    public void closeTemplateSource(Object templateSource) throws IOException {
        // No es necesario cerrar nada para un StringReader.
    }
}
```

---

## 3. Uso del Servicio

Para generar el contenido final, solo necesitas inyectar el `TemplateService` en tu controlador o cualquier otro servicio y llamarlo con el nombre de la **plantilla maestra** y las **variables necesarias**.

```java
@Autowired
private TemplateService templateService;

public void sendEmail() {
    Map<String, Object> variables = new HashMap<>();
    variables.put("recipient_name", "Juan");
    variables.put("order_number", "12345");
    variables.put("subject", "Tu pedido ha sido enviado");

    try {
        String finalHtmlContent =
            templateService.getComposedTemplate("pedido_enviado_final", variables);

        // finalHtmlContent contiene el HTML completo
        System.out.println(finalHtmlContent);
    } catch (Exception e) {
        // Manejo de errores
        e.printStackTrace();
    }
}
```

```

Si quieres, puedo hacer que el **Markdown** sea aún más atractivo:  
- Añadir **diagramas de flujo** para explicar cómo FreeMarker arma las plantillas.  
- Incluir una **tabla** con la estructura de la base de datos.  
- Crear un ejemplo visual del **header**, **body** y **footer**.  

¿Quieres que prepare esa versión mejorada?
```
