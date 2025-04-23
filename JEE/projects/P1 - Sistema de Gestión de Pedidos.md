## 🎯 Proyecto: Sistema de Gestión de Pedidos

Vamos a crear un proyecto completo desde cero, incluyendo:

✅ Backend (Java EE con JAX-RS, JPA, JMS y MDB)  
✅ Frontend (Angular para la interfaz del usuario)  
✅ Comunicación asíncrona con JMS (ActiveMQ)  
✅ Notificación por email (usando una librería Java)  
✅ Despliegue en local, y después en la nube (Azure)

---

## 🔄 ¿Qué haremos primero?

### **Paso 1: Backend (Java)**
> Porque es el corazón de la aplicación: maneja la lógica, la base de datos y la comunicación por colas JMS.

### **Paso 2: Frontend (Angular)**
> Porque se conecta al backend para crear pedidos desde el navegador.

### **Paso 3: Integración de JMS y MDB**
> Para procesar los pedidos de forma asíncrona.

### **Paso 4: Envío de correos electrónicos**
> Para notificar al usuario.

### **Paso 5: Despliegue en Azure (o local si preferís comenzar ahí)**
> Para poner todo online, como un pro.

---

## ✅ Paso 1 - Preparar el entorno de desarrollo

### 🔧 **Herramientas necesarias**
| Herramienta             | Para qué sirve                     |
| ----------------------- | ---------------------------------- |
| Java JDK 11+            | Para compilar y correr el backend  |
| Apache Maven            | Para construir el proyecto Java    |
| WildFly                 | Servidor Java EE donde desplegamos |
| ActiveMQ                | Para las colas JMS                 |
| IntelliJ IDEA o Eclipse | IDE para programar en Java         |
| Postman                 | Para probar las APIs               |
| Node.js + Angular CLI   | Para el frontend                   |