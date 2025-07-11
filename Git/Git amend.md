### ¿Qué hace `git commit --amend`?

En lugar de crear un nuevo commit, `git commit --amend` toma el último commit, aplica tus nuevos cambios (los que tienes en el **Staging Area**) o el nuevo mensaje, y crea un **nuevo commit completamente diferente** que reemplaza al anterior. Es decir, no estás "editando" el commit original, sino creando uno nuevo con las correcciones, que luego sobrescribe al anterior.

Esto es importante porque si ya subiste ese commit a una rama remota compartida, cambiarlo con `--amend` y luego intentar hacer `git push` requerirá un `git push --force` (o `git push --force-with-lease`), lo cual puede ser problemático si otros ya han trabajado sobre ese commit. Por eso, `git commit --amend` es **ideal para commits que aún no has subido (pushed)**.

-----

### Cómo usar `git commit --amend`

Aquí tienes los escenarios más comunes:

#### Escenario 1: Cambiar el Mensaje del Último Commit

Si te equivocaste en el mensaje del commit anterior y no necesitas cambiar ningún archivo:

1.  Asegúrate de que no tienes cambios pendientes en tu **Working Directory** o **Staging Area**.

    ```bash
    git status
    # Debe decir algo como "nothing to commit, working tree clean"
    ```

2.  Ejecuta el comando para enmendar el commit. Esto abrirá tu editor de texto predeterminado con el mensaje del último commit.

    ```bash
    git commit --amend
    ```

3.  Edita el mensaje del commit, guárdalo y cierra el editor. Git creará un nuevo commit con el nuevo mensaje, reemplazando el anterior.

#### Escenario 2: Añadir Cambios o Archivos al Último Commit

Si te olvidaste de añadir un archivo o de incluir algunos cambios en el commit anterior:

1.  Realiza los cambios necesarios en tus archivos o crea los archivos nuevos.

2.  Añade esos cambios al **Staging Area** (como lo harías antes de un commit normal).

    ```bash
    git add nombre_del_archivo_modificado.txt
    # O para añadir todo lo modificado/nuevo:
    git add .
    ```

3.  Ejecuta el comando para enmendar el commit. Esto abrirá tu editor de texto.

    ```bash
    git commit --amend
    ```

    El mensaje de commit que verás será el del commit anterior. Puedes dejarlo como está o modificarlo.

4.  Guarda el mensaje y cierra el editor. Git tomará los cambios que acabas de añadir al Staging Area, los combinará con los del commit anterior y creará un *nuevo* commit que reemplazará al original. El nuevo commit tendrá el nuevo mensaje (si lo cambiaste) y contendrá todos los cambios (los originales más los que acabas de añadir).

#### Escenario 3: Cambiar el Mensaje Y Añadir Cambios

Combina los pasos de los escenarios 1 y 2: haz tus cambios, añádelos al Staging Area con `git add .`, y luego ejecuta `git commit --amend` para editar el mensaje y consolidar los cambios.

-----

### Consideraciones Importantes

  * **No uses `--amend` en commits ya subidos y compartidos:** Si el commit que estás enmendando ya fue subido a una rama remota y otros desarrolladores ya lo han bajado o han trabajado sobre él, reescribir el historial con `git commit --amend` (que crea un commit con un nuevo hash) causará problemas. Los demás tendrán el "viejo" commit, y tú intentarás subir un "nuevo" commit diferente. En ese caso, para subir tu cambio después de un `amend`, necesitarías hacer un `git push --force` (o `git push --force-with-lease`), lo cual puede desincronizar los repositorios de tus compañeros.
      * **Si ya subiste el commit y otros lo han visto:** Es preferible hacer un nuevo commit que deshaga los cambios (`git revert`) o simplemente hacer un nuevo commit con las correcciones.
  * **Puedes usarlo para limpiar tu historial antes de un `push`:** `git commit --amend` es muy útil cuando estás trabajando localmente y haces un commit, pero luego te das cuenta de que olvidaste algo o el mensaje no era claro. Puedes "limpiar" tu último commit antes de subirlo al remoto.