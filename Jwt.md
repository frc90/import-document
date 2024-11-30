# JASON Web Token (JWT)

**Índice**

1. [1. Que es JWT?](#id1)
2. [2. Token JWT](#id4)
3. [3. Firma de un token JWT](#id3)
4. [4. Token JWT seguro](#id4)
5. [5. Ciclo de vida de un Token JWT](#id5)

<div id='id1' />

## 1. Que es JWT?

Es un estándar qué está dentro del documento RFC 7519.

En el mismo se define un mecanismo para poder propagar entre dos partes, y de forma segura, la identidad de un determinado usuario, además con una serie de claims o privilegios.

<div id='id2' />

## 2. Token JWT

En la práctica, se trata de una cadena de texto que tiene tres partes codificadas en Base64, cada una de ellas separadas por un punto, como la que vemos:

```json
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

Podemos utilizar un [debugger online](https://jwt.io/) para decodificar ese token y visualizar su contenido. Si accedemos al mismo y pegamos dentro el texto completo, se nos mostrará lo que contiene.

Podemos ver el contenido del token sin necesidad de saber la clave con la cual se ha generado, aunque no podremos validarlo sin la misma.

### Un token tiene tres partes:

**_Header_**: encabezado dónde se indica, al menos, el algoritmo y el tipo de token, que en el caso del ejemplo anterior era el algoritmo HS256 y un token JWT.

**_Payload_**: donde aparecen los datos de usuario y privilegios, así como toda la información que queramos añadir, todos los datos que creamos convenientes.

**_Signature_**: una firma que nos permite verificar si el token es válido, y aquí es donde radica el quid de la cuestión, ya que si estamos tratando de hacer una comunicación segura entre partes y hemos visto que podemos coger cualquier token y ver su contenido con una herramienta sencilla, ¿dónde reside entonces la potencia de todo esto?

<div id='id3' />

## 3. Firma de un token JWT

La firma se construye de tal forma que vamos a poder verificar que el remitente es quien dice ser, y que el mensaje no se ha modificado por el camino.

Se construye como el HMACSHA256, que son las siglas de Hash-Based Message Authentication Code (Código de Autenticación de Mensajes), cifrado además con el algoritmo SHA de 256 bits. Se aplica esa función a:

- **Codificación en Base64 de header**.

- **Codificación en Base64 de payload**.

- **Un secreto**, establecido por la aplicación.

De esta forma, si alguien modifica el token por el camino, por ejemplo, inyectando alguna credencial o algún dato malicioso, entonces podríamos verificar que la comprobación de la firma no es correcta, por lo que no podemos confiar en el token recibido y deberíamos denegar la solicitud de recursos que nos haya realizado, ya sea para obtener datos o modificarlos.

Si lo que estamos requiriendo es que el usuario esté autenticado, deberíamos denegar esa petición, por lo que siempre que trabajemos con JWT deberíamos verificar con esa firma si el token es válido o no lo es.

## 4. Token JWT seguro

<div id='id4' />

Aunque el algoritmo nos permita verificar la firma, y, por tanto, confiar o no, tanto el encabezado como el cuerpo si llevan muchos datos van en abierto, ya que Base64 no es un cifrado, es simplemente una codificación que es muy fácilmente decodificable.

JWT nos invita siempre a que la comunicación entre partes se realice con **HTTPS** para encriptar el tráfico, de forma que, si alguien lo interceptara, el propio tráfico a través de **HTTP** sobre esos sockets SSL, cifrara toda la comunicación, la del token y todo lo demás, y así añadir esa posibilidad de seguridad.

<div id='id5' />

## 5. Ciclo de vida de un Token JWT

Como hemos visto, JWT no es un estándar de autenticación, sino que simplemente un estándar que nos permite hacer una comunicación entre dos partes de identidad de usuario. Con este proceso, además, podríamos implementar la autenticación de usuario de una manera que fuera relativamente segura.

<img src="img/jwt/Life cicle.png" alt="contenedor" />

1. Comenzaríamos desde el cliente, haciendo una petición POST para enviar el usuario y contraseña, y realizar el proceso de login.

2. Se comprobaría que ese usuario y su contraseña son correctos, y de serlos, generar el token JWT para devolverlo al usuario.

3. A partir de ahí la aplicación cliente, con ese token, haría peticiones solicitando recursos, siempre con ese token JWT dentro de un encabezado, que sería Authorization: Bearer XXXXXXX, siendo Bearer el tipo de prefijo seguido de todo el contenido del token.

4. En el servidor se comprobaría el token mediante la firma, para verificar que el token es seguro, y, por tanto, podemos confiar en el usuario.

5. Dentro del cuerpo del token, además, tenemos los datos de quién es el usuario que ha realizado esa petición, porque podemos contener en el payload todos los datos de usuario que queramos.

6. Tras verificar que el token es correcto y saber quién es el que ha hecho la petición, podemos aplicar entonces el mecanismo de control de acceso, saber si puede acceder o no, y si es así, responder con el recurso protegido, de manera que lo podría recibir de una forma correcta.

De esta forma podríamos implementar el proceso de autenticación, y hacerlo, además, con estos JSON Web Token.
