# Administración de Sistemas de Información (ASI)
## Solucionario Completo de Prácticas y Pruebas de Evaluación

---

# PARTE 1: Estudio Experimental Práctica 3 (Usuarios, ficheros y permisos)

## Primera parte: Warming up

### 1. Indique a qué grupo principal y a qué grupos secundarios pertenecen tanto el usuario "normal" (es decir, asiuser) como el usuario "root".

**Respuesta:**  
Para consultar la pertenencia a grupos se utiliza la orden `id`:
```bash
id asiuser
id root
```
- **Usuario `asiuser`:**
  - **Grupo principal:** `asiuser` (GID 1000).
  - **Grupos secundarios:** Grupos estándar del sistema en Debian como `cdrom`, `floppy`, `audio`, `dip`, `video`, `plugdev`, `netdev` y `sudo`.
- **Usuario `root`:**
  - **Grupo principal:** `root` (GID 0).
  - **Grupos secundarios:** Únicamente `root` (GID 0).

---

### 2. En el directorio de dispositivos existe un fichero llamado "ttyS0" que permite acceder al primer puerto serie del PC. Ese fichero en concreto, ¿Es un fichero de dispositivo? Justifíquelo.

**Respuesta:**  
Sí, es un fichero especial de dispositivo de tipo carácter. Al comprobarlo con:
```bash
ls -l /dev/ttyS0
```
Se observa que el primer carácter de la cadena de permisos es una `c` (en lugar de `-` para fichero ordinario o `d` para directorio). Además, en vez de mostrar un tamaño en bytes, muestra dos números separados por coma denominados *major number* (identificador del controlador en el kernel) y *minor number* (instancia del dispositivo).

---

### 3. Indique, de forma justificada, si el usuario "normal" tiene acceso al primer puerto serie del PC.

**Respuesta:**  
No tiene acceso. Los permisos por defecto en Debian para este nodo de dispositivo son:
```text
crw-rw---- 1 root dialout 4, 64 /dev/ttyS0
```
El propietario (`root`) y los miembros del grupo (`dialout`) tienen permisos de lectura y escritura (`rw-`), pero la terna correspondiente al resto del mundo (*others*) carece de todo permiso (`---`). Dado que el usuario estándar `asiuser` no pertenece al grupo `dialout`, el sistema deniega el acceso.

---

### 4. NOTA: El resto de las tareas de la primera parte se deben llevar a cabo como usuario normal.

**Respuesta:**  
Se realizan las operaciones desde la cuenta no privilegiada `asiuser`.

---

### 5. Como usuario "normal", intente crear (en su directorio de usuario) un enlace simbólico y un enlace duro a /bin/ls. ¿Es posible?

**Respuesta:**  
- **Enlace simbólico:** Sí es posible:
  ```bash
  ln -s /bin/ls ~/ls_sym
  ```
  Un enlace simbólico almacena la ruta textual del fichero apuntado y solo requiere permiso de escritura en el directorio destino del usuario (`~`).
- **Enlace duro:** Sí es posible mediante:
  ```bash
  ln /bin/ls ~/ls_hard
  ```
  Crear un enlace duro añade una referencia en el directorio al mismo inodo de `/bin/ls`. Es posible siempre que el directorio personal del usuario se encuentre montado dentro del mismo sistema de ficheros que `/bin`. Si `/home` residiera en una partición o sistema de ficheros separado, el comando fallaría porque los enlaces duros no pueden cruzar distintos sistemas de ficheros.

---

### 6. El comando "ps" permite a un usuario saber los procesos asociados a ese terminal y le indica, entre otras cosas, el terminal que está usando. Dicho terminal se corresponde con un fichero bajo el directorio /dev. Teclee ese comando e indique los permisos que tiene el fichero de dispositivo que se corresponde con el terminal que está usando.

**Respuesta:**  
Al ejecutar:
```bash
ps
```
En la columna `TTY` se identifica el terminal en uso (por ejemplo, `pts/0`). Para examinar sus permisos:
```bash
ls -l /dev/pts/0
```
La salida típica muestra:
```text
crw--w---- 1 asiuser tty 136, 0 /dev/pts/0
```
- **Propietario:** `asiuser` con permisos de lectura y escritura (`rw-`).
- **Grupo:** `tty` con permiso de escritura únicamente (`--w-`).
- **Otros:** Sin ningún permiso (`---`).

---

### 7. ¿Qué ocurre si tecleo echo "Hola" > /dev/XXX (XXX se corresponde con el terminal que descubrió en el apartado anterior). ¿Qué ocurre? A la vista de lo que ha pasado y teniendo en cuenta la respuesta que dio en el apartado anterior ¿cree que otros usuarios podrían escribir en su terminal?

**Respuesta:**  
- **Qué ocurre:** La palabra `Hola` se imprime por la pantalla de ese mismo terminal, dado que se redirige la salida estándar al dispositivo de caracteres asociado al terminal interactivo.
- **¿Pueden otros usuarios escribir?:** No directamente mediante una redirección, ya que los permisos para otros usuarios son `---`. Si otro usuario ordinario intenta redirigir texto hacia `/dev/pts/0`, recibirá un error de permiso denegado (`Permission denied`). Solo podrían escribirle mediante herramientas del sistema como `wall` o `write`, las cuales disponen de privilegios especiales (setgid `tty`).

---

## Segunda parte: Creando múltiples cuentas y grupos

### 8. Elabore un esquema de usuarios, grupos, directorios y permisos que le permita cumplir los siguientes requisitos:
- **Tanto los directivos como los administradores de sistemas deben pertenecer también al grupo de "otros".**
- **Se debe establecer una estructura de directorios de usuarios (/home) tal que asegure que los ficheros que estén ubicados en el $HOME de los usuarios del grupo "dire" no puedan ser accedidos por los usuarios que NO pertenecen a ese grupo, incluso si los directivos configuran mal los permisos de su $HOME.**

**Respuesta:**  
- **Grupos a crear:** `dire`, `utec`, `otros`.
- **Usuarios de prueba:** `usu_dire` (grupo principal `dire`), `usu_utec` (grupo principal `utec`), `usu_otros` (grupo principal `otros`).
- **Grupos secundarios:** Se añade a `usu_dire` y `usu_utec` al grupo `otros`.
- **Estructura de directorios y aislamiento:**  
  Para garantizar que nadie ajeno a `dire` acceda a sus ficheros aunque un directivo establezca `chmod 777` en su directorio personal, los directorios personales no colgarán directamente de `/home`, sino de un subdirectorio intermedio `/home/dire`:
  - Directorio contenedor: `/home/dire`
  - Propietario y grupo: `root:dire`
  - Permisos: `750` (`drwxr-x---`) o `770` (`drwxrwx---`).
  - Al no otorgar permiso de ejecución (`x` o paso) al resto de usuarios en `/home/dire`, ningún proceso ejecutado por un usuario que no pertenezca al grupo `dire` podrá atravesar la ruta hacia `/home/dire/usu_dire`.

---

### 9. Cree 3 cuentas (una de cada tipo de usuario). Asigne los permisos a los directorios de forma que se cumplan los requisitos señalados en el apartado anterior. Indique la salida del comando "ls -la /home".

**Respuesta:**  
Comandos ejecutados como `root`:
```bash
# Creación de grupos
addgroup dire
addgroup utec
addgroup otros

# Creación del directorio intermedio seguro para directivos
mkdir /home/dire
chown root:dire /home/dire
chmod 750 /home/dire

# Creación de cuentas
adduser --home /home/dire/usu1 --ingroup dire usu1
adduser --ingroup utec usu2
adduser --ingroup otros usu3

# Asignación de grupos secundarios
adduser usu1 otros
adduser usu2 otros
```

Salida representativa de `ls -la /home`:
```text
drwxr-xr-x  5 root  root  4096 /home
drwxr-x---  3 root  dire  4096 /home/dire
drwxr-xr-x  2 usu2  utec  4096 /home/usu2
drwxr-xr-x  2 usu3  otros 4096 /home/usu3
```
- `/home/dire` tiene permisos numéricos `750`, propietario `root`, grupo `dire`.

---

### 10. Escriba la salida de "id -a usu1" donde usu1 es el nombre del usuario que pertenece al grupo "dire".

**Respuesta:**  
```bash
id -a usu1
```
Salida obtenida:
```text
uid=1001(usu1) gid=1001(dire) grupos=1001(dire),1003(otros)
```

---

## Tercera parte: Compartiendo información

### 11. Requisitos para el directorio /home/repositorio:
- **Usuario "repo" (grupo principal "nogroup"), cuenta bloqueada con shell válida, $HOME /home/repositorio.**
- **En $HOME, un directorio para el grupo "dire" y otro para todos los usuarios, propiedad de "repo".**
- **En el directorio para todos: cualquier usuario puede crear ficheros o directorios, pero solo el propietario del fichero puede borrarlo.**
- **En el directorio de "dire": solo miembros de "dire" pueden ver contenido y crear. Los ajenos no tienen acceso. Miembros de "dire" pueden borrar cualquier fichero aunque no sea suyo.**

**Respuesta:**  
Diseño de la estructura:
1. Directorio `/home/repositorio/todos`:
   - Propietario: `repo:nogroup`
   - Permisos: `1777` (`drwxrwxrwt`). El *sticky bit* (`1` o `+t`) restringe el borrado exclusivamente al dueño del archivo, al dueño del directorio o al superusuario.
2. Directorio `/home/repositorio/dire`:
   - Propietario: `repo:dire`
   - Permisos: `2770` (`drwxrws---`). El permiso de grupo `rwx` permite a los miembros listar, crear y borrar cualquier entrada (sin sticky bit). El bit *setgid* (`2` o `g+s`) hace que los nuevos ficheros creados hereden automáticamente el grupo `dire`. Otros usuarios tienen `0` (`---`).

---

### 12. Lleve a cabo la tarea señalada en el apartado anterior. Indique comandos, justificación de permisos y salida de "ls -lad /home/repositorio".

**Respuesta:**  
Comandos para crear y configurar el usuario y directorios:
```bash
# Creación del usuario repo con cuenta bloqueada y shell válida
adduser --home /home/repositorio --ingroup nogroup --shell /bin/bash --disabled-password --gecos "" repo

# Creación de subdirectorios
mkdir /home/repositorio/todos
mkdir /home/repositorio/dire

# Configuración de propiedad
chown repo:nogroup /home/repositorio/todos
chown repo:dire /home/repositorio/dire

# Asignación de permisos
chmod 1777 /home/repositorio/todos
chmod 2770 /home/repositorio/dire

# Asegurar permiso de paso en el repositorio raíz
chmod 755 /home/repositorio
```

- **Justificación de permisos:**
  - `/home/repositorio/todos` (`1777`): Lectura, escritura y paso para todos; el bit pegajoso (*sticky bit*) impide que un usuario borre el trabajo de otro.
  - `/home/repositorio/dire` (`2770`): Lectura, escritura y ejecución reservadas para el propietario y el grupo `dire`; `setgid` fuerza la pertenencia al grupo `dire` de los nuevos elementos; la ausencia del sticky bit permite el borrado cruzado entre directivos.
- **Salida de `ls -lad /home/repositorio`:**
  ```text
  drwxr-xr-x 4 repo nogroup 4096 /home/repositorio
  ```
  Permisos numéricos `755`, propietario `repo`, grupo `nogroup`.

---

### 13. Realice las pruebas para comprobar la configuración del repositorio.

**Respuesta:**  
- **Prueba 1 (asiuser en directorio para todos y acceso a dire):**
  ```bash
  su - asiuser -c "touch /home/repositorio/todos/fichero_asiuser"
  ls -l /home/repositorio/todos/fichero_asiuser
  # Funciona correctamente

  su - asiuser -c "ls /home/repositorio/dire"
  # Devuelve: ls: cannot open directory '/home/repositorio/dire': Permission denied
  ```
- **Prueba 2 (usu1 de "dire" no puede borrar fichero de asiuser en 'todos'):**
  ```bash
  su - usu1 -c "rm /home/repositorio/todos/fichero_asiuser"
  # Devuelve: rm: cannot remove '/home/repositorio/todos/fichero_asiuser': Operation not permitted
  ```
  El sticky bit impide la eliminación.
- **Prueba 3 (usu1 de "dire" crea fichero en 'dire'):**
  ```bash
  su - usu1 -c "touch /home/repositorio/dire/acta_reunion.txt"
  ls -l /home/repositorio/dire/acta_reunion.txt
  # Comprobación de herencia de grupo por setgid:
  # -rw-r--r-- 1 usu1 dire 0 ... acta_reunion.txt
  ```

---

## Cuarta parte: Permisos especiales

### 14. Inicie sesión con el usuario "root" ¿está establecido el bit setuid en el fichero /usr/bin/su"? ¿En qué ha tenido que fijarse para saber la respuesta?

**Respuesta:**  
Sí, está establecido.  
Se comprueba mediante la orden:
```bash
ls -l /usr/bin/su
```
Salida obtenida:
```text
-rwsr-xr-x 1 root root ... /usr/bin/su
```
Nos fijamos en la terna de permisos del propietario (`rws`): la presencia de la letra `s` en la posición donde habitualmente se muestra el permiso de ejecución (`x`) indica que el bit *setuid* se encuentra activo.

---

### 15. Modifique los permisos de /usr/bin/su para que el bit setuid esté desactivado y que el resto de los permisos no cambie. Indique lo que ha tecleado.

**Respuesta:**  
```bash
chmod u-s /usr/bin/su
```
(O bien de forma absoluta mediante `chmod 0755 /usr/bin/su`).

---

### 16. Inicie sesión en otro terminal como usuario "normal" e intente usar el programa "su" para abrir una Shell de superusuario ¿funciona? Explique por qué no funciona.

**Respuesta:**  
- **Resultado:** No funciona; el comando devuelve un mensaje de fallo de autenticación o error interno.
- **Explicación:** El programa `su` necesita autenticar al usuario frente a `/etc/shadow` y, posteriormente, cambiar su identidad (UID) a 0 invocando llamadas al sistema privilegiadas (`setuid(0)`). Al deshabilitar el bit setuid, `su` se ejecuta únicamente con los privilegios del usuario ordinario que lo lanza, el cual no tiene acceso a `/etc/shadow` ni capacidad para asumir la identidad de `root`.

---

### 17. Vuelva a establecer los permisos de "su" de forma que tenga de nuevo el bit setuid activo. Indique lo que ha tecleado.

**Respuesta:**  
```bash
chmod u+s /usr/bin/su
```
(O bien en formato numérico `chmod 4755 /usr/bin/su`).

---

### 18. Compruebe que "su" funciona ahora correctamente.

**Respuesta:**  
Al ejecutar `su -` desde una cuenta ordinaria e introducir la contraseña de `root`, se obtiene con éxito un prompt de superusuario (`#`).

---

### 19. En Debian existe la orden "/usr/bin/systemctl poweroff" que nos permite hacer una parada ordenada del sistema y apagarlo una vez que se ha producido esto. ¿Puede un usuario no privilegiado apagar el sistema de esta forma? Compruébelo y anote el mensaje de error.

**Respuesta:**  
Si el usuario ordinario está conectado por consola remota (por ejemplo, a través de SSH) o no dispone de sesión local activa controlada por logind, la ejecución de:
```bash
/usr/bin/systemctl poweroff
```
Falla mostrando el siguiente mensaje de error:
```text
Failed to power off system via logind: Interactive authentication required.
Failed to start poweroff.target: Access denied
```

---

### 20. ¿De qué forma se podría lograr que los usuarios pertenecientes al grupo "utec" puedan apagar el sistema sin necesidad de conocer la contraseña del superusuario (y sin usar "sudo" o "su"). Realice los cambios pertinentes, anote lo que ha hecho y compruebe el funcionamiento.

**Respuesta:**  
Se puede crear una copia específica del binario de apagado o asignar permisos especiales con *setuid* y restricción de grupo:
```bash
# Se crea una copia o enlace al comando poweroff
cp /lib/systemd/systemd-shutdown /usr/local/bin/apagar_sistema
# O bien sobre una utilidad binaria de apagado como /sbin/poweroff:
chown root:utec /usr/local/bin/apagar_sistema
chmod 4750 /usr/local/bin/apagar_sistema
```
- **Funcionamiento:**
  - El grupo propietario es `utec`.
  - Los permisos `4750` (`-rwsr-x---`) permiten la ejecución al propietario (`root`) y a los miembros del grupo `utec`.
  - El resto de usuarios (`---`) no puede leerlo ni ejecutarlo.
  - Al poseer el bit setuid (`4`), cuando un miembro de `utec` lo ejecuta, el proceso corre con los privilegios efectivos de `root`, efectuando el apagado sin requerir contraseñas.

---

### 21. Suponga que ahora desea que cualquier usuario pueda apagar el sistema sin necesidad de conocer la contraseña del superusuario y sin necesidad de pertenecer al grupo "utec". La idea es que se le pueda suministrar a un usuario un nombre de usuario (ej: "apagar") y contraseña que le permita apagar el sistema, pero que ese usuario no pueda hacer nada más. ¿Cómo se podría hacer esto?

**Respuesta:**  
Se crea un usuario específico cuya shell de inicio de sesión no sea una consola interactiva (`/bin/bash`), sino directamente el binario de apagado o un script con permisos setuid que ejecute la orden de parada:
```bash
# Creación del usuario con shell asignada al comando de apagado
adduser --shell /sbin/poweroff apagar
```
(O vinculándolo a un script en `/usr/local/bin/hacer_apagado` si `/sbin/poweroff` requiere invocación como root).  
Al iniciar sesión el usuario `apagar`, el sistema arranca inmediatamente dicho binario; una vez completada la parada, el sistema se apaga y el usuario en ningún momento obtiene una shell interactiva.

---

### 22. Compruebe que lo que ha hecho en el apartado anterior funciona y anote los pasos que ha tenido que dar.

**Respuesta:**  
1. Se establece contraseña para la cuenta: `passwd apagar`.
2. Se inicia sesión local o mediante `su - apagar` o vía consola tty con dicho usuario.
3. El sistema lanza la orden asignada como shell y procede de inmediato al apagado ordenado del sistema sin conceder acceso a ninguna otra orden.

---

## Quinta parte: Listas de control de acceso (ACLs)

### 23. Compruebe si existen los ejecutables "getfacl" y "setfacl". Si no existen, instale el soporte ejecutando "apt-get install acl" como superusuario.

**Respuesta:**  
```bash
which getfacl setfacl || apt-get install acl
```

---

### 24. Lea la página de manual de "setfacl" a la hora de resolver el apartado 23.

**Respuesta:**  
Se consulta la sintaxis básica mediante `man setfacl`. La opción `-m` modifica la ACL (`setfacl -m u:usuario:permisos fichero`).

---

### 25. ¿Qué tendría que hacer si quisiera que el usuario "asiuser" pudiera tener acceso al directorio de los directivos que se creó en la parte segunda? Anote lo que ha hecho.

**Respuesta:**  
Para conceder acceso exclusivo a `asiuser` sin comprometer el aislamiento frente al resto de usuarios, se le otorga una regla de lista de control de acceso extendida (permiso de paso y lectura):
```bash
setfacl -m u:asiuser:rx /home/dire
```

---

### 26. Compruebe, mediante getfacl, que está establecido el permiso extendido.

**Respuesta:**  
```bash
getfacl /home/dire
```
Salida obtenida:
```text
# file: home/dire
# owner: root
# group: dire
user::rwx
user:asiuser:r-x
group::r-x
mask::r-x
other::---
```
El listado tradicional con `ls -ld /home/dire` muestra ahora un signo `+` al final de la cadena de permisos (`drwxr-x---+`).

---

### 27. Compruebe empíricamente que el usuario usado en el apartado 25 tiene, efectivamente, acceso al directorio de los directivos.

**Respuesta:**  
Iniciando sesión como `asiuser`:
```bash
su - asiuser
ls /home/dire
```
La orden lista el contenido del directorio sin errores de permiso.

---

## Sexta parte: Shell scripts y ficheros de inicio de sesión

### 28. Cree un script (/usr/local/bin/change_umask) que establezca los permisos por defecto de los ficheros de forma distinta dependiendo del grupo principal al que pertenece el usuario:
- **Para usuarios del grupo "dire": umask tal que no permita ningún acceso a "otros" y permita lectura/ejecución al grupo.**
- **Para el resto: no hacer cambios.**
- **El script debe mostrar un mensaje indicando que se han cambiado los permisos y a qué valor.**

**Respuesta:**  
Contenido del script `/usr/local/bin/change_umask`:
```bash
#!/bin/bash
# Obtener el grupo principal del usuario actual
GRUPO_PRINCIPAL=$(id -gn)

if [ "$GRUPO_PRINCIPAL" = "dire" ]; then
    # Máscara 027:
    # Ficheros: 666 - 027 = 640 (rw- r-- ---)
    # Directorios: 777 - 027 = 750 (rwx r-x ---)
    umask 027
    echo "Máscara de permisos modificada para el grupo dire: umask 027"
fi
```
Se otorgan permisos de ejecución:
```bash
chmod 755 /usr/local/bin/change_umask
```

---

### 29. Ejecute el script (/usr/local/bin/change_umask) para comprobar si funciona. ¿Se ha cambiado la máscara de permisos? ¿Por qué cree que no se ha cambiado?

**Respuesta:**  
- **Resultado:** Al ejecutar `/usr/local/bin/change_umask` y luego verificar con `umask`, el valor de la máscara no ha cambiado en la shell interactiva.
- **Por qué:** Al invocar un script mediante su ruta ejecutable, el sistema crea un nuevo proceso hijo (una subshell). El comando `umask` modifica únicamente el entorno del proceso hijo. Cuando el script finaliza, el proceso hijo muere y la shell padre conserva su máscara original intacta.

---

### 30. Si se deseara que se ejecutara ese script para que nos aseguráramos de que todos los usuarios tienen correctamente establecida la máscara de permisos ¿qué tendríamos que modificar?

**Respuesta:**  
Se debe incluir la invocación en los ficheros globales de inicio de sesión del sistema, concretamente en `/etc/profile` (para *login shells*) o en `/etc/bash.bashrc` (para shells interactivas).

---

### 31. Incorpore en los ficheros de configuración del apartado anterior una llamada al script. Inicie sesión como un usuario del grupo "dire" y comprobará que no funciona, a pesar de que se muestra el mensaje. ¿Por qué sucede?

**Respuesta:**  
Sucede exactamente por el mismo motivo explicado en la pregunta 29: si en `/etc/profile` se invoca el ejecutable como `/usr/local/bin/change_umask`, se lanza en una subshell independiente; el mensaje se imprime en pantalla, pero la máscara de la shell principal no se ve alterada.

---

### 32. Para evitar lo que sucede en el apartado anterior, debe invocarse el script de esta forma: ". /usr/local/bin/change_umask". Compruebe que ahora funciona correctamente. Describa lo sucedido.

**Respuesta:**  
- **Descripción:** Al anteponer el comando punto (`.`) o `source`, la shell actual no crea un subproceso, sino que lee y ejecuta las órdenes del archivo directamente en el contexto y memoria de la shell en curso. De este modo, la llamada interna a `umask 027` altera con éxito la máscara del entorno de trabajo del usuario.

---
---

# PARTE 2: Ejemplo 1ª Prueba de Evaluación

### 1. ¿Qué diferencias existen entre una "login shell" y una "non-login shell"?

**Respuesta:**  
- **Login shell:** Es la primera shell que se obtiene al autenticarse en el sistema mediante usuario y contraseña (por ejemplo, en una consola virtual de texto `tty1` o mediante una conexión `ssh`). Al arrancar, ejecuta de forma secuencial `/etc/profile` y el primer fichero que encuentre entre `~/.bash_profile`, `~/.bash_login` o `~/.profile`. Al cerrarse, ejecuta `~/.bash_logout`.
- **Non-login shell:** Es una shell secundaria abierta una vez que la sesión ya ha sido iniciada (por ejemplo, al abrir una ventana de terminal en un entorno gráfico o al teclear `bash`). No ejecuta los ficheros de perfil, sino que lee `/etc/bash.bashrc` y `~/.bashrc`.

---

### 2. ¿Qué diferencia existe entre teclear, en una shell, «./ls» y «ls»? ¿Guarda esto alguna relación con una variable de entorno del sistema?

**Respuesta:**  
- `./ls` indica una ruta relativa explícita: ordena al intérprete ejecutar el binario denominado `ls` ubicado exactamente en el directorio de trabajo actual (`.`).
- `ls` sin prefijo de ruta hace que la shell busque dicho ejecutable de manera secuencial a través de la lista de directorios definida en la variable de entorno `$PATH`.

---

### 3. Indique cómo crear (en UNIX/Linux) un directorio con nombre (usando la opción que menos caracteres necesite):
a. «Mi dir»  
b. «Mi  dir»  
c. «Mi*dir»  
d. «Mi-dir»  

**Respuesta:**  
- a. `mkdir Mi\ dir` (12 caracteres).
- b. `mkdir "Mi  dir"` (14 caracteres; usar comillas ahorra caracteres frente a escapar múltiples espacios).
- c. `mkdir Mi\*dir` (12 caracteres; se escapa el comodín `*` para evitar expansión).
- d. `mkdir Mi-dir` (11 caracteres; el guion no es carácter especial para la shell y no requiere escape).

---

### 4. Suponga el siguiente script llamado "prueba", con permisos de ejecución y ejecutado desde su propio directorio mediante «./prueba ASI»:
```bash
#!/bin/bash
echo "Param1: $0"
echo "Param2: $1"
echo "Param3: $#"
echo "Param4: $?"
```
¿Qué salida se mostrará por pantalla?

**Respuesta:**  
```text
Param1: ./prueba
Param2: ASI
Param3: 1
Param4: 0
```
- `$0` es el nombre con el que se ha invocado el script (`./prueba`).
- `$1` es el primer argumento recibido (`ASI`).
- `$#` es la cantidad total de argumentos posicionales (`1`).
- `$?` muestra el código de salida del último comando ejecutado antes de esa línea (el `echo` anterior, que finalizó con éxito: `0`).

---

### 5. Suponga que se desea identificar, usando una expresión regular, contenedores de carga mediante una cadena que comience por tres letras en mayúsculas o minúsculas, seguido del carácter menos y una secuencia numérica de uno o más dígitos. ¿Qué expresión regular usaría?

**Respuesta:**  
Expresión regular extendida:
```text
^[a-zA-Z]{3}-[0-9]+$
```
- `^` y `$` fijan el inicio y final de la línea.
- `[a-zA-Z]{3}` exige exactamente tres caracteres alfabéticos.
- `-` representa el carácter guion literal.
- `[0-9]+` exige uno o más dígitos numéricos.

---

### 6. Diferencia entre "sudo" y "su". ¿Qué contraseña se usa en ambos casos?

**Respuesta:**  
- **`su` (substitute user):** Inicia una nueva shell cambiando la identidad al usuario indicado (por defecto `root`). Requiere introducir la **contraseña del usuario destino** (`root`).
- **`sudo` (superuser do):** Ejecuta un comando puntual con privilegios elevados según lo configurado en `/etc/sudoers`. Requiere introducir la **contraseña del propio usuario que ejecuta el comando**.

---

### 7. Suponga que desea bloquear la cuenta de un usuario editando el/los fichero/s del sistema correspondientes. ¿Cómo lo haría? (Se necesita que cuando se desbloquee la cuenta, el usuario pueda seguir usando su contraseña).

**Respuesta:**  
Se edita el archivo `/etc/shadow` de forma segura mediante el comando:
```bash
vipw -s
```
Se localiza la línea correspondiente al usuario y se añade un signo de exclamación `!` justo al principio de su hash de contraseña (segundo campo).  
Para desbloquearla, basta con borrar dicho carácter `!`, restaurando el valor original intacto.

---

### 8. ¿Para qué sirve "umask"? Apoye su explicación con un ejemplo.

**Respuesta:**  
`umask` define la máscara de bits que el kernel sustrae a los permisos máximos predeterminados al crear nuevos elementos (ficheros parten de `666` y directorios de `777`).  
- **Ejemplo:** Si se establece `umask 027`:
  - Nuevos directorios: $777 - 027 = 750$ (`rwxr-x---`).
  - Nuevos ficheros: $666 - 027 = 640$ (`rw-r-----`).

---

### 9. ¿Qué usuarios, además de root, podrían apagar el sistema si los permisos del programa "poweroff" son estos?
`-rwsr-x--- 1 root asi 13848 Dec 2 17:29 poweroff`

**Respuesta:**  
Únicamente los usuarios pertenecientes al grupo **`asi`**.  
El permiso de grupo otorga lectura y ejecución (`r-x`). Como el ejecutable tiene activado el bit *setuid* asociado al propietario `root` (`rws`), al ser ejecutado por un miembro de `asi` correrá con permisos efectivos de `root`, pudiendo apagar el equipo. El resto de usuarios carece de acceso (`---`).

---

### 10. Suponga la siguiente salida de «ls -la /home/asiuser»:
```text
drwx--x--- 3 asiuser asi 4096 Nov 18 13:29 .
drwxrwxrwx 3 root    root 4096 Dec  2 18:00 ..
-rw-rw-r-- 1 asiuser asi   90 Nov 18 13:29 fich1.txt
```
Indique qué ocurre en los siguientes casos:
- **a) Un usuario del grupo "asi" teclea «cat /home/asiuser/fich1.txt»:**  
  **Funciona.** El directorio `/home/asiuser` tiene permiso de paso (`x`) para el grupo `asi`, y sobre el fichero `fich1.txt` el grupo posee permiso de lectura (`r`).
- **b) Un usuario del grupo "isi" teclea «cat /home/asiuser/fich1.txt»:**  
  **Falla (Permission denied).** El usuario entra en la categoría de "otros" respecto a `/home/asiuser`, cuyos permisos son `---` (no tiene permiso `x` para atravesar el directorio).
- **c) Un usuario del grupo "asi" teclea «rm /home/asiuser/fich1.txt»:**  
  **Falla (Permission denied).** Para borrar un fichero se necesita permiso de escritura (`w`) sobre el directorio que lo contiene. El grupo `asi` solo tiene `--x` en el directorio.

---

### 11. Suponga la siguiente salida de «ps -f»:
```text
UID        PID  PPID  C STIME TTY      TIME CMD
skywalker 2723 2437  0 18:54 tty1 00:00:00 bash
skywalker 2957 2723  0 19:19 tty1 00:00:00 ps -f
```
- **a) Relación paterno-filial:**  
  El proceso `bash` (PID 2723) es el padre del proceso `ps -f` (PID 2957), dado que el PPID de este último es 2723.
- **b) PID y PPID de una nueva shell lanzada mediante «bash»:**  
  El **PPID será 2723** (la shell actual genera una copia mediante `fork`). El **PID será un número nuevo mayor a 2957**, imposible de predecir exactamente porque depende de los procesos asignados por el kernel en ese instante.
- **c) PID y PPID de una nueva shell lanzada mediante «exec bash»:**  
  La llamada al sistema `exec` sustituye el código del proceso actual sin crear uno nuevo. Por tanto, el **PID seguirá siendo 2723** y su **PPID seguirá siendo 2437**.

---

### 12. Suponga un sistema con una sola CPU (1 core). Proceso 1: 75% en estado "D" y 25% en estado "R". Proceso 2: 100% en estado "R".
- **a) Diferencia entre estado "D" y "R":**  
  - `R` (Running/Runnable): El proceso está ejecutándose o preparado en la cola de CPU compitiendo por tiempo de procesamiento.
  - `D` (Uninterruptible Sleep): El proceso está bloqueado a la espera de un evento de hardware (normalmente E/S de disco) y no compite por tiempo de CPU ni puede atender señales.
- **b) Tiempo de CPU asignado a cada proceso en un intervalo de 60 segundos:**  
  - Durante el 75% del tiempo (45 segundos), el Proceso 1 está en `D`, dejando la CPU en exclusiva al Proceso 2: **45 s para Proceso 2**.
  - Durante el 25% restante (15 segundos), ambos procesos están simultáneamente en `R`. Al tener un único núcleo, se reparten equitativamente el tiempo de CPU al 50%: **7.5 s cada uno**.
  - **Total Proceso 1:** $7.5\text{ segundos}$.
  - **Total Proceso 2:** $45 + 7.5 = 52.5\text{ segundos}$.

---

### 13. Suponga que quiere ejecutar un proceso llamado "calcular" de forma que se descarte cualquier salida por pantalla que genere y que, además, ese proceso siga vivo al salir de la shell desde la que se lanza. ¿Qué habría que teclear?

**Respuesta:**  
```bash
nohup ./calcular > /dev/null 2>&1 &
```
(O bien `./calcular &> /dev/null &` seguido de `disown`).

---

### 14. Indique para qué sirven, qué evento las activa y si el proceso puede manejarlas: STOP, KILL, TERM e INT.

**Respuesta:**  
- **STOP (19):** Detiene la ejecución del proceso (lo suspende). Activada por llamadas de control o comandos de depuración. **No se puede capturar, manejar ni ignorar.**
- **KILL (9):** Finaliza el proceso de forma inmediata e incondicional. Activada manualmente con `kill -9`. **No se puede capturar, manejar ni ignorar.**
- **TERM (15):** Petición de terminación ordenada. Activada por defecto al usar `kill <PID>`. **Sí se puede manejar, capturar o ignorar.**
- **INT (2):** Interrupción del proceso. Activada desde el terminal mediante la combinación de teclas `Ctrl+C`. **Sí se puede manejar, capturar o ignorar.**

---

### 15. ¿Qué significa que un proceso esté en estado "zombie"?

**Respuesta:**  
Significa que el proceso hijo ya ha finalizado su ejecución y liberado su memoria, pero continúa figurando en la tabla de procesos del sistema debido a que su proceso padre aún no ha leído su código de salida mediante la llamada al sistema `wait()`.

---

### 16. Programar la alarma para despertar ejecutando el programa "despierta" de Lunes a Viernes a las 8.00h excepto el mes de agosto.

**Respuesta:**  
Se añade la siguiente entrada en la tabla del usuario con `crontab -e`:
```text
0 8 * 1-7,9-12 1-5 /ruta/completa/despierta
```
- `0`: Minuto 0.
- `8`: Hora 8.
- `*`: Cualquier día del mes.
- `1-7,9-12`: Meses de enero a julio y de septiembre a diciembre (excluye el 8, agosto).
- `1-5`: Días de la semana de lunes a viernes.

---

### 17. ¿Qué aporta "anacron" que no tenga cron?

**Respuesta:**  
`anacron` permite ejecutar trabajos con periodicidad diaria, semanal o mensual en equipos que no permanecen encendidos las 24 horas del día. Si el equipo estaba apagado en el momento en que debía ejecutarse una tarea, `anacron` detecta la omisión y la lanza automáticamente poco después de encender el sistema.

---

### 18. ¿Para qué sirven "/boot/vmlinuz-..." y "/boot/initrd.img-..."?

**Respuesta:**  
- **`vmlinuz-...`:** Es el archivo binario ejecutable y comprimido del núcleo (kernel) de Linux.
- **`initrd.img-...`:** Es una imagen de sistema de ficheros temporal en memoria RAM (*Initial RAM Disk*) que contiene los módulos y controladores esenciales para que el kernel pueda reconocer el disco raíz y montarlo durante el arranque.

---

### 19. Diez discos de 500GB en RAID-0, RAID-1 y RAID-5:
- **a) Tamaño del sistema de ficheros más grande posible:**
  - RAID-0: $10 \times 500\text{ GB} = 5000\text{ GB} = 5\text{ TB}$.
  - RAID-1: $500\text{ GB}$ (espejo total sobre los 10 discos).
  - RAID-5: $(10 - 1) \times 500\text{ GB} = 4500\text{ GB} = 4.5\text{ TB}$.
- **b) Discos que pueden fallar simultáneamente sin pérdida de datos:**
  - RAID-0: 0 discos (si falla uno, se pierde el volumen).
  - RAID-1: 9 discos.
  - RAID-5: 1 disco.
- **c) Rendimiento:**
  - **Mejor rendimiento general:** RAID-0 (paraleliza lecturas y escrituras en los 10 discos sin sobrecarga de paridad).
  - **Peor rendimiento en escritura:** RAID-5 (penalización por cálculo de paridad) y RAID-1 (debe escribir el bloque en los 10 discos).

---

### 20. Diferencia entre "mkfs -t ext4 /dev/sda" y "mkfs -t ext4 /dev/sda1". Recomendación y salida de «fdisk -l /dev/sda».

**Respuesta:**  
- **Recomendación:** Se debe usar **`mkfs -t ext4 /dev/sda1`**.  
- **Consecuencias:**  
  - `/dev/sda` representa el disco físico completo. Ejecutar `mkfs` sobre él sobrescribiría el principio del disco, destruyendo la tabla de particiones MBR/GPT y la partición de Windows.
  - `/dev/sda1` aplica el formato exclusivamente a la primera partición, dejando intacta la segunda partición donde reside Windows.
- **Salida de `fdisk -l /dev/sda`:**  
  - Si se usa `/dev/sda`: advertirá de que el disco carece de tabla de particiones válida.
  - Si se usa `/dev/sda1`: mostrará la tabla de particiones intacta con `/dev/sda1` y `/dev/sda2`.

---
---

# PARTE 3: Ejemplo 2ª Prueba de Evaluación

### 1. ¿Qué debe teclear en la Shell para que la salida estándar del programa "encuentra" se añada a "/tmp/salida1.txt" y la de errores sea descartada?

**Respuesta:**  
```bash
./encuentra >> /tmp/salida1.txt 2> /dev/null
```

---

### 2. Describa, mediante una expresión regular, una dirección MAC (xx:xx:xx:xx:xx:xx con dígitos hexadecimales).

**Respuesta:**  
Expresión regular extendida:
```text
^([0-9a-fA-F]{2}:){5}[0-9a-fA-F]{2}$
```

---

### 3. Cree un shell-script que busque el fichero que se le indique en el primer parámetro en el directorio indicado en el segundo parámetro (opcional; si se omite busca en todo el sistema). Si el número de parámetros es incorrecto, mostrar ayuda.

**Respuesta:**  
```bash
#!/bin/bash
if [ $# -eq 1 ]; then
    find / -name "$1"
elif [ $# -eq 2 ]; then
    find "$2" -name "$1"
else
    echo "Uso: $0 <nombre_fichero> [directorio_busqueda]"
    exit 1
fi
```

---

### 4. Indique qué hace este script:
```bash
#!/bin/bash
test -r /etc/passwd || exit
cat /etc/passwd | grep ja
```

**Respuesta:**  
Primero evalúa si el usuario actual tiene permisos de lectura sobre `/etc/passwd`. Si la comprobación falla, el operador lógico `||` provoca la salida inmediata del script mediante `exit`. Si el fichero es legible, vuelca su contenido y filtra mediante `grep` para mostrar únicamente las líneas que contengan la cadena de texto `ja`.

---

### 5. ¿De qué forma puedo hacer que para todos los ficheros y directorios creados por un usuario determinado, los permisos establecidos únicamente permitan acceder al propietario (sin usar chmod)?

**Respuesta:**  
Se configura la máscara de creación en su fichero de inicio de sesión (`~/.bashrc` o `~/.profile`):
```bash
umask 077
```
De este modo, los ficheros se crearán con permisos `600` (`rw-------`) y los directorios con `700` (`rwx------`).

---

### 6. Uso de "chmod":
- **a) Formas de especificación:**  
  - Modo numérico u octal: Asigna valores absolutos mediante la suma de bits de lectura (4), escritura (2) y ejecución (1) para usuario, grupo y otros.
  - Modo simbólico: Modifica selectivamente usando letras para las categorías (`u`, `g`, `o`, `a`), operadores (`+`, `-`, `=`) y tipos de permisos (`r`, `w`, `x`).
- **b) Permiso rw solo para el dueño en "fich":**  
  ```bash
  chmod 600 fich
  ```
- **c) Añadir permiso de paso a todo el mundo en el directorio "fich" sin alterar el resto:**  
  ```bash
  chmod a+X fich
  ```

---

### 7. ¿Qué diferencia existe entre «chmod -R a+rx /dire» y «chmod -R a+rX /dire»?

**Respuesta:**  
- `chmod -R a+rx /dire`: Otorga permiso de lectura y ejecución a todos los elementos del árbol, haciendo indebidamente ejecutables incluso a los ficheros de texto plano u ordinarios.
- `chmod -R a+rX /dire`: Otorga permiso de lectura a todo, pero la `X` mayúscula aplica el permiso de ejecución **únicamente a los directorios** (permitiendo el paso) y a los ficheros que ya tenían previamente permiso de ejecución para algún usuario.

---

### 8. Indique cómo hacer que su ordenador se apague esta noche a las 3 de la mañana.

**Respuesta:**  
Mediante la utilidad de ejecución diferida `at` (ejecutado como `root`):
```bash
at 03:00
systemctl poweroff
```
(Pulsar `Ctrl+D` para guardar el trabajo).

---

### 9. Salida de "uptime": "load average: 2, 2.1, 2.1" en un sistema mono-core. Se ejecutan como root: «nice +19 ./calcula1» y «nice -20 ./calcula2».
- **a) Carga del sistema pasados unos minutos:**  
  La carga media se estabilizará en torno a **2** (o ligeramente superior si se suman a los procesos ya existentes), ya que ambos procesos demandan el 100% de CPU y se encuentran en estado `R`.
- **b) % CPU asignado a cada uno:**  
  El planificador de CPU otorga prácticamente todo el tiempo al proceso con máxima prioridad (`nice -20`), asignando aproximadamente un **95% - 99% a `calcula2`** y solo un **1% - 5% a `calcula1`**.

---

### 10. ¿Qué es un "runlevel" y qué diferencias existen entre "telinit 0", "telinit 1" y "telinit 6"?

**Respuesta:**  
Un *runlevel* o nivel de ejecución define el estado operativo de un sistema basado en init SysV determinando qué servicios y subsistemas se inician o detienen.
- `telinit 0`: Apaga y detiene por completo el sistema (Halt / Poweroff).
- `telinit 1`: Modo monousuario (mantenimiento y rescate; sin entorno gráfico ni servicios de red).
- `telinit 6`: Reinicia completamente el sistema (Reboot).

---

### 11. ¿De qué forma puedo hacer que un conjunto de usuarios puedan apagar el sistema sin necesidad de usar "su" o "sudo", ni suministrándoles la contraseña de root?

**Respuesta:**  
1. Se crea un grupo específico (por ejemplo, `apagadores`) y se añaden a él los usuarios autorizados:
   ```bash
   addgroup apagadores
   adduser usuario1 apagadores
   ```
2. Se asigna la propiedad de una orden de apagado al grupo y se establece el bit *setuid*:
   ```bash
   chown root:apagadores /sbin/poweroff
   chmod 4750 /sbin/poweroff
   ```

---

### 12. NAS vs SAN: semejanzas y diferencias.

**Respuesta:**  
- **Semejanzas:** Ambas son arquitecturas de almacenamiento compartido en red.
- **Diferencias:**
  - **NAS (Network Attached Storage):** Proporciona almacenamiento a **nivel de fichero**. El dispositivo NAS gestiona su propio sistema de ficheros y se accede mediante protocolos de red como NFS o CIFS/SMB.
  - **SAN (Storage Area Network):** Proporciona almacenamiento a **nivel de bloque**. El servidor cliente percibe el almacenamiento como discos físicos locales mediante redes dedicadas de alta velocidad (iSCSI, Fibre Channel).

---

### 13. Indique cómo saber si un servidor está escuchando en el puerto 25 sin usar "ps" ni "top".

**Respuesta:**  
Se ejecuta:
```bash
netstat -tln
# o en sistemas modernos:
ss -tln
```
Se debe inspeccionar la columna **Local Address** buscando una entrada que finalice en `:25` con el estado en **LISTEN**.

---

### 14. Autenticación de clave pública en SSH.

**Respuesta:**  
El usuario genera un par de claves criptográficas (pública y privada). La clave pública se copia en el servidor dentro del fichero `~/.ssh/authorized_keys`. Al iniciar la conexión, el servidor genera un mensaje aleatorio (*challenge*) que cifra con dicha clave pública; el cliente demuestra su identidad descifrando el mensaje con su clave privada local.

---

### 15. ¿Qué ocurre si intenta montar un sistema de ficheros en un directorio que no está vacío?

**Respuesta:**  
El sistema de ficheros se monta correctamente, pero los archivos y subdirectorios preexistentes en ese directorio quedan temporalmente ocultos e inaccesibles. Cuando el sistema de ficheros montado se desmonte (`umount`), los contenidos originales volverán a estar visibles.

---

### 16. Política de backups (Primer domingo: Total; Resto de domingos: Nivel 1; Resto de días: Nivel 2):
- **a) ¿Qué copia toma como referencia la realizada el viernes?**  
  Una copia de Nivel $N$ respalda los cambios ocurridos desde la última copia de nivel estrictamente inferior ($< N$). Por tanto, la copia de nivel 2 realizada el viernes toma como referencia la copia de **Nivel 1 del domingo anterior**.
- **b) Esquema para que las copias diarias respalden diferencias respecto al día anterior sin usar el término incremental:**  
  Para que cada día tome como base el día inmediatamente anterior, el nivel debe ser estrictamente incremental cada jornada:
  - Domingo: Nivel 1.
  - Lunes: Nivel 2.
  - Martes: Nivel 3.
  - Miércoles: Nivel 4.
  - Jueves: Nivel 5.
  - Viernes: Nivel 6.
  - Sábado: Nivel 7.

---

### 17. RTO y RPO en copias de seguridad.

**Respuesta:**  
- **RPO (Recovery Point Objective):** Tiempo máximo de datos que la organización puede permitirse perder. Determina directamente la **frecuencia** con la que se deben realizar las copias (a menor RPO, copias más frecuentes).
- **RTO (Recovery Time Objective):** Tiempo máximo admisible para restaurar el servicio tras un fallo.
- Para minimizar el RTO se deben utilizar **copias totales o copias acumulativas/diferenciales**, ya que para restaurar solo se requiere recuperar la copia total y la última copia diferencial.

---

### 18. Dominio, Árbol y Bosque en Directorio Activo.

**Respuesta:**  
- **Dominio:** Unidad administrativa y límite de seguridad básico que agrupa ordenadores y cuentas que comparten una misma base de datos de directorio.
- **Árbol:** Conjunto jerárquico de uno o más dominios que comparten un espacio de nombres DNS contiguo y mantienen relaciones de confianza transitivas bidireccionales.
- **Bosque:** Conjunto de uno o varios árboles de dominios independientes que comparten el mismo esquema de directorio, catálogo global y configuración común.

---

### 19. ¿Qué tipo de directiva se debe establecer en GPO para impedir apagar el equipo sin haber iniciado sesión?

**Respuesta:**  
Es una directiva de **Configuración de equipo** (*Computer Configuration*), concretamente una directiva de seguridad local perteneciente a la categoría de **Opciones de seguridad** (*Security Options*).

---

### 20. Describa los estados de una directiva de grupo (GPO) y su relación con conflictos entre directivas.

**Respuesta:**  
- **Estados:**  
  1. **Habilitada (Enabled):** La directiva se aplica explícitamente.  
  2. **Deshabilitada (Disabled):** La directiva se desactiva explícitamente.  
  3. **No configurada (Not Configured):** No se toma ninguna decisión en esa GPO.  
- **Resolución de conflictos:** El estado "No configurada" no genera conflictos y permite que prevalezcan los valores heredados. Cuando varias directivas entran en conflicto con valores contradictorios (Habilitada frente a Deshabilitada), el conflicto se resuelve según el orden de precedencia estándar de Active Directory: **LSDOU** (*Local, Sitio, Dominio, Unidad Organizativa*), prevaleciendo siempre la directiva más cercana al objeto.