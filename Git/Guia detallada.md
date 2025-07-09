## Guía Detallada de Git: Tu Referencia Rápida

Git es un sistema de control de versiones distribuido que te permite rastrear cambios en tus archivos, colaborar con otros y revertir a versiones anteriores de tu código. Es una herramienta esencial en el desarrollo de software moderno.

-----

### 1\. Conceptos Fundamentales de Git

Antes de sumergirnos en los comandos, entendamos algunos conceptos clave:

  * **Repositorio (Repo)**: Es la base de tu proyecto. Contiene todos los archivos de tu proyecto y el historial completo de todos los cambios.
      * **Local**: El repositorio en tu propia máquina.
      * **Remoto**: El repositorio alojado en un servidor (como GitLab, GitHub, Bitbucket), donde puedes colaborar con otros.
  * **Commit**: Un "punto de guardado" o instantánea de tu código en un momento específico. Cada commit tiene un mensaje que describe los cambios que incluye.
  * **Rama (Branch)**: Una línea independiente de desarrollo. Permite a los desarrolladores trabajar en nuevas características o correcciones de errores sin afectar la línea principal de código. La rama principal suele llamarse `main` o `master`.
  * **Merge (Fusionar)**: El proceso de combinar los cambios de una rama en otra.
  * **HEAD**: Un puntero que siempre apunta al último commit en la rama actual.
  * **Working Directory (Directorio de Trabajo)**: Los archivos de tu proyecto tal como están actualmente en tu sistema de archivos.
  * **Staging Area (Área de Preparación/Índice)**: Un área intermedia donde preparas los cambios que quieres incluir en tu próximo commit. Los archivos deben pasar por aquí antes de ser commiteados.

-----

### 2\. Configuración Inicial de Git

Antes de usar Git por primera vez, necesitas configurar tu identidad. Esto es importante porque cada commit que hagas incluirá esta información.

```bash
# Configura tu nombre de usuario (aparecerá en tus commits)
git config --global user.name "Tu Nombre"

# Configura tu dirección de correo electrónico (aparecerá en tus commits)
git config --global user.email "tu_email@example.com"

# Opcional: Configura tu editor de texto predeterminado para Git
git config --global core.editor "code --wait" # Para VS Code
git config --global core.editor "nano" # Para Nano
git config --global core.editor "vim" # Para Vim
```

-----

### 3\. Creando y Clonando Repositorios

#### **Inicializar un nuevo repositorio local:**

Si tienes una carpeta de proyecto existente que quieres convertir en un repositorio Git.

```bash
# Navega a la carpeta de tu proyecto
cd /ruta/a/tu/proyecto

# Inicializa un nuevo repositorio Git
git init
```

#### **Clonar un repositorio existente (remoto):**

Si quieres trabajar en un proyecto que ya existe en GitLab, GitHub, etc.

```bash
# Clona el repositorio a tu máquina local
git clone <URL_del_repositorio_remoto>
# Ejemplo: git clone https://gitlab.com/usuario/mi-proyecto.git
```

-----

### 4\. Flujo de Trabajo Básico: Add, Commit, Push

Este es el ciclo de vida fundamental para guardar y compartir tus cambios.

#### **Verificar el estado del repositorio:**

Muestra los archivos modificados, los que están en el área de preparación y los que aún no han sido rastreados.

```bash
git status
```

#### **Añadir archivos al Staging Area (preparar para commit):**

Esto marca los cambios que quieres incluir en tu próximo commit.

```bash
# Añadir un archivo específico
git add nombre_del_archivo.txt

# Añadir múltiples archivos específicos
git add archivo1.js archivo2.css

# Añadir todos los archivos modificados y nuevos en el directorio actual y subdirectorios
git add .

# Añadir todos los cambios de archivos ya rastreados (sin incluir nuevos archivos no rastreados)
git add -u
```

#### **Realizar un commit (guardar los cambios):**

Crea una instantánea permanente de los archivos que están en el Staging Area. Siempre incluye un mensaje descriptivo.

```bash
# Commit con un mensaje corto en una línea
git commit -m "Mensaje descriptivo del commit"

# Commit con un mensaje más detallado (abre tu editor de texto configurado)
git commit
```

#### **Enviar cambios al repositorio remoto (Push):**

Sube tus commits locales a la rama correspondiente en el repositorio remoto.

```bash
# Envía los commits de tu rama actual al remoto
git push

# Primera vez que subes una rama (establece el "upstream")
git push --set-upstream origin <nombre_de_tu_rama>
# o de forma abreviada:
git push -u origin <nombre_de_tu_rama>

# Ejemplo: git push -u origin main
```

-----

### 5\. Sincronización con el Repositorio Remoto

#### **Obtener los últimos cambios del remoto (Fetch):**

Descarga los cambios de una rama remota, pero **no los integra** en tu rama local de trabajo. Simplemente actualiza la referencia de las ramas remotas.

```bash
git fetch
```

#### **Descargar y fusionar los últimos cambios (Pull):**

Combina `git fetch` y `git merge`. Descarga los cambios de una rama remota y automáticamente los integra en tu rama local actual.

```bash
# Trae y fusiona los cambios de la rama remota que estás siguiendo
git pull

# Trae y fusiona los cambios de una rama remota específica
git pull origin <nombre_de_la_rama_remota>
# Ejemplo: git pull origin main
```

-----

### 6\. Ramas (Branches)

Las ramas son fundamentales para el flujo de trabajo en Git.

#### **Listar ramas:**

```bash
# Listar todas las ramas locales
git branch

# Listar todas las ramas (locales y remotas)
git branch -a
```

#### **Crear una nueva rama:**

```bash
# Crea una nueva rama (no te cambia a ella)
git branch <nombre_de_la_nueva_rama>
```

#### **Cambiar a una rama existente:**

```bash
git checkout <nombre_de_la_rama>
```

#### **Crear y cambiar a una nueva rama (atajo):**

```bash
git checkout -b <nombre_de_la_nueva_rama>
```

#### **Eliminar una rama:**

  * **Localmente:**
    ```bash
    # Eliminar una rama local (solo si está fusionada con la rama actual)
    git branch -d <nombre_de_la_rama>

    # Eliminar una rama local (forzado, incluso si tiene commits no fusionados)
    git branch -D <nombre_de_la_rama>
    ```
  * **Remotamente:**
    ```bash
    git push origin --delete <nombre_de_la_rama_remota>
    # Ejemplo: git push origin --delete feature/nueva-funcionalidad
    ```

#### **Fusionar (Merge) ramas:**

Combina los cambios de una rama en la rama actual.

```bash
# Primero, asegúrate de estar en la rama a la que quieres traer los cambios
git checkout <rama_destino> # Ejemplo: git checkout main

# Luego, fusiona la rama de origen en la rama actual
git merge <rama_origen> # Ejemplo: git merge feature/nueva-funcionalidad
```

-----

### 7\. Ver el Historial

#### **Ver el historial de commits:**

```bash
# Muestra el historial completo de commits (fecha, autor, mensaje)
git log

# Muestra un historial más conciso (un commit por línea)
git log --oneline

# Muestra el historial con un gráfico de ramas y merges
git log --graph --oneline --all

# Ver los cambios introducidos por un commit específico
git show <hash_del_commit>
```

-----

### 8\. Deshacer Cambios

Deshacer acciones en Git puede ser un poco complejo, ¡úsalo con cuidado\!

#### **Deshacer cambios en el directorio de trabajo (antes de `git add`):**

```bash
# Descarta los cambios en un archivo específico
git checkout -- <nombre_del_archivo>

# Descarta todos los cambios no añadidos en el directorio de trabajo
git restore . # git restore es más moderno que git checkout -- .
```

#### **Deshacer cambios del Staging Area (después de `git add`, antes de `git commit`):**

```bash
# Quita un archivo del Staging Area, pero mantiene los cambios en tu directorio de trabajo
git reset HEAD <nombre_del_archivo>
# o el comando más moderno:
git restore --staged <nombre_del_archivo>

# Quita todos los archivos del Staging Area
git reset HEAD .
# o el comando más moderno:
git restore --staged .
```

#### **Deshacer el último commit (con cuidado):**

  * **`git reset` (Elimina commits del historial, cuidado al usarlo en ramas compartidas):**

    ```bash
    # Mueve HEAD y la rama actual al commit anterior, pero mantiene los cambios en tu directorio de trabajo (y los deja en staged)
    git reset --soft HEAD~1

    # Mueve HEAD y la rama actual al commit anterior, los cambios quedan en tu directorio de trabajo pero NO en staged
    git reset --mixed HEAD~1 # Este es el default si no pones nada

    # Mueve HEAD y la rama actual al commit anterior, y DESCARTA los cambios (elimina los archivos modificados)
    git reset --hard HEAD~1
    ```

    `HEAD~1` se refiere al commit anterior al actual. `HEAD~2` dos commits antes, y así sucesivamente. También puedes usar el **hash del commit** al que quieres regresar.

  * **`git revert` (Crea un nuevo commit que deshace los cambios del commit anterior, más seguro para historial compartido):**

    ```bash
    # Crea un nuevo commit que revierte los cambios de un commit específico
    git revert <hash_del_commit_a_revertir>

    # Revierte el último commit
    git revert HEAD
    ```

    `git revert` es preferible cuando los commits ya han sido publicados en un repositorio remoto, ya que no reescribe el historial.

-----

### 9\. Manejo de Conflictos de Fusión (Merge Conflicts)

Los conflictos ocurren cuando Git no sabe cómo combinar automáticamente los cambios de dos ramas (por ejemplo, si ambos modificaron la misma línea en el mismo archivo).

#### **Cuando ocurre un conflicto:**

1.  Git pausará la fusión y te informará sobre los archivos con conflictos.

2.  Abre esos archivos en tu editor. Verás marcadores especiales:

    ```
    <<<<<<< HEAD
    Tu código en la rama actual
    =======
    El código de la rama que estás fusionando
    >>>>>>> nombre-de-la-otra-rama
    ```

3.  Edita el archivo para resolver el conflicto, eliminando los marcadores `<<<<<<<`, `=======`, `>>>>>>>` y dejando el código final deseado.

4.  Guarda el archivo.

5.  Añade el archivo resuelto al Staging Area: `git add <nombre_del_archivo_con_conflicto>`

6.  Completa el merge commit: `git commit -m "Resolución de conflicto"` (Git a menudo generará un mensaje de commit predeterminado para esto).

-----

### 10\. Etiquetar Versiones (Tags)

Los tags se usan para marcar puntos específicos en el historial como importantes (por ejemplo, versiones de lanzamiento).

```bash
# Crear un tag ligero (solo un puntero a un commit)
git tag v1.0.0

# Crear un tag anotado (incluye nombre del autor, fecha, mensaje, etc.)
git tag -a v1.0.0 -m "Versión de lanzamiento 1.0.0"

# Listar tags
git tag

# Subir tags al remoto
git push origin --tags

# Eliminar un tag local
git tag -d v1.0.0

# Eliminar un tag remoto
git push origin --delete v1.0.0
```

-----

### 11\. Otros Comandos Útiles

  * **`git status -s` o `git status --short`**: Versión corta del estado.
  * **`git diff`**: Muestra las diferencias entre el directorio de trabajo y el Staging Area, o entre el Staging Area y el último commit.
      * `git diff`: Cambios no staged.
      * `git diff --staged`: Cambios staged.
      * `git diff HEAD`: Todos los cambios desde el último commit.
  * **`git rm <archivo>`**: Elimina un archivo del directorio de trabajo y del índice (lo prepara para el próximo commit).
  * **`git mv <origen> <destino>`**: Renombra o mueve un archivo y lo prepara para el próximo commit.
  * **`git remote -v`**: Muestra las URLs de tus repositorios remotos.
  * **`git remote add <nombre> <URL>`**: Añade un nuevo remoto a tu repositorio (raramente usado después del `git clone` inicial).

-----