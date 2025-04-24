### ✅ **Solución 1: Eliminar y forzar la descarga de nuevo**

1. **Cerrá IntelliJ completamente.**
2. Andá a la carpeta:

```
C:\Users\tu_usuario\.m2\repository\com\microsoft\sqlserver\mssql-jdbc\
```
***NOTA: Mi caso***

```
C:\Users\frc90\.m2\repository\com\microsoft\sqlserver\mssql-jdbc
```


3. Borrá la carpeta `12.10.0.jre11` completa (o la versión que estés usando).
4. Luego, abrí una terminal y ejecutá:

```bash
mvn dependency:purge-local-repository -DreResolve=true
```

> Esto **limpia la dependencia corrupta** y **vuelve a bajarla desde cero**.

5. Reabrí IntelliJ y hacé **clic derecho sobre el proyecto → Maven → Reload Project**.

---

### ✅ **Solución 2: Agregar manualmente como librería (por si falla la anterior)**

1. Descargá el `.jar` directamente desde el repositorio de Maven Central:
   - Link directo:  
     [https://repo1.maven.org/maven2/com/microsoft/sqlserver/mssql-jdbc/12.10.0.jre11/](https://repo1.maven.org/maven2/com/microsoft/sqlserver/mssql-jdbc/12.10.0.jre11/)
2. Copiá el `.jar` a una carpeta dentro del proyecto, por ejemplo:  
   `lib/mssql-jdbc-12.10.0.jre11.jar`
3. En IntelliJ:  
   - `File > Project Structure > Modules > Dependencies > + > JARs or directories`
   - Seleccioná ese `.jar`

⚠️ **Ojo**: Si hacés esto, quitá temporalmente la dependencia de Maven en el `pom.xml`, o poné un comentario para evitar conflicto.

---

### 🔁 Verificación

Cuando vuelva a estar bien cargada, al tipear:

```xml
<property name="jakarta.persistence.jdbc.driver" value="com.microsoft.sqlserver.jdbc.SQLServerDriver"/>
```

en el XML, ya te debería **autocompletar `com.microsoft...`** y no más `sun.*`.

---