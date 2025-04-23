### 🔧 Pasos para instalar **WildFly** y **ActiveMQ**:

| Herramienta  | Estado    | Comentario                                      |
| ------------ | --------- | ----------------------------------------------- |
| **WildFly**  | ❌ No está | Necesario para correr el backend Java EE        |
| **ActiveMQ** | ❌ No está | Lo usaremos más adelante para la mensajería JMS |

---

## 🎯 ¿Qué sigue?

Vamos a instalar **WildFly**, que es lo primero que necesitamos para correr nuestro backend Java. Después armamos el proyecto en Maven.

---

## 🛠️ Paso 1: Instalar WildFly en Windows

### 📥 1. Descargar WildFly
- Andá a la web oficial:  
  👉 [https://www.wildfly.org/downloads/](https://www.wildfly.org/downloads/)
- Bajá la versión más reciente (por ejemplo `WildFly 30.0.1 Final`).

### 📂 2. Extraer el ZIP
- Extraelo en una carpeta fácil, por ejemplo:  
  `C:\wildfly`

### ⚙️ 3. Probar que funciona
Abrí PowerShell y ejecutá:
```powershell
cd C:\wildfly\bin
.\standalone.bat
```

> Esto levanta el servidor. Si ves logs con `WildFly Full <versión> started`, entonces ¡funciona!
