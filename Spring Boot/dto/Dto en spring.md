
<img src="img/dto.png">

Para crear tu propio mapper en Java y evitar depender de librerías de terceros, hay varias estrategias que podés seguir. La elección dependerá de la complejidad de tus objetos y de cuánta automatización quieras implementar. Aquí te presento las opciones principales, desde la más manual hasta soluciones que utilizan reflexión para mayor dinamismo:

---

### **1. Mapeo Manual (El enfoque más simple y directo)**

Este es el método más básico y fácil de entender. Simplemente creas un método o una clase que toma un objeto de entrada y construye un nuevo objeto de salida, asignando las propiedades una por una.

**Cuándo usarlo:**
* Cuando tenés un número limitado de objetos para mapear.
* Cuando tus objetos tienen estructuras muy diferentes y no hay un patrón claro para la automatización.
* Cuando la legibilidad y el control explícito son primordiales.

**Ejemplo:**

```java
// Objeto de Origen
public class UsuarioEntidad {
    private String id;
    private String nombre;
    private String email;

    // Getters y Setters
    public String getId() { return id; }
    public void setId(String id) { this.id = id; }
    public String getNombre() { return nombre; }
    public void setNombre(String nombre) { this.nombre = nombre; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
}

// Objeto de Destino (DTO - Data Transfer Object)
public class UsuarioDTO {
    private String identificador;
    private String nombreCompleto;
    private String correoElectronico;

    // Getters y Setters
    public String getIdentificador() { return identificador; }
    public void setIdentificador(String identificador) { this.identificador = identificador; }
    public String getNombreCompleto() { return nombreCompleto; }
    public void setNombreCompleto(String nombreCompleto) { this.nombreCompleto = nombreCompleto; }
    public String getCorreoElectronico() { return correoElectronico; }
    public void setCorreoElectronico(String correoElectronico) { this.correoElectronico = correoElectronico; }
}

// Clase Mapper Manual
public class UsuarioMapperManual {

    public UsuarioDTO toDTO(UsuarioEntidad entidad) {
        if (entidad == null) {
            return null;
        }
        UsuarioDTO dto = new UsuarioDTO();
        dto.setIdentificador(entidad.getId());
        dto.setNombreCompleto(entidad.getNombre());
        dto.setCorreoElectronico(entidad.getEmail());
        return dto;
    }

    public UsuarioEntidad toEntidad(UsuarioDTO dto) {
        if (dto == null) {
            return null;
        }
        UsuarioEntidad entidad = new UsuarioEntidad();
        entidad.setId(dto.getIdentificador());
        entidad.setNombre(dto.getNombreCompleto());
        entidad.setEmail(dto.getCorreoElectronico());
        return entidad;
    }
}
```

---

### **2. Mapeo con Interfaces Funcionales y Programación Lambda (Para un código más conciso)**

Podés usar interfaces funcionales para definir contratos de mapeo y luego implementar esos contratos con expresiones lambda. Esto es útil si querés una estructura más modular para tus mappers.

**Cuándo usarlo:**
* Cuando querés un código más conciso y expresivo.
* Para definir claramente la operación de mapeo.
* Si tus objetos no tienen una gran cantidad de campos y el mapeo es relativamente directo.

**Ejemplo:**

```java
import java.util.function.Function;

// Misma UsuarioEntidad y UsuarioDTO que en el ejemplo anterior

public class UsuarioMapperFuncional {

    // Definimos un mapeador de Entidad a DTO
    public static final Function<UsuarioEntidad, UsuarioDTO> ENTITY_TO_DTO_MAPPER = entidad -> {
        if (entidad == null) {
            return null;
        }
        UsuarioDTO dto = new UsuarioDTO();
        dto.setIdentificador(entidad.getId());
        dto.setNombreCompleto(entidad.getNombre());
        dto.setCorreoElectronico(entidad.getEmail());
        return dto;
    };

    // Definimos un mapeador de DTO a Entidad
    public static final Function<UsuarioDTO, UsuarioEntidad> DTO_TO_ENTITY_MAPPER = dto -> {
        if (dto == null) {
            return null;
        }
        UsuarioEntidad entidad = new UsuarioEntidad();
        entidad.setId(dto.getIdentificador());
        entidad.setNombre(dto.getNombreCompleto());
        entidad.setEmail(dto.getCorreoElectronico());
        return entidad;
    };

    // Métodos de utilidad para aplicar los mapeadores
    public UsuarioDTO mapToDTO(UsuarioEntidad entidad) {
        return ENTITY_TO_DTO_MAPPER.apply(entidad);
    }

    public UsuarioEntidad mapToEntidad(UsuarioDTO dto) {
        return DTO_TO_ENTITY_MAPER.apply(dto);
    }
}
```

---

### **3. Mapeo con Reflexión (Para mayor automatización)**

La reflexión te permite inspeccionar y manipular clases, métodos y campos en tiempo de ejecución. Esto es útil si querés automatizar el mapeo de propiedades con el mismo nombre o patrones de nombres similares. Es más complejo y tiene un rendimiento ligeramente inferior al mapeo manual, pero reduce el código repetitivo.

**Cuándo usarlo:**
* Cuando tenés muchas clases con estructuras similares que necesitan ser mapeadas.
* Cuando querés un alto grado de automatización y estás dispuesto a aceptar la complejidad y el ligero impacto en el rendimiento.
* Cuando las propiedades a mapear tienen nombres idénticos o siguen una convención de nombres clara.

**Consideraciones:**
* **Rendimiento:** La reflexión es más lenta que el acceso directo a los campos. Para aplicaciones de alto rendimiento, esto podría ser un factor.
* **Manejo de errores:** Es más propenso a `NoSuchMethodException` o `IllegalAccessException` si los nombres de los campos no coinciden o no tenés los permisos adecuados.
* **Complejidad:** El código es más complejo de escribir y depurar.

**Ejemplo (Mapeo de propiedades con nombres idénticos):**

```java
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

// Usaremos las mismas clases UsuarioEntidad y UsuarioDTO, pero para el ejemplo
// simplificaremos UsuarioDTO para tener nombres de campo idénticos:
// public class UsuarioDTO {
//     private String id;
//     private String nombre;
//     private String email;
//     // Getters y Setters
// }

public class GenericMapper {

    public static <S, T> T map(S source, Class<T> targetClass)
            throws InstantiationException, IllegalAccessException,
            NoSuchMethodException, InvocationTargetException {

        if (source == null) {
            return null;
        }

        T target = targetClass.getDeclaredConstructor().newInstance();

        for (Field sourceField : source.getClass().getDeclaredFields()) {
            try {
                // Hacemos el campo accesible por si es privado
                sourceField.setAccessible(true);
                Object value = sourceField.get(source);

                // Intentamos encontrar el campo correspondiente en la clase destino
                Field targetField = targetClass.getDeclaredField(sourceField.getName());
                targetField.setAccessible(true);
                targetField.set(target, value);

            } catch (NoSuchFieldException e) {
                // Opcional: Manejar campos que no existen en el destino
                System.out.println("Advertencia: Campo '" + sourceField.getName() +
                                   "' no encontrado en la clase destino " + targetClass.getName());
            }
        }
        return target;
    }

    // Un mapeador más avanzado que usa getters y setters
    public static <S, T> T mapUsingGettersSetters(S source, Class<T> targetClass)
            throws InstantiationException, IllegalAccessException,
            NoSuchMethodException, InvocationTargetException {

        if (source == null) {
            return null;
        }

        T target = targetClass.getDeclaredConstructor().newInstance();

        for (Method sourceMethod : source.getClass().getMethods()) {
            if (sourceMethod.getName().startsWith("get") && sourceMethod.getParameterCount() == 0) {
                try {
                    String fieldName = sourceMethod.getName().substring(3); // Remove "get"
                    fieldName = Character.toLowerCase(fieldName.charAt(0)) + fieldName.substring(1);

                    Object value = sourceMethod.invoke(source);

                    String setterName = "set" + sourceMethod.getName().substring(3);
                    Method targetSetter = targetClass.getMethod(setterName, sourceMethod.getReturnType());
                    targetSetter.invoke(target, value);

                } catch (NoSuchMethodException e) {
                    // Opcional: Manejar si no hay un setter correspondiente en la clase destino
                    System.out.println("Advertencia: No se encontró setter para '" + sourceMethod.getName() +
                                       "' en la clase destino " + targetClass.getName());
                }
            }
        }
        return target;
    }
}
```

**Ejemplo de uso del `GenericMapper`:**

```java
// Para que GenericMapper funcione con nombres de campo idénticos
public class UsuarioDTO {
    private String id;
    private String nombre;
    private String email;

    public String getId() { return id; }
    public void setId(String id) { this.id = id; }
    public String getNombre() { return nombre; }
    public void setNombre(String nombre) { this.nombre = nombre; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
}

// En algún lugar de tu código:
// UsuarioEntidad entidad = new UsuarioEntidad();
// entidad.setId("123");
// entidad.setNombre("Juan Perez");
// entidad.setEmail("juan@example.com");
//
// try {
//     UsuarioDTO dto = GenericMapper.map(entidad, UsuarioDTO.class); // Usando mapeo directo de campos
//     // O si prefieres:
//     // UsuarioDTO dto = GenericMapper.mapUsingGettersSetters(entidad, UsuarioDTO.class);
//     System.out.println("DTO ID: " + dto.getId());
//     System.out.println("DTO Nombre: " + dto.getNombre());
//     System.out.println("DTO Email: " + dto.getEmail());
// } catch (InstantiationException | IllegalAccessException |
//          NoSuchMethodException | InvocationTargetException e) {
//     e.printStackTrace();
// }
```

---

### **4. Mapeo con Anotaciones (Para mayor configuración y control con reflexión)**

Si querés un control más fino sobre el mapeo sin escribir todo a mano, podés combinar la reflexión con anotaciones personalizadas. Esto te permitiría, por ejemplo, indicar que un campo en el origen debe mapearse a un campo con un nombre diferente en el destino.

**Cuándo usarlo:**
* Cuando necesitás un alto grado de automatización pero también flexibilidad para manejar diferencias en los nombres de los campos o transformaciones específicas.
* Cuando querés que las reglas de mapeo estén directamente en las clases involucradas.

**Ejemplo (conceptual):**

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;

// Anotación personalizada para mapeo
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface MapField {
    String value() default ""; // Nombre del campo en el destino
    boolean ignore() default false; // Ignorar este campo
}

// Objeto de Destino con anotaciones
public class UsuarioDTOAnotado {
    @MapField(value = "id") // Mapea el campo 'id' de la entidad a 'identificador' en el DTO
    private String identificador;
    @MapField(value = "nombre")
    private String nombreCompleto;
    @MapField(value = "email")
    private String correoElectronico;
    @MapField(ignore = true) // Ignorar este campo si existe en el origen
    private String algunCampoExtra;

    // Getters y Setters
    public String getIdentificador() { return identificador; }
    public void setIdentificador(String identificador) { this.identificador = identificador; }
    public String getNombreCompleto() { return nombreCompleto; }
    public void setNombreCompleto(String nombreCompleto) { this.nombreCompleto = nombreCompleto; }
    public String getCorreoElectronico() { return correoElectronico; }
    public void setCorreoElectronico(String correoElectronico) { this.correoElectronico = correoElectronico; }
    public String getAlgunCampoExtra() { return algunCampoExtra; }
    public void setAlgunCampoExtra(String algunCampoExtra) { this.algunCampoExtra = algunCampoExtra; }
}

public class AnnotationMapper {

    public static <S, T> T map(S source, Class<T> targetClass)
            throws InstantiationException, IllegalAccessException,
            NoSuchMethodException, InvocationTargetException {

        if (source == null) {
            return null;
        }

        T target = targetClass.getDeclaredConstructor().newInstance();

        for (Field targetField : targetClass.getDeclaredFields()) {
            MapField mapAnnotation = targetField.getAnnotation(MapField.class);

            if (mapAnnotation != null && mapAnnotation.ignore()) {
                continue; // Ignorar campos marcados para ser ignorados
            }

            String sourceFieldName = targetField.getName();
            if (mapAnnotation != null && !mapAnnotation.value().isEmpty()) {
                sourceFieldName = mapAnnotation.value(); // Usar el nombre especificado en la anotación
            }

            try {
                Field sourceField = source.getClass().getDeclaredField(sourceFieldName);
                sourceField.setAccessible(true);
                targetField.setAccessible(true);
                targetField.set(target, sourceField.get(source));
            } catch (NoSuchFieldException e) {
                // Opcional: Manejar campos que no existen en el origen
                System.out.println("Advertencia: Campo origen '" + sourceFieldName +
                                   "' no encontrado para el campo destino '" + targetField.getName() +
                                   "' en " + source.getClass().getName());
            }
        }
        return target;
    }
}
```

---

### **Consideraciones Finales**

* **Complejidad vs. Reusabilidad:** Cuanto más quieras automatizar y generalizar tu mapper, más complejo se volverá el código subyacente (especialmente con reflexión). Evaluá si el esfuerzo de crear un mapper genérico vale la pena para tu caso de uso.
* **Rendimiento:** Para la mayoría de las aplicaciones, la diferencia de rendimiento entre el mapeo manual y la reflexión es insignificante. Sin embargo, si estás mapeando millones de objetos por segundo en un entorno de alto rendimiento, la reflexión podría ser un cuello de botella.
* **Mantenimiento:** El mapeo manual es el más fácil de mantener cuando los modelos cambian significativamente. Los mappers basados en reflexión pueden requerir más atención si los nombres de los campos o las convenciones cambian.
* **Transformaciones Personalizadas:** Para transformaciones más complejas (por ejemplo, combinar varios campos de origen en uno de destino, o formatear fechas), el mapeo manual o un mapeador con lógica explícita serán más fáciles de implementar que intentar hacerlo genéricamente con reflexión.
* **Inmutabilidad:** Si tus objetos son inmutables (todos los campos son `final` y se construyen a través del constructor), el mapeo se vuelve más complejo, ya que no podés usar setters. Tendrías que pasar los valores directamente al constructor del objeto de destino. Esto generalmente requerirá un enfoque más manual o constructores parametrizados en tu mapper.

Elegí el enfoque que mejor se adapte a tus necesidades. Si solo necesitas mapear unos pocos objetos y tenés control total sobre sus estructuras, el mapeo manual es probablemente la opción más clara y segura. Si estás buscando algo más automatizado para muchos objetos con estructuras similares, la reflexión o incluso anotaciones pueden ser una buena solución, pero con la advertencia de su mayor complejidad.