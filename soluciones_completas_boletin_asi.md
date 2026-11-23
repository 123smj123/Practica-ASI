# BOLETÍN DE TAREAS - ADMINISTRACIÓN DE SISTEMAS DE INFORMACIÓN (ASI)
**Dpto. Tecnología Electrónica - Universidad de Sevilla**  
*Curso 2026/27*

---

# TEMA 2: Shell Scripts y Entorno Bash

### 1. Ver ficheros de arranque y comprobar que contiene secuencias de órdenes que se irán ejecutando cuando el usuario inicia sesión.
**Respuesta:**
Para examinar los scripts de arranque del sistema y personales:
```bash
cat /etc/profile
cat /etc/bash.bashrc
cat ~/.profile
cat ~/.bashrc
```
En estos ficheros encontramos secuencias de órdenes en Bash tales como comprobaciones condicionales (`if [ -f ... ]`), configuración de variables de entorno globales y de sesión (`PATH`, `PS1`, `UMASK`) e invocaciones a funciones y alias.

---

### 2. Añada la orden `echo "Esta es una login-shell"` al final del fichero de arranque personal asociado a la "login-shell". Añada `echo "Esta es una non-login shell"` al final del fichero de arranque personal asociado a la "non-login shell". Pruebe a abrir una ventana de terminal ¿Qué mensaje aparece? ¿De qué tipo de inicio de sesión estamos hablando? Pruebe ahora a iniciar sesión en "tty2" (use Ctrl-Alt F3) ¿Qué mensaje aparece? ¿De qué tipo de inicio de sesión estamos hablando? Nota: Puede volver al interfaz gráfico con Ctrl-Alt F1.
**Respuesta:**
1. Modificación de los ficheros:
   ```bash
   echo 'echo "Esta es una login-shell"' >> ~/.profile
   echo 'echo "Esta es una non-login shell"' >> ~/.bashrc
   ```
2. **Ventana de terminal en entorno gráfico:**
   - Mensaje que aparece: `"Esta es una non-login shell"`.
   - Tipo de inicio de sesión: **Non-login shell interactiva** (no requiere autenticación completa de usuario para abrirse).
3. **Inicio de sesión en terminal tty3 (Ctrl+Alt+F3):**
   - Mensajes que aparecen: Se muestran **ambos mensajes**:
     ```
     Esta es una non-login shell
     Esta es una login-shell
     ```
   - Tipo de inicio de sesión: **Login shell** (se solicitan credenciales `login` y `password`).

---

### 3. Habrá observado que en uno de los casos del apartado anterior aparecen los dos mensajes. Intente descubrir el motivo (pista: mire el contenido de los scripts de arranque).
**Respuesta:**
Al inspeccionar el fichero `~/.profile` con `cat ~/.profile`, se observa que incluye el siguiente bloque:
```bash
if [ -n "$BASH_VERSION" ]; then
    # include .bashrc if it exists
    if [ -f "$HOME/.bashrc" ]; then
        . "$HOME/.bashrc"
    fi
fi
```
Al tratarse de una *login shell*, Bash lee `~/.profile`, y este ejecuta internamente (*source* mediante el operador `.`) el fichero `~/.bashrc`. Por ello, se imprime primero el mensaje de `~/.bashrc` y a continuación el de `~/.profile`.

---

### 4. Abra una nueva ventana de terminal y ejecute `ls -la` observe la salida que se muestra (puede usar la rueda del ratón para ver todo el contenido). ¿Cuántos directorios existen?
**Respuesta:**
```bash
ls -la
```
Para determinar el número exacto de directorios existentes en la salida (incluyendo los enlaces especiales `.` y `..`), basta con filtrar aquellas líneas cuyo primer carácter en la columna de permisos sea una `d`:
```bash
ls -la | grep "^d" | wc -l
```

---

### 5. Observará que hay dos entradas con un nombre especial: `.` y `..` ¿se trata de ficheros o de directorios?
**Respuesta:**
Se trata de **directorios**, tal y como indica la letra `d` en la primera posición de la cadena de permisos (`drwxr-xr-x`):
- `.` representa el directorio actual de trabajo.
- `..` representa el directorio padre (inmediatamente superior en la jerarquía).

---

### 6. Teclee `ls -la <nombre_directorio>`, donde `<nombre_directorio>` se corresponde con uno de los directorios (sin contar `.` y `..`) que observó en la tarea 4. ¿Qué le muestra ahora la salida del programa?
**Respuesta:**
```bash
ls -la Descargas
```
Muestra el listado en formato largo de todos los ficheros y subdirectorios (incluidos los ocultos) contenidos **dentro** de dicho subdirectorio.

---

### 7. Teclee `ls -la ./` ¿es la salida igual que alguna de las tareas anteriores? ¿por qué cree que ocurre esto?
**Respuesta:**
```bash
ls -la ./
```
Sí, la salida es idéntica a la obtenida con `ls -la` en la tarea 4. Ocurre porque `./` hace referencia al directorio actual, que es el objetivo por defecto de `ls` cuando no se especifica ruta.

---

### 8. ¿Qué muestra la salida de `ls -la ..`?
**Respuesta:**
```bash
ls -la ..
```
Muestra el contenido detallado del directorio padre del actual (si nos encontramos en `/home/usuario`, muestra el contenido de `/home`, listando las carpetas del resto de usuarios).

---

### 9. Teclee `cd` y a continuación `pwd`. La salida de este último le mostrará cuál es el directorio principal (llamado HOME) del usuario con el que ha iniciado sesión. Anote ese directorio.
**Respuesta:**
```bash
cd
pwd
```
Salida devuelta:
```
/home/<nombre_usuario>
```

---

### 10. Muestre el contenido de la variable de entorno PATH mediante `echo $PATH`. ¿Se buscará en un directorio llamado `mis_scripts` que cuelga de su directorio de usuario (anotado en el paso anterior)?
**Respuesta:**
```bash
echo $PATH
```
Salida típica:
`/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin`  
**No se buscará**, ya que el directorio `/home/<usuario>/mis_scripts` no se encuentra explícitamente incluido entre las rutas separadas por dos puntos (`:`) de `$PATH`.

---

### 11. Si quisiera ejecutar un script llamado "tarea11" sin modificar la variable PATH, ¿cómo lo haría? (ponga un ejemplo de uso mediante un path relativo y mediante un path absoluto).
**Respuesta:**
- **Path relativo:** `./tarea11` (o `../mis_scripts/tarea11` según la posición actual).
- **Path absoluto:** `/home/usuario/mis_scripts/tarea11` (o `$HOME/mis_scripts/tarea11`).

---

### 12. Las páginas de manual siempre siguen el mismo formato. En cada una de ellas se describe brevemente para qué sirve ese programa, se definen los parámetros y, finalmente, se describen cada una de las opciones con las que se puede invocar dicho programa. Visualice la ayuda de "ls" e indique: Para qué sirve esa orden; El significado de las opciones -a, -l y -d.
**Respuesta:**
```bash
man ls
```
- **Utilidad:** Lista información sobre ficheros y directorios (el directorio actual por defecto).
- **`-a` (`--all`):** Muestra todos los ficheros, incluyendo los ocultos (aquellos que comienzan por un punto `.`).
- **`-l`:** Formato largo; muestra permisos, enlaces, propietario, grupo, tamaño en bytes y fecha de modificación.
- **`-d` (`--directory`):** Lista los directorios como entradas propias en lugar de listar su contenido.

---

### 13. Una de las cuestiones que se pueden ver en las páginas de manual es el número de parámetros que se pueden usar para cada programa y si son obligatorios o no. Los parámetros opcionales se ponen entre corchetes ([]). Compruebe las páginas de manual de "ls" y de "cp" e indique en qué casos es posible invocar ese programa sin parámetros y en qué casos hay parámetros que son obligatorios.
**Respuesta:**
- **`ls`:** En `man ls` la sintaxis es `ls [OPTION]... [FILE]...`. Como todos los argumentos aparecen entre corchetes `[...]`, se puede invocar **sin ningún parámetro**, tomando el directorio actual por defecto.
- **`cp`:** En `man cp` la sintaxis es `cp [OPTION]... SOURCE... DIRECTORY` o `cp [OPTION]... SOURCE DEST`. Los parámetros `SOURCE` y `DEST`/`DIRECTORY` van sin corchetes; son **obligatorios** (se requieren al menos dos operandos).

---

### 14. Teclee `cd` y a continuación `pwd`. Con la primera orden habrá indicado a la shell que desea cambiar al directorio principal (HOME) del usuario y con el segundo se estará mostrando el directorio de trabajo actual (print working directory).
**Respuesta:**
```bash
cd
pwd
```
Se retorna a `/home/usuario`.

---

### 15. Use "cd" para cambiar al directorio "Descargas" ("Downloads" si el sistema operativo se instaló en inglés) que cuelga de su directorio principal. Compruebe mediante "pwd" que todo ha ido bien. Teclee ahora `cd ..`, ¿en qué directorio cree que va a estar? (compruébelo), teclee ahora `cd -` ¿en qué directorio cree que va a estar? (compruébelo). ¿Cuál es la vía (o path) de acceso absoluta del directorio padre de su directorio de trabajo actual? ¿Cómo podría cambiar a ese directorio usando un path relativo? (compruebe que funciona). ¿A qué directorio se cambiará si teclea `cd /root`? ¿funciona? ¿A qué directorio cambiará si teclea `cd nombre_usuario` (donde nombre_usuario es el "login" que ha usado)? ¿funciona?
**Respuesta:**
```bash
cd ~/Descargas
pwd              # /home/usuario/Descargas
cd ..
pwd              # Se regresa a /home/usuario
cd -
pwd              # Se regresa al directorio previo: /home/usuario/Descargas
```
- **Path absoluto del directorio padre actual:** `/home/usuario`.
- **Cambio con path relativo:** `cd ..`
- **`cd /root`:** Intenta entrar al directorio de root; **no funciona** para un usuario normal (error: `Permiso denegado`).
- **`cd nombre_usuario`:** Intenta entrar en un subdirectorio llamado así bajo la ruta actual. **No funciona** (a no ser que exista una carpeta local con ese nombre exacto; no debe confundirse con `cd ~nombre_usuario`).

---

### 16. Vuelva a su directorio de usuario y teclee `ls` compare esa salida con la de `ls -a` e indique las diferencias observadas.
**Respuesta:**
```bash
cd ~
ls
ls -a
```
Con `ls` solo se observan los ficheros y directorios visibles. Con `ls -a` se observan además todas las entradas ocultas (las que comienzan por punto, como `.bashrc`, `.profile`, `.config`, así como `.` y `..`).

---

### 17. Compruebe si mediante `ls -l` se muestran los ficheros y directorios que empiezan por ".". ¿Qué ocurre si se usan las dos opciones a la vez (`ls -la`)?
**Respuesta:**
```bash
ls -l
ls -la
```
Con `ls -l` **no se muestran** los ficheros que empiezan por `.`. Al combinar ambas opciones (`ls -la`), se muestran todos los archivos (incluidos los ocultos) en formato detallado.

---

### 18. Muestre, de forma recursiva, el contenido de un directorio que empiece por ".".
**Respuesta:**
```bash
ls -laR ~/.config
```

---

### 19. ¿Qué estaré haciendo si tecleo `cd ~`?
**Respuesta:**
Cambiar de forma inmediata al directorio personal (`$HOME`) del usuario activo.

---

### 20. Muévase al directorio raíz (`cd /`) y compruebe las diferencias entre `ls -la /home` y `ls -lad /home`.
**Respuesta:**
```bash
cd /
ls -la /home
ls -lad /home
```
- `ls -la /home`: Lista todo el contenido interior del directorio `/home` (las carpetas personales de los usuarios).
- `ls -lad /home`: Muestra única y exclusivamente la línea descriptiva del propio directorio `/home` (sus permisos, propietario `root`, inodo y tamaño), sin inspeccionar su interior.

---

### 21. SIN PROBARLO, indique qué ocurre si tecleo lo siguiente `rm -rf .`, `rm -rf ./`, `rm -rf /.` (ojo, algunas de estas órdenes borran completamente el disco).
**Respuesta:**
- `rm -rf .` y `rm -rf ./`: Bash y `rm` moderno abortan la ejecución con el error `"rm: refusing to remove '.' or '..' directory: skipping '.'"` para evitar la autodestrucción accidental del directorio actual.
- `rm -rf /.` : La shell expande la ruta al directorio raíz (`/`). `rm` procederá a borrar recursivamente y sin confirmación todos los ficheros y sistemas de ficheros montados en el sistema operativo, dejando la máquina inoperativa.

---

### 22. Compruebe empíricamente qué diferencias existen entre `cat /etc/passwd` y `more /etc/passwd`.
**Respuesta:**
- `cat /etc/passwd`: Vuelca la totalidad del fichero en la salida estándar de un solo golpe, haciendo scroll hasta el final.
- `more /etc/passwd`: Pagina el contenido pantalla a pantalla, permitiendo avanzar progresivamente pulsando la barra espaciadora o la tecla `Enter`.

---

### 23. Muévase a su directorio de usuario y cree, mediante una sola orden, un directorio llamado "dir1" y dentro de éste un directorio llamado "dir2".
**Respuesta:**
```bash
cd ~
mkdir -p dir1/dir2
```

---

### 24. Cree un fichero vacío llamado "mifich" en "~/dir1/dir2". Anote lo que ha tecleado.
**Respuesta:**
```bash
touch ~/dir1/dir2/mifich
```

---

### 25. Obtenga un listado recursivo del directorio "dir1" que cuelga de su directorio de usuario (a veces le llamaremos a ese directorio $HOME o, simplemente, "home del usuario"). Indique el tamaño y la fecha que tienen el fichero "mifich" creado anteriormente.
**Respuesta:**
```bash
ls -lR ~/dir1
```
- **Tamaño:** 0 bytes.
- **Fecha:** La marca de tiempo del momento exacto en el que se ejecutó el comando `touch`.

---

### 26. Averigüe dónde está el ejecutable del programa "ls". Muestre el tipo de fichero del que se trata. Indique también el tipo de fichero de "/etc/passwd". Averigüe también la ubicación del programa "hostname".
**Respuesta:**
```bash
which ls
file /usr/bin/ls
file /etc/passwd
which hostname
```
- Ubicación de `ls`: `/usr/bin/ls` (o `/bin/ls`).
- Tipo de `/usr/bin/ls`: `ELF 64-bit LSB pie executable, x86-64... dinamically linked`.
- Tipo de `/etc/passwd`: `ASCII text`.
- Ubicación de `hostname`: `/usr/bin/hostname` (o `/bin/hostname`).

---

### 27. ¿Por qué no funciona `which cd` si es lo que nos sirve para cambiar de directorio?
**Respuesta:**
No funciona porque `cd` no es un programa binario externo en disco, sino una orden interna integrada de la propia shell (*shell builtin*). Si fuera un binario externo, cambiaría el directorio en un proceso hijo independiente y el proceso padre (nuestra shell) no alteraría su ruta.

---

### 28. Compruebe la salida del programa "w".
**Respuesta:**
```bash
w
```
Muestra la hora actual, el tiempo que el sistema lleva encendido (*uptime*), el número de usuarios conectados, la media de carga (*load average* en 1, 5 y 15 minutos), y la lista de usuarios con su terminal (`TTY`), IP remota, tiempo de inactividad (`IDLE`) y el comando que están ejecutando (`WHAT`).

---

### 29. Muévase al directorio raíz y compruebe qué ocurre (indique qué directorios se muestran) si se teclea `ls -lad s*`. Indique ahora por qué `ls -lad s??` muestra dos directorios mientras que `ls -lad s???` también muestra dos pero no incluye los del comando anterior (`ls -lad s??`).
**Respuesta:**
```bash
cd /
ls -lad s*
```
- `ls -lad s*`: Muestra todas las entradas que comienzan por `s` (`sbin`, `srv`, `sys`, `snap`...).
- `ls -lad s??`: El patrón busca nombres que empiecen por `s` seguidos de exactamente **dos** caracteres adicionales (longitud 3). Coincide con `srv` y `sys`.
- `ls -lad s???`: Busca nombres que comiencen por `s` seguidos de exactamente **tres** caracteres adicionales (longitud 4). Coincide con `sbin` y `snap`. No incluye los anteriores porque tienen 3 caracteres y no 4.

---

### 30. ¿Qué ocurre si tecleo `ls -las`? (indique las diferencias respecto a lo que hizo en la tarea anterior).
**Respuesta:**
```bash
ls -las
```
En este caso, la `s` no actúa como comodín de nombre de archivo sino como la **opción `-s`** de `ls` (mostrar el tamaño en bloques asignados a cada fichero). Lista todo el directorio actual en formato largo y con conteo de bloques.

---

### 31. ¿Cuál de estas opciones cree que es la más adecuada si se desea mostrar el listado (largo) de su directorio de usuario?:  
Opción a: `ls -la ~`  
Opción b: `ls -la ~/*`  
Nota: Indique las diferencias entre opción a y b.
**Respuesta:**
La más adecuada es la **Opción a** (`ls -la ~`).
- **Opción a:** Lista el directorio `$HOME`, incluyendo ficheros ocultos y subdirectorios de primer nivel sin expandirlos.
- **Opción b:** La shell expande `~/*` a todos los ficheros *no ocultos*, y `ls` procede a listar el contenido interno completo de cada uno de los subdirectorios existentes, produciendo una salida masiva e ignorando los ficheros ocultos de la raíz del HOME.

---

### 32. Elimine el directorio creado en el paso 23 y todo su contenido (nota: debe hacerlo mediante una sola orden).
**Respuesta:**
```bash
rm -rf ~/dir1
```

---

### 33. Compruebe si existe un alias para "ls" y si existe indique qué hace. Pruébelo mediante `ls -la`.
**Respuesta:**
```bash
alias ls
```
Habitualmente está definido como: `alias ls='ls --color=auto'`. Su función es colorear la salida de los ficheros según su tipo (directorios azul, ejecutables verde, etc.).

---

### 34. ¿Qué ocurre si intento "eliminar" el alias de "ls" mediante `alias ls=`?
**Respuesta:**
```bash
alias ls=
```
No elimina el alias; define un nuevo alias donde el comando `ls` queda asignado a una cadena vacía. Al invocar `ls`, la shell no ejecutará nada.

---

### 35. Utilice la orden "unalias" para deshacer el alias de "ls". Ejecute ahora `ls -la` e indique las diferencias observadas con el paso 33.
**Respuesta:**
```bash
unalias ls
ls -la
```
El alias queda completamente destruido. Al ejecutar `ls -la`, la salida se muestra en texto plano monocromático, sin ningún color.

---

### 36. Establezca una variable PRUEBA1 al valor "MiPrueba". Compruebe que funciona tecleando `echo $PRUEBA1`.
**Respuesta:**
```bash
PRUEBA1="MiPrueba"
echo $PRUEBA1
```

---

### 37. Pruebe a listar el contenido de las variables de entorno que están establecidas en su shell. Compruebe el valor de alguna de ellas: HOSTNAME, PATH y PS1.
**Respuesta:**
```bash
env
echo $HOSTNAME
echo $PATH
echo $PS1
```

---

### 38. En la ventana donde ha establecido la variable PRUEBA1, abra una nueva shell (por ejemplo, puede probar a abrir un terminal mediante "gnome-terminal" o simplemente arrancar una shell mediante "bash"). Ejecute entonces `echo $PRUEBA1`, ¿funciona? Indique la razón por la que no ha funcionado.
**Respuesta:**
```bash
bash
echo $PRUEBA1
```
No muestra nada (cadena vacía). No ha funcionado porque `PRUEBA1` fue creada como una variable local de la shell y los procesos hijos no heredan variables que no hayan sido exportadas.

---

### 39. Cierre la shell abierta en el apartado anterior y realice los pasos necesarios para que la variable PRUEBA1 sea "promocionada" a la categoría de variable de entorno. Repita el apartado anterior y compruebe si PRUEBA1 se ha heredado correctamente a la nueva shell.
**Respuesta:**
```bash
exit
export PRUEBA1
bash
echo $PRUEBA1
```
Ahora sí devuelve `MiPrueba`, ya que `export` copia la variable a la tabla de entorno transmitida en el `fork`/`exec`.

---

### 40. Pruebe ahora a abrir un nuevo terminal usando el interfaz gráfico. Compruebe que la variable PRUEBA1 ahora no está establecida. ¿Por qué ocurre eso si PRUEBA1 es una variable de entorno?
**Respuesta:**
Ocurre porque las variables de entorno solo se heredan en el árbol de procesos descendientes del proceso que las exportó. Una nueva ventana gráfica es hija del gestor de ventanas/escritorio, no de la terminal anterior. Para hacerla persistente en cualquier terminal debe escribirse en `~/.bashrc`.

---

### 41. ¿Qué ocurre si teclea `VAR1=Hola usuario`? Comprobará que da un error y que la variable no está establecida al valor esperado. ¿Por qué ocurre eso? Soluciónelo.
**Respuesta:**
Da el error `usuario: command not found`. La shell interpreta el espacio en blanco como el fin de la asignación e intenta ejecutar `usuario` como un comando pasando `VAR1=Hola` en su entorno local.  
**Solución:** Encerrar el texto entre comillas:
```bash
VAR1="Hola usuario"
```

---

### 42. ¿Qué diferencias existen entre?:  
- `var=hostname`  
- `var=` `hostname` ` ` (o `var=$(hostname)`)  
- `var=$hostname`
**Respuesta:**
- `var=hostname`: Almacena literalmente el texto `"hostname"` en la variable.
- `var=\`hostname\`` o `var=$(hostname)`: Ejecuta el comando `hostname` mediante sustitución de comandos y guarda su salida (el nombre del equipo).
- `var=$hostname`: Asigna a `var` el valor que tenga actualmente la variable llamada `hostname`.

---

### 43. ¿Qué ocurre si tecleo `mkdir Un directorio con espacios y un *`? Compruebe que con el uso de " o de \ se puede lograr nuestro propósito.
**Respuesta:**
Crea múltiples carpetas individuales (`Un`, `directorio`, `con`, `espacios`, `y`, `un`) y expande `*` creando directorios por cada elemento local.  
**Solución correcta:**
```bash
mkdir "Un directorio con espacios y un *"
# O usando escapes:
mkdir Un\ directorio\ con\ espacios\ y\ un\ \*
```

---

### 44. ¿Cómo podría crear un fichero que contenga el carácter "\"?
**Respuesta:**
Escapándolo con otra barra inclinada o entrecomillándolo:
```bash
touch "fichero\conbarra"
# O bien:
touch fichero\\conbarra
```

---

### 45. ¿Se creará algún fichero si se teclea `touch *`? ¿Qué es lo que ha ocurrido al teclear lo anterior?
**Respuesta:**
No crea un fichero llamado `*`. La shell expande el comodín `*` a los nombres de todos los ficheros y directorios existentes en la ruta actual, por lo que `touch` actualiza la fecha de modificación de todos ellos. Si el directorio estuviese completamente vacío, crearía un fichero llamado `*`.

---

### 46. ¿Cómo puedo crear un fichero de nombre "*"? ¿Cómo puedo borrar un fichero que se llame así? Compruébelo.
**Respuesta:**
```bash
touch "./*"      # O bien: touch \*
rm -- "./*"      # O bien: rm \*
```

---

### 47. Desde su directorio de usuario, observe la diferencia entre `echo "*"` y `echo *` ¿A qué es debida la diferencia?
**Respuesta:**
- `echo "*"`: Imprime textualmente el carácter asterisco `*`.
- `echo *`: La shell expande el comodín e imprime la lista de todos los ficheros del directorio actual.  
La diferencia se debe a que las comillas desactivan la expansión de comodines (*globbing*) de la shell.

---

### 48. Pruebe a ejecutar `cat` sin ningún parámetro. ¿Qué ocurre? ¿A dónde está redirigida "stdin" y "stdout"?
**Respuesta:**
```bash
cat
```
El cursor se queda parpadeando esperando datos.  
- `stdin` (fd 0) está conectado al **teclado** (la terminal).
- `stdout` (fd 1) está conectado a la **pantalla**.  
Todo lo que se escribe en el teclado se devuelve inmediatamente reflejado por pantalla. Se finaliza con `Ctrl+D`.

---

### 49. ¿Qué ocurre si se teclea `cat /etc/passwd > /tmp/fich.txt`? ¿A dónde se redirige ahora la salida estándar? ¿Y la de errores?
**Respuesta:**
El contenido de `/etc/passwd` se escribe dentro de `/tmp/fich.txt` sin mostrar nada por pantalla.
- `stdout` (fd 1) se redirige al fichero `/tmp/fich.txt`.
- `stderr` (fd 2) no se redirige; sigue conectada a la pantalla.

---

### 50. Pruebe ahora a teclear `cat > /tmp/fich1.txt`. Comprobará que no vuelve a obtener el prompt de la shell. Teclee cualquier cosa (bastará con un par de líneas) y teclee "Ctrl-D" para terminar. ¿De dónde se está tomando ahora la entrada estándar? ¿Y la salida estándar, a dónde se redirige?
**Respuesta:**
- `stdin` (fd 0) se toma del **teclado**.
- `stdout` (fd 1) se redirige al fichero `/tmp/fich1.txt`.

---

### 51. Compruebe ahora qué ocurre si se teclea `echo "Hola" >> /tmp/fich1.txt` (use el propio programa "cat" para ver qué ha pasado).
**Respuesta:**
```bash
echo "Hola" >> /tmp/fich1.txt
cat /tmp/fich1.txt
```
El operador `>>` concatena (*append*) la palabra `"Hola"` al final del fichero respetando las líneas escritas en la tarea 50.

---

### 52. ¿Qué ocurre si se teclea `cat < ~/.profile`? ¿Cuál es, en este caso, la entrada estándar y la salida estándar?
**Respuesta:**
- `stdin` (fd 0) se redirige desde el fichero `~/.profile`.
- `stdout` (fd 1) sigue apuntando a la **pantalla**.  
El comando muestra el contenido de `~/.profile`.

---

### 53. Sin hacerlo, ¿qué cree que pasaría si se tecleara `cat > ~/.profile`?
**Respuesta:**
El operador `>` trunca el fichero destino a longitud 0 inmediatamente. El fichero `~/.profile` quedaría completamente borrado/en blanco esperando entrada por teclado, destruyendo la configuración del usuario.

---

### 54. ¿Qué hace la siguiente orden `cat /etc/passwd >> ~/fichero`? Compruebe que está en lo cierto.
**Respuesta:**
Añade el contenido completo de `/etc/passwd` al final de `~/fichero` (si no existe, lo crea; si existe, anexa la información sin sobreescribir lo previo).

---

### 55. ¿Qué significa (respecto a la configuración de la salida estándar y la entrada estándar) `ls -la | more`? Realice un esquema que muestre como se conectan "stdin" y "stdout" en ambos programas (cat y more).
**Respuesta:**
Significa que la salida estándar (`stdout`) de `ls -la` se conecta directamente a la entrada estándar (`stdin`) de `more` mediante un pipe en memoria del kernel:
```
+-----------+ stdout (fd 1)        stdin (fd 0) +------------+
|   ls -la  | ------------> [PIPE] -----------> |    more    | ---> Pantalla
+-----------+                                   +------------+
```

---

### 56. ¿Qué ocurre (y por qué ocurre) al teclear `ls -laR / | cat > /tmp/salida.txt`? ¿Qué ocurre en este caso con la salida de errores de "ls"?
**Respuesta:**
`ls` recorre recursivamente toda la raíz y su salida se pasa por tubería a `cat`, que la redirige a `/tmp/salida.txt`.  
La salida de errores (`stderr`, fd 2) no entra en la tubería y salta directamente a la **pantalla**, mostrando múltiples mensajes de `"Permiso denegado"`.

---

### 57. ¿Qué ocurre si tecleo `more /dev/zero`? Indique por qué ocurre.
**Respuesta:**
`/dev/zero` es un dispositivo especial de caracteres que genera un flujo infinito de bytes nulos (`\0`). La terminal se satura mostrando caracteres en blanco de forma continua sin llegar nunca al final. Debe cancelarse con `Ctrl+C`.

---

### 58. ¿Qué ocurre si tecleo `ls -laR / > /dev/null`? ¿Por qué salen mensajes por pantalla, a pesar de tener redirigida la salida estándar? ¿De qué forma puedo ejecutar la orden anterior pero descartando también los mensajes de error?
**Respuesta:**
Salen mensajes porque solo se redirigió `stdout` (fd 1); los errores van por `stderr` (fd 2), que sigue en la terminal.  
Para descartar ambos flujos:
```bash
ls -laR / > /dev/null 2>&1
# O bien:
ls -laR / &> /dev/null
```

---

### 59. Cree un fichero tipo "tubería" llamado /tmp/tubo1. Use "ls" para obtener el listado extendido con la información de ese fichero. En concreto queremos saber cómo se sabe que es un fichero tipo "tubería" (además de por el color de la fuente) y el tamaño del fichero.
**Respuesta:**
```bash
mkfifo /tmp/tubo1
ls -l /tmp/tubo1
```
- Se identifica como tubería con nombre (FIFO) porque el primer carácter de los permisos es una **`p`** (`prw-r--r--`).
- Su tamaño en disco es **0 bytes**, ya que actúa como canal de comunicación en memoria del kernel.

---

### 60. Teclee `cat /etc/passwd > /tmp/tubo1`. ¿Qué está haciendo? Compruebe si ha recuperado el prompt del sistema. ¿Qué tamaño tiene ahora /tmp/tubo1?
**Respuesta:**
Está escribiendo en la FIFO. El comando se bloquea y **no se recupera el prompt**, porque las tuberías con nombre son síncronas: el escritor se bloquea hasta que un lector abra el otro extremo. Su tamaño sigue siendo de 0 bytes.

---

### 61. En el apartado anterior se ha observado que no se devuelve el prompt porque aunque estoy "llenando" la tubería mediante el programa "cat" no hay nadie "vaciándola" y por esa razón el programa "cat" no termina. Abra otro terminal y pruebe a ver el contenido de /tmp/tubo1 mediante "cat". ¿Qué aparece por pantalla? ¿Ha finalizado el programa "cat" del apartado anterior?
**Respuesta:**
```bash
cat /tmp/tubo1
```
Aparece por pantalla el contenido de `/etc/passwd`. En el primer terminal el proceso escritor finaliza con éxito y devuelve inmediatamente el prompt.

---

### 62. Vea la página de manual de "find" e indique para qué sirve "cmin".
**Respuesta:**
```bash
man find
```
`-cmin n`: Evalúa si el estado del archivo (*change time* de inodo: permisos, dueño, tamaño, etc.) fue modificado hace exactamente $n$ minutos (`+n` hace más de $n$ minutos, `-n` hace menos de $n$ minutos).

---

### 63. ¿Es correcto teclear `find . -name * -print` para listar de forma recursiva todos los ficheros y directorios empezando por el directorio de trabajo? Si no funciona indique qué es lo que ocurre y cómo solucionarlo.
**Respuesta:**
No es correcto. La shell expande el comodín `*` a los archivos del directorio de trabajo antes de pasárselo a `find`, provocando errores de sintaxis si hay más de un archivo.  
**Solución:** Escapar o entrecomillar el asterisco:
```bash
find . -name "*" -print
# O bien:
find . -name \* -print
```

---

### 64. Cree un fichero vacío (da igual el nombre) en su directorio de usuario. A continuación use find (de forma repetida) para comprobar que "cmin" funciona como se espera (use en sus pruebas cmin -2, cmin 2 y cmin +2).
**Respuesta:**
```bash
cd ~
touch prueba_cmin.txt
find . -name "prueba_cmin.txt" -cmin -2   # Lo encuentra (menos de 2 min)
# Esperar a los 2 minutos justos:
find . -name "prueba_cmin.txt" -cmin 2
# Tras pasar más de 2 minutos:
find . -name "prueba_cmin.txt" -cmin +2  # Lo encuentra (más de 2 min)
```

---

### 65. Use "find" para listar (de forma recursiva) todos los ficheros y directorios de su directorio de usuario que hayan sido modificados en los últimos 7 días.
**Respuesta:**
```bash
find ~ -mtime -7
```

---

### 66. Repita lo anterior pero en este caso para mostrar los ficheros que se hayan modificado en los últimos 5 minutos.
**Respuesta:**
```bash
find ~ -type f -mmin -5
```

---

### 67. ¿Cómo puedo mostrar las líneas del fichero /etc/passwd que contienen la cadena "bash"?
**Respuesta:**
```bash
grep "bash" /etc/passwd
```

---

### 68. ¿Cómo puedo mostrar las líneas del fichero /etc/passwd que no contienen la cadena "bash"?
**Respuesta:**
```bash
grep -v "bash" /etc/passwd
```

---

### 69. ¿Cuál sería la expresión regular que permita identificar una dirección de correo electrónico? Nota: Considere que un correo electrónico está formado por USUARIO@DOMINIO donde USUARIO es cualquier cadena de 1 o más caracteres o números y DOMINIO se corresponde con un conjunto (dos o más) de cadenas de uno o más caracteres o números separados por el carácter ".". Use "grep" para probar que funciona.
**Respuesta:**
Expresión regular extendida:
```bash
grep -E '^[a-zA-Z0-9]+@([a-zA-Z0-9]+\.){1,}[a-zA-Z0-9]+$' fichero_pruebas.txt
```

---

### 70. Teniendo en cuenta que el separador de campos del fichero /etc/passwd es ":", obtenga un listado que contenga solamente la lista de todos los nombres de usuario ordenados alfabéticamente.
**Respuesta:**
```bash
cut -d: -f1 /etc/passwd | sort
```

---

### 71. Realice un listado de las distintas shell (último campo) que aparecen en el fichero /etc/passwd así como el número de veces que aparece cada una.
**Respuesta:**
```bash
cut -d: -f7 /etc/passwd | sort | uniq -c
```

---

### 72. Una de las opciones más usada de "tail" es la opción "-f" que nos permite monitorizar un fichero. Esto suele hacerse con los denominados ficheros de log (que muestran mensajes de actividad del sistema) como "/var/log/syslog". Ejecute `tail -f /var/log/syslog` y compruebe que se muestran las últimas líneas de ese fichero. Compruebe además que el programa no termina (no tiene prompt de la shell) y que según se vaya escribiendo en ese fichero, los nuevos mensajes aparecerán en pantalla. Cuando termine, use Ctrl-C para salir del programa.
**Respuesta:**
```bash
tail -f /var/log/syslog
```
Muestra las últimas 10 líneas y mantiene abierto el descriptor; al generarse nuevos eventos en el sistema operativo, se imprimen inmediatamente en pantalla. Se interrumpe con `Ctrl+C`.

---

### 73. Muestre por pantalla el contenido del fichero /etc/passwd en el que todas las letras se han pasado a mayúsculas.
**Respuesta:**
```bash
tr '[:lower:]' '[:upper:]' < /etc/passwd
# O bien:
tr 'a-z' 'A-Z' < /etc/passwd
```

---

### 74. Realice un shell-script simple que muestre por pantalla el mensaje "Hola. Esto funciona". Asígnele permiso de ejecución (`chmod 755 nombre_fichero`) e intente ejecutarlo invocándolo con un path absoluto y con un path relativo.
**Respuesta:**
1. Crear `holamundo.sh`:
   ```bash
   cat << 'EOF' > holamundo.sh
   #!/bin/bash
   echo "Hola. Esto funciona"
   EOF
   ```
2. Dar permisos y probar:
   ```bash
   chmod 755 holamundo.sh
   ./holamundo.sh                  # Relativo
   ~/holamundo.sh                  # Absoluto (o /home/usuario/holamundo.sh)
   ```

---

### 75. Cree en su directorio de usuario un directorio llamado "bin" y mueva allí el shell-script creado en el apartado anterior. Modifique la variable de entorno PATH para que se pueda invocar ese shell-script sin necesidad de usar un path absoluto o relativo.
**Respuesta:**
```bash
mkdir -p ~/bin
mv holamundo.sh ~/bin/
export PATH="$HOME/bin:$PATH"
holamundo.sh
```

---

### 76. Realice los cambios del apartado anterior pero para que sean persistentes (es decir, que se sigan aplicando si el usuario reinicia sesión). Pruébelo.
**Respuesta:**
```bash
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

---

### 77. Realice un script que haga lo siguiente: Imprimir "El nombre del programa es: " y el nombre del script; Imprimir "En número de parámetros del programa es: " y el número de parámetros; Imprimir "El primer parámetro es: " y el valor del primer parámetro; Imprimir "El valor de la variable HOME es:" y el valor de HOME. Pruebe que el script funciona correctamente.
**Respuesta:**
Contenido de `info_params.sh`:
```bash
#!/bin/bash
echo "El nombre del programa es: $0"
echo "El número de parámetros del programa es: $#"
echo "El primer parámetro es: $1"
echo "El valor de la variable HOME es: $HOME"
```
Prueba:
```bash
chmod 755 info_params.sh
./info_params.sh Dos parámetros
```

---

### 78. Compruebe qué ocurre con la salida del script anterior si se invoca así: `./script *` ¿Por qué obtiene esa salida? ¿Es posible invocar al script de forma que el carácter "*" sea realmente el único parámetro del programa?
**Respuesta:**
- Al ejecutar `./info_params.sh *`, la shell realiza *globbing* y expande `*` como la lista de todos los ficheros del directorio actual. `$#` mostrará el número total de ficheros y `$1` el primer fichero de la lista.
- **Para que sea el único parámetro:** Debe entrecomillarse o escaparse para anular la expansión:
  ```bash
  ./info_params.sh "*"
  # O bien:
  ./info_params.sh \*
  ```

---

### 79. Realice un script que establezca una variable (use el nombre que quiera) y que posteriormente imprima por pantalla su valor. Compruebe qué ocurre si: Se invoca como `./script` y una vez terminado compruebe si la variable permanece; Se invoca tecleando `source ./script` o `. ./script` y compruebe si permanece.
**Respuesta:**
Contenido de `var_script.sh`:
```bash
#!/bin/bash
MI_VAR="PruebaScope"
echo "Dentro del script: $MI_VAR"
```
```bash
chmod 755 var_script.sh
./var_script.sh
echo $MI_VAR        # No muestra nada. Se ejecutó en una subshell hija.

source ./var_script.sh
echo $MI_VAR        # Muestra "PruebaScope". Se ejecutó en el contexto de la shell actual.
```

---

### 80. Realice un script que tome dos números como parámetros y que escriba el resultado de la suma de los dos números. Debe detectarse un uso incorrecto (más o menos parámetros de los indicados) y, en ese caso, mostrar un mensaje que indique cómo se usa el script.
**Respuesta:**
Contenido de `suma.sh`:
```bash
#!/bin/bash
if [ $# -ne 2 ]; then
    echo "Uso incorrecto del script."
    echo "Sintaxis: $0 <numero1> <numero2>"
    exit 1
fi

suma=$(($1 + $2))
echo "El resultado de la suma es: $suma"
```

---

### 81. Teclee el nombre de un programa que existe (p.ej. "ls"), ¿qué valor se obtiene en `$?`?
**Respuesta:**
```bash
ls
echo $?
```
Se obtiene **`0`** (código de terminación exitosa en Unix/Linux).

---

### 82. Teclee una orden que de error (p.ej. `ls esteficheroseguroquenoexiste`), ¿qué valor se obtiene en `$?`?
**Respuesta:**
```bash
ls esteficheroseguroquenoexiste
echo $?
```
Se obtiene un valor **distinto de 0** (normalmente `2` en `ls` por fallo de acceso al fichero).

---

### 83. Realice un script que, basándose en el fichero /etc/passwd, muestre el directorio de usuario (sexto parámetro) de todos los usuarios que usan bash como shell.
**Respuesta:**
Contenido de `usuarios_bash.sh`:
```bash
#!/bin/bash
grep '/bin/bash$' /etc/passwd | cut -d: -f6
```

---

# TEMA 3: Usuarios, Grupos y Permisos

### 84. Teclee `ls -l $HOME` y `ls -l /` e indique de qué usuario y grupo son los ficheros que aparecen listados.
**Respuesta:**
```bash
ls -l $HOME
ls -l /
```
- En `$HOME`: Pertenecen al usuario normal del sistema y a su grupo principal homónimo (ej. usuario `alumno`, grupo `alumno`).
- En `/`: Prácticamente la totalidad pertenecen al usuario `root` y al grupo `root`.

---

### 85. Usando la cuenta de usuario normal, intente ver el contenido del fichero /etc/shadow. ¿Tiene permiso? (la respuesta debe ser no).
**Respuesta:**
```bash
cat /etc/shadow
```
Respuesta: **No**. Devuelve `cat: /etc/shadow: Permiso denegado`. Sus permisos (`-rw-r-----` o `----------`) restringen el acceso a `root` y al grupo `shadow`.

---

### 86. Una de las formas de poder ver el contenido del fichero consiste en "promocionar" al usuario privilegiado y leerlo entonces. Pruebe con `sudo cat /etc/shadow` comprobará que si introduce la contraseña del usuario normal podrá ver el fichero. Eso es debido a que el usuario normal tiene permiso para usar "sudo".
**Respuesta:**
```bash
sudo cat /etc/shadow
```
Al introducir la contraseña del usuario normal (que pertenece al grupo `sudo`), `sudo` valida el permiso en `/etc/sudoers` y ejecuta `cat` con privilegios efectivos de `root` (UID 0), permitiendo leer el contenido cifrado.

---

### 87. Si vamos a realizar varias tareas como usuario privilegiado, podemos invocar "sudo" de esta forma: `sudo -s` para obtener una shell de usuario privilegiado. A partir de ahí, toda orden que se teclee desde esa shell será ejecutada como el usuario "root" ¿Existe algún cambio el prompt del sistema que le indique que ha tenido éxito al ejecutar "sudo"?
**Respuesta:**
```bash
sudo -s
```
Sí:
- El carácter final del prompt cambia de **`$`** (usuario ordinario) a **`#`** (superusuario `root`).
- El nombre del usuario en el prompt cambia a `root@equipo`.

---

### 88. Mire en la página de manual de "sudo" qué diferencias existen entre `sudo -s` y `sudo -i`.
**Respuesta:**
- `sudo -s` (*shell*): Arranca la shell indicada por la variable `SHELL` actual (o la de root) pero **no** simula un login completo; mantiene la mayor parte de las variables de entorno del usuario y no cambia el directorio de trabajo actual.
- `sudo -i` (*login*): Simula un inicio de sesión completo (`login shell`); lee los ficheros de arranque de root (`/root/.profile`, `/etc/profile`), cambia al directorio home de root (`/root`) y carga el entorno limpio del superusuario.

---

### 89. El equivalente a "sudo -i" usando "su" es `su -`. Pruebe a usarlo, ¿le funciona la contraseña del usuario normal para poder escalar privilegios? ¿por qué?
**Respuesta:**
```bash
su -
```
**No funciona** con la contraseña del usuario normal. `su` exige autenticarse introduciendo la contraseña de la cuenta destino (la contraseña de `root`). En distribuciones como Ubuntu la cuenta `root` viene bloqueada sin contraseña por defecto.

---

### 90. ¿Qué shell tiene el usuario "root"? ¿Cuál es su directorio?
**Respuesta:**
```bash
grep '^root:' /etc/passwd
```
- **Shell:** `/bin/bash`
- **Directorio HOME:** `/root`

---

### 91. ¿Qué shell tiene el usuario "bin"? ¿Cuál es su directorio? ¿Qué hace la shell del usuario "bin" (compruébelo ejecutándola)?
**Respuesta:**
```bash
grep '^bin:' /etc/passwd
```
- **Shell:** `/usr/sbin/nologin` (o `/bin/false`).
- **Directorio:** `/bin` (o `/usr/bin`).
- **Comportamiento:** Si se intenta ejecutar `/usr/sbin/nologin`, imprime un mensaje advirtiendo de que la cuenta no está disponible y cierra la sesión inmediatamente retornando un código de salida no nulo para impedir conexiones interactivas.

---

### 92. ¿Qué contraseña (campo pw en /etc/shadow) tiene el usuario "root"?
**Respuesta:**
```bash
sudo grep '^root:' /etc/shadow
```
Aparece habitualmente un asterisco (`*`) o un signo de exclamación (`!`). Indica que la cuenta está bloqueada para login directo por contraseña.

---

### 93. ¿Qué contraseña tiene el usuario "bin"?
**Respuesta:**
```bash
sudo grep '^bin:' /etc/shadow
```
Aparece un asterisco (`*`) o exclamación (`!`), indicando que es una cuenta de sistema sin clave directa.

---

### 94. De los pasos necesarios para cambiar la contraseña del usuario "root". ¿Ha cambiado algo en el campo de contraseña de ese usuario?
**Respuesta:**
```bash
sudo passwd root
```
Sí; el campo de contraseña de `root` en `/etc/shadow` pasa de tener `*` o `!` a contener el hash criptográfico del algoritmo (ej. `$6$...` para SHA-512 o `$y$...` para yescrypt).

---

### 95. Pruebe a "promocionar" su usuario normal (con el que ha iniciado sesión) al usuario "root" usando `su -`. ¿funciona? ¿Qué contraseña ha usado?
**Respuesta:**
```bash
su -
```
Ahora **sí funciona**. Se introduce la nueva contraseña establecida para `root` en la tarea 94.

---

### 96. Compruebe que una vez que es superusuario, puede iniciar sesión como cualquier otro usuario del sistema sin necesidad de conocer la contraseña de ese usuario. Por ejemplo, pruebe a teclear `su - usuarionormal` (donde usuarionormal es el nombre de usuario con el que normalmente inicia sesión).
**Respuesta:**
```bash
su - usuario
id
```
El cambio se produce inmediatamente sin solicitar ninguna contraseña. La orden `id` confirma que el UID y GID han cambiado a los del usuario normal.

---

### 97. Repita el apartado anterior pero ahora inténtelo con el usuario "bin". ¿Funciona ahora? ¿Por qué no funciona?
**Respuesta:**
```bash
su - bin
```
No abre ninguna sesión interactiva y expulsa al usuario devolviendo el prompt. No funciona porque su shell está configurada como `/usr/sbin/nologin`.

---

### 98. Desde una sesión como usuarionormal, ejecute "sudo" para ejecutar la orden "id" como usuario "bin" (use la página de manual si es necesario) ¿Es posible?
**Respuesta:**
```bash
sudo -u bin id
```
**Sí es posible**. La salida confirma `uid=2(bin) gid=2(bin) grupos=2(bin)`. `sudo` con el parámetro `-u` ejecuta mandatos con la identidad de cualquier cuenta sin requerir que esta disponga de shell interactiva.

---

### 99. Indique (consultando los ficheros de contraseñas y el fichero de grupos) a qué grupo principal y secundarios pertenecen el usuario "bin" y el usuario normal del sistema.
**Respuesta:**
```bash
grep '^bin:' /etc/passwd
grep ':bin' /etc/group
grep '^usuario:' /etc/passwd
grep ':.*usuario' /etc/group
```
- **Usuario `bin`:** Grupo principal `bin` (GID 2); habitualmente pertenece a grupos secundarios como `daemon`.
- **Usuario normal:** Grupo principal con su propio nombre (ej. GID 1000); grupos secundarios: `adm`, `sudo`, `plugdev`, etc.

---

### 100. Abra una ventana de terminal como usuario normal y compruebe, mediante el programa "id", a qué grupos pertenece ese usuario. Promocione a superusuario y compruebe ahora a qué grupos pertenece.
**Respuesta:**
```bash
id
sudo -s
id
```
- Como usuario normal: Muestra `uid=1000(usuario) gid=1000(usuario) groups=1000(usuario),4(adm),27(sudo)...`
- Como `root`: Muestra `uid=0(root) gid=0(root) groups=0(root)`.

---

### 101. Utilice "adduser" para crear un nuevo usuario llamado "tarea101", hecho esto compruebe: UID asociado al nuevo usuario; Grupo/s a los que pertenece (indique GID y nombre del grupo); ¿Cuál es el directorio HOME del nuevo usuario? ¿Qué ficheros contiene? ¿Qué permisos tiene asignados?
**Respuesta:**
```bash
sudo adduser tarea101
grep '^tarea101:' /etc/passwd
id tarea101
ls -la /home/tarea101
```
- **UID:** Siguiente disponible para usuario regular (ej. 1001).
- **Grupos:** Grupo principal `tarea101` (GID homónimo, ej. 1001). Sin grupos secundarios iniciales.
- **HOME:** `/home/tarea101`.
- **Ficheros:** Contiene los ficheros copiados de `/etc/skel` (`.bashrc`, `.profile`, `.bash_logout`).
- **Permisos del HOME:** `drwxr-x---` (750) o `drwxr-xr-x` (755).

---

### 102. ¿Qué cambios se producen en el fichero /etc/shadow si se bloquea (mediante "passwd") la cuenta del usuario creado en la tarea anterior. ¿puede ahora iniciar sesión como ese usuario (inténtelo en tty3, por ejemplo)? ¿Puede el superusuario iniciar sesión como si fuera ese usuario? Desbloquee (mediante "passwd") la cuenta y compruebe qué cambios se produce en el fichero /etc/shadow. Compruebe que ahora puede iniciar sesión (ej. En tty3) como ese usuario usando la contraseña que definió cuando lo creó.
**Respuesta:**
1. Bloqueo:
   ```bash
   sudo passwd -l tarea101
   sudo grep '^tarea101:' /etc/shadow
   ```
   Se añade una exclamación `!` al principio del hash de la contraseña.
2. En `tty3`: Rechaza el acceso (no se puede iniciar sesión con contraseña).
3. Superusuario: `root` **sí puede** acceder ejecutando `su - tarea101`.
4. Desbloqueo:
   ```bash
   sudo passwd -u tarea101
   sudo grep '^tarea101:' /etc/shadow
   ```
   Se elimina el signo `!`. El acceso con su clave original vuelve a funcionar en `tty3`.

---

### 103. Cree un script llamado /bin/mishell que ponga un mensaje "Bienvenido!" y establezca ese script como shell para el nuevo usuario. ¿Qué ocurre ahora si intenta iniciar sesión (pruebe a usar uno de los tty) como ese usuario? ¿Puede iniciar sesión como ese usuario desde una shell de superusuario (usando su)? Deje la shell del nuevo usuario establecida al valor inicial (/bin/bash) y compruebe que todo vuelve a funcionar correctamente.
**Respuesta:**
1. Crear el script:
   ```bash
   sudo tee /bin/mishell << 'EOF'
   #!/bin/bash
   echo "Bienvenido!"
   EOF
   sudo chmod 755 /bin/mishell
   sudo chsh -s /bin/mishell tarea101
   ```
2. Al iniciar sesión en `tty`: Se imprime `"Bienvenido!"` y la sesión se cierra instantáneamente porque el script finaliza.
3. Desde superusuario: Con `su - tarea101` ocurre lo mismo (imprime el mensaje y retorna al prompt de root).
4. Restauración:
   ```bash
   sudo chsh -s /bin/bash tarea101
   ```

---

### 104. Cree un nuevo grupo denominado "profes" y haga que el usuario "tarea101" pertenezca también a ese grupo (grupo secundario).
**Respuesta:**
```bash
sudo addgroup profes
sudo adduser tarea101 profes
# O bien:
sudo usermod -a -G profes tarea101
```

---

### 105. Cree (usando adduser) un usuario denominado "usu1" que pertenezca (como grupo principal) al grupo "profes" recién creado y que el directorio HOME sea /home/profes/usu1 (nota: tenga en cuenta que si el directorio /home/profes no existe tendrá que crearlo manualmente antes de añadir el usuario).
**Respuesta:**
```bash
sudo mkdir -p /home/profes
sudo adduser --ingroup profes --home /home/profes/usu1 usu1
```

---

### 106. Abra dos terminales nuevos. Use uno de ellos para añadir al grupo profes al usuario normal del sistema. Una vez hecho ese paso, compruebe si en el otro terminal abierto el usuario pertenece, efectivamente, al nuevo grupo creado. Abra una nueva ventana de terminal y compruebe si ahora funciona.
**Respuesta:**
1. En Terminal 1:
   ```bash
   sudo adduser $USER profes
   ```
2. En Terminal 2: Al ejecutar `id`, el nuevo grupo **no aparece**, porque los tokens de seguridad y grupos de un proceso se fijan en el momento de crear la sesión.
3. En una nueva ventana: Al ejecutar `id`, el grupo `profes` **sí aparece**, ya que lee la información de grupos actualizada al iniciar el nuevo proceso shell.

---

### 107. Una vez que se ha completado correctamente la tarea 106 (compruebe que pertenece al grupo "profes") cree un fichero vacío en su directorio de usuario. ¿A qué grupo pertenece? Teclee ahora `newgrp profes` e intente crear un nuevo fichero vacío. ¿A qué grupo pertenece el nuevo fichero?
**Respuesta:**
```bash
touch fich1.txt
ls -l fich1.txt       # Pertenece al grupo principal habitual del usuario
newgrp profes
touch fich2.txt
ls -l fich2.txt       # Pertenece al grupo "profes"
```
`newgrp` inicia una nueva subshell donde el grupo activo por defecto para la creación de ficheros pasa a ser `profes`.

---

### 108. Muévase al directorio /dev y compruebe si hay ficheros cuyo nombre empiece por sda. ¿de qué tipo son esos ficheros? ¿Qué permisos tienen esos ficheros?
**Respuesta:**
```bash
cd /dev
ls -l sda*
```
- **Tipo:** Ficheros especiales de **bloque** (la primera letra de permisos es una **`b`**, ej. `brw-rw----`).
- **Permisos:** Normalmente `660` (`brw-rw----`), con propietario `root` y grupo `disk`.

---

### 109. Compruebe si existe, en el directorio /dev, un fichero denominado ttyS0. ¿de qué tipo es? ¿qué permisos tiene?
**Respuesta:**
```bash
ls -l /dev/ttyS0
```
- **Tipo:** Fichero especial de **carácter** (la primera letra de permisos es una **`c`**, ej. `crw-rw----`).
- **Permisos:** Habitualmente `660` (`crw-rw----`), usuario `root` y grupo `dialout`.

---

### 110. ¿Qué tipo de dispositivo y qué permisos tienen los ficheros /dev/null y /dev/zero?
**Respuesta:**
```bash
ls -l /dev/null /dev/zero
```
- **Tipo:** Dispositivos de **carácter** (`c`).
- **Permisos:** `crw-rw-rw-` (permisos 666, acceso universal de lectura y escritura para todos los usuarios).

---

### 111. Mantenga dos ventanas de terminal abiertas (usuario normal) y teclee, en una de ellas, ps. Anote el dispositivo obtenido. Desde el otro terminal compruebe si existe ese dispositivo (`/dev/pts/x`), de qué tipo es y qué permisos tiene. Desde ese otro terminal, pruebe a teclear `echo "HOLA" > /dev/pts/x`.
**Respuesta:**
1. En Terminal 1: `ps` muestra en la columna TTY, por ejemplo, `pts/1`.
2. En Terminal 2:
   ```bash
   ls -l /dev/pts/1
   ```
   Es un dispositivo de caracteres (`c`), permisos `crw--w----`, con propietario el propio usuario.
3. En Terminal 2 ejecutar:
   ```bash
   echo "HOLA" > /dev/pts/1
   ```
   La palabra `"HOLA"` aparece impresa de inmediato en la pantalla de la Terminal 1.

---

### 112. Cree, como superusuario, un directorio llamado /tmp/DIR1 y dentro de ese directorio un fichero llamado fichero.txt que contiene el texto "Hola". Asigne permisos rwx------ al directorio DIR1 y añada permiso de lectura y escritura para "others" en el fichero recién creado. Desde otro terminal, como usuario normal, compruebe si puede editar el fichero. ¿Es posible? ¿por qué no es posible?
**Respuesta:**
```bash
sudo mkdir /tmp/DIR1
echo "Hola" | sudo tee /tmp/DIR1/fichero.txt
sudo chmod 700 /tmp/DIR1
sudo chmod 606 /tmp/DIR1/fichero.txt
```
Como usuario normal:
```bash
cat /tmp/DIR1/fichero.txt
```
**No es posible** (error `Permiso denegado`). Para acceder a cualquier fichero dentro de una ruta, el usuario debe tener obligatoriamente permiso de ejecución (`x`, permiso de paso) en todos los directorios que forman el camino jerárquico. Al tener `/tmp/DIR1` permisos `700`, ningún otro usuario puede atravesarlo.

---

### 113. Añada ahora permiso de lectura (r) a "others" para DIR1 e intente, como usuario normal, ver el contenido de ese directorio usando la salida "larga" de "ls". ¿Funciona? ¿Qué permiso necesita para visualizar correctamente el contenido de DIR1?
**Respuesta:**
```bash
sudo chmod o+r /tmp/DIR1
ls -l /tmp/DIR1
```
**No funciona correctamente**. Se leen los nombres de los ficheros contenidos pero los atributos, tamaños, fechas e inodos aparecen como interrogaciones (`?`). Para obtener los atributos en un listado largo se requiere permiso de ejecución (`x`) para poder hacer la llamada `stat()` sobre las entradas.

---

### 114. Asigne ahora el permiso de lectura y ejecución (r-x) a "others" e intente, como usuario normal, ver el contenido de ese directorio usando la salida "larga" de "ls". ¿Funciona?
**Respuesta:**
```bash
sudo chmod o+rx /tmp/DIR1
ls -l /tmp/DIR1
```
**Sí funciona** perfectamente. Con `r` se lee la tabla del directorio y con `x` se atraviesa para consultar los metadatos de los ficheros.

---

### 115. Asigne ahora el permiso de ejecución (--x) a "others" e intente, como usuario normal, ver el contenido de ese directorio usando la salida "larga" de "ls". ¿Funciona? Intente ver el contenido de /tmp/DIR1/fichero.txt ¿funciona? Intente ahora editar fichero.txt y añada una línea que ponga "Se puede editar". ¿Funciona? ¿Cómo se explica eso si no tiene permiso de escritura en el directorio?
**Respuesta:**
```bash
sudo chmod o=x /tmp/DIR1
ls -l /tmp/DIR1             # Da error: Permiso denegado (falta 'r' en el directorio)
cat /tmp/DIR1/fichero.txt   # Funciona (se puede atravesar el directorio y leer el fichero)
echo "Se puede editar" >> /tmp/DIR1/fichero.txt # Funciona
```
Se explica porque para modificar el contenido de un fichero solo se requiere permiso de escritura (`w`) sobre el propio fichero y permiso de paso (`x`) en los directorios del path. La escritura en el directorio solo es necesaria si se crean o borran entradas.

---

### 116. Hemos comprobado que puede editar el fichero pero, ¿puede borrarlo? ¿por qué? ¿qué permisos necesitaría asignar y dónde para que el usuario normal pueda borrar fichero.txt? De los pasos necesarios para que sea posible.
**Respuesta:**
- Como usuario normal: `rm /tmp/DIR1/fichero.txt` da error `Permiso denegado`.
- **Motivo:** Borrar un archivo modifica la estructura interna del directorio contenedor (elimina su entrada), por lo que se exige permiso de escritura (`w`) sobre el directorio `/tmp/DIR1`.
- **Solución:**
  ```bash
  sudo chmod o+w /tmp/DIR1
  ```

---

### 117. Abra un nuevo terminal como usuario normal y cree un fichero vacío en su directorio de usuario. ¿Qué permisos se han asignado a ese fichero? ¿qué permisos (en formato numérico) tendría que asignar para que solo el propietario pueda acceder y modificar ese fichero? ¿diría lo mismo si se tratara de un directorio?
**Respuesta:**
```bash
touch nuevo.txt
ls -l nuevo.txt
```
- Permisos asignados por defecto: `664` (`-rw-rw-r--`) o `644` (`-rw-r--r--`).
- Para acceso exclusivo de lectura/escritura del propietario en un fichero: **`600`** (`chmod 600 nuevo.txt`).
- En un directorio **no es lo mismo**: debe añadirse el permiso de paso/ejecución (`x`), siendo el valor numérico **`700`** (`chmod 700 nuevodir`).

---

### 118. ¿De qué forma puede establecerse que todos los ficheros y directorios que se creen tengan, por defecto, permiso de acceso y modificación solo para el propietario? Compruebelo. ¿Qué pasos tendría que dar para que el cambio que acaba de realizar sea persistente?
**Respuesta:**
Estableciendo la máscara de creación en `077`:
```bash
umask 077
touch f_privado.txt && ls -l f_privado.txt   # Permisos: -rw------- (600)
mkdir d_privado && ls -ld d_privado          # Permisos: drwx------ (700)
```
Para hacerlo persistente, se añade al final de `~/.bashrc`:
```bash
echo "umask 077" >> ~/.bashrc
```

---

### 119. Cree un fichero con cualquier contenido en su directorio de usuario y use "ln" para crear un enlace simbólico que apunte a ese nuevo fichero. Compruebe con "ls" los permisos que se han establecido al fichero de enlace. ¿Qué permisos tiene? ¿Hay algo en la primera columna de la salida extendida que indique el que ese fichero es un enlace? ¿Qué tamaño tiene el enlace?
**Respuesta:**
```bash
echo "Contenido base" > orig.txt
ln -s orig.txt enlace.sym
ls -l enlace.sym
```
- **Permisos:** `lrwxrwxrwx` (los permisos del enlace simbólico son siempre virtuales y no se evalúan; se aplican los del fichero destino).
- **Primera columna:** Comienza por la letra **`l`**.
- **Tamaño:** La longitud exacta en caracteres del nombre o ruta del fichero destino (para `orig.txt`, son 8 bytes).

---

### 120. Pruebe a editar el fichero de enlace y compruebe que está editando realmente el fichero a donde apunta el enlace. Intente modificar los permisos del enlace y compruebe si realmente se cambian los permisos del fichero enlace o si, por el contrario, se cambian los permisos del fichero al que apunta el enlace.
**Respuesta:**
```bash
echo "Nueva linea" >> enlace.sym
cat orig.txt                       # Se comprueba que orig.txt tiene la modificación
chmod 600 enlace.sym
ls -l orig.txt                     # orig.txt ha cambiado a -rw-------
ls -l enlace.sym                   # Sigue siendo lrwxrwxrwx
```
Cualquier modificación de permisos con `chmod` se propaga al fichero referenciado.

---

### 121. Borre el fichero real (no el enlace) e intente ahora editar el fichero de enlace. ¿Qué ocurre? (Termine borrando el enlace).
**Respuesta:**
```bash
rm orig.txt
cat enlace.sym
```
El enlace simbólico queda roto (*dangling link*). `cat` indica `No existe el archivo o el directorio`.  
Borrar enlace:
```bash
rm enlace.sym
```

---

### 122. También es posible crear enlaces simbólicos que apunten a directorios. Pruebe a crear en su directorio de usuario un fichero de enlace llamado "root" que apunta al directorio "/". ¿Qué permisos tiene? ¿Hay algo en la primera columna de la salida extendida que indique el que ese fichero es un enlace? ¿Qué tamaño tiene el enlace?
**Respuesta:**
```bash
ln -s / root_sym
ls -ld root_sym
```
- **Permisos:** `lrwxrwxrwx`.
- **Primera columna:** Comienza con `l`.
- **Tamaño:** 1 byte (la cadena `"/"` mide un carácter).

---

### 123. Compruebe, mediante "cd" que es posible cambiar al directorio "root" y que realmente le redirige a "/". Compruebe ahora cuál es el directorio actual de trabajo, ¿es "/" o es otra cosa? Compruebe ahora qué ocurre si cambia al directorio "padre". ¿A dónde le lleva? ¿Conclusiones?
**Respuesta:**
```bash
cd root_sym
pwd            # Muestra el path lógico: /home/usuario/root_sym
pwd -P         # Muestra el path físico real: /
cd ..
pwd            # Devuelve a /home/usuario
```
**Conclusión:** Bash mantiene de forma lógica la ruta a través del enlace simbólico en variables internas (`$PWD`), permitiendo que `cd ..` regrese al directorio donde residía el enlace, aunque físicamente estemos en la raíz.

---

### 124. Cree un fichero llamado "tarea124" y añada en el algún contenido. Compruebe los permisos del fichero y el valor de la segunda columna (en el listado extendido de ls). Observará que el valor de la segunda columna es un "1" lo que indica que ese fichero está referenciado una sola vez.
**Respuesta:**
```bash
echo "Datos de prueba" > tarea124
ls -l tarea124
```
La columna 2 muestra un `1`, indicando que existe una única entrada de directorio apuntando a ese inodo.

---

### 125. Cree ahora un enlace duro llamado "tarea125" que apunte al fichero creado en la tarea anterior. Compruebe ahora los permisos y tamaño de ambos ficheros. ¿Qué indica ahora la segunda columna de la salida de "ls"? (observará que ahora se indica, para los dos ficheros, el número 2 indicando que el fichero está referenciado en dos sitios. NOTA: Eso no sucede con los enlaces simbólicos).
**Respuesta:**
```bash
ln tarea124 tarea125
ls -l tarea124 tarea125
```
Ambos ficheros muestran idénticos permisos, propietario, tamaño, fecha y un **`2`** en la segunda columna. Con `ls -i` se constata que comparten exactamente el mismo número de inodo.

---

### 126. Compruebe que si edita "tarea125" y modifica el contenido, también se ha modificado el contenido del otro fichero.
**Respuesta:**
```bash
echo "Mas datos" >> tarea125
cat tarea124
```
El contenido añadido aparece en `tarea124` porque ambos nombres apuntan a los mismos bloques de datos en el sistema de ficheros.

---

### 127. Modifique los permisos del fichero "tarea125" y observe si se han modificado también los permisos de tarea124.
**Respuesta:**
```bash
chmod 600 tarea125
ls -l tarea124
```
Los permisos de `tarea124` también han cambiado a `-rw-------`. Al modificar permisos se altera la estructura del inodo común.

---

### 128. Pruebe a borrar el fichero "tarea124" y compruebe si su contenido sigue estando accesible desde el otro fichero. Compruebe el valor de la segunda columna de la salida extendida de "ls" para ese otro fichero.
**Respuesta:**
```bash
rm tarea124
cat tarea125
ls -l tarea125
```
El contenido sigue plenamente accesible. La segunda columna de `tarea125` pasa a valer **`1`**. El fichero no se destruye físicamente mientras su contador de enlaces sea mayor que 0.

---

### 129. ¿Qué ocurriría si activa el bit setuid al editor "nano"? Hágalo y compruebe que la respuesta a la pregunta anterior es correcta. ¿Qué ocurre si le quita permiso de ejecución a "otros"? ¿Quiénes podrían ejecutar el editor con privilegio de superusuario en este último caso? Nota: Cuando termine esta tarea asegúrese de que ha dejado los permisos del editor "nano" a los originales.
**Respuesta:**
```bash
sudo chmod u+s /bin/nano
```
Cualquier usuario que ejecute `nano` adquiere UID efectivo 0 (`root`), pudiendo editar y sobreescribir cualquier fichero del sistema (`/etc/shadow`, etc.).
- Si se le quita ejecución a otros (`sudo chmod o-x /bin/nano`):
  Solo el propietario (`root`) y los miembros del **grupo** asignado al binario podrán ejecutarlo con privilegios de root.
- **Restauración:**
  ```bash
  sudo chmod 755 /bin/nano
  ```

---

### 130. Cree, como usuario normal, un directorio llamado DIR2 en /tmp. ¿de qué grupo es ese directorio? ¿se corresponde con un grupo principal o secundario del usuario? Modifique el grupo de ese directorio para que pertenezca a uno de los grupos secundarios de los que es miembro el usuario normal. Si crea un fichero dentro de /tmp/DIR2 ¿a qué grupo pertenece? Pruebe a activar en /tmp/DIR2 el bit "setgid" y, a continuación, cree un nuevo fichero dentro de /tmp/DIR2, ¿de qué grupo es ahora el fichero?
**Respuesta:**
```bash
mkdir /tmp/DIR2
ls -ld /tmp/DIR2               # Pertenece al grupo principal del usuario
chgrp profes /tmp/DIR2         # Cambiado a un grupo secundario
touch /tmp/DIR2/fich_sin_sgid
ls -l /tmp/DIR2/fich_sin_sgid # Pertenece al grupo principal del usuario
chmod g+s /tmp/DIR2            # Activación de SETGID en el directorio
touch /tmp/DIR2/fich_con_sgid
ls -l /tmp/DIR2/fich_con_sgid # Pertenece automáticamente al grupo "profes"
```

---

### 131. En esta tarea será necesario que exista, además del superusuario y el usuario practicas, un tercer usuario (da igual el nombre) que deberá crear si no existe. Cree, como superusuario, un directorio /tmp2 y asígnele permisos 777. Intente crear un fichero "mifich" en ese directorio como usuario practicas y asígnele permisos para que solo pueda acceder al él (rw) el propio usuario practicas. Compruebe ahora si el tercer usuario puede o no borrar ese fichero.
**Respuesta:**
```bash
sudo mkdir /tmp2
sudo chmod 777 /tmp2
# Como usuario practicas:
touch /tmp2/mifich
chmod 600 /tmp2/mifich
# Como tercer usuario:
rm /tmp2/mifich
```
El tercer usuario **sí puede borrarlo**, debido a que el directorio `/tmp2` tiene permisos `777` (permiso de escritura `w` para todos en el directorio).

---

### 132. Repita la tarea anterior pero añadiendo el permiso "sticky-bit" al directorio /tmp2.
**Respuesta:**
```bash
sudo chmod +t /tmp2
# Comprobación de permisos: drwxrwxrwt
```
Ahora el tercer usuario intenta `rm /tmp2/mifich` y el sistema responde: **`rm: cannot remove 'mifich': Operation not permitted`**. El sticky bit impide que un usuario borre o renombre ficheros que no sean de su propiedad en un directorio compartido.

---

### 133. Cree (usando adduser) un usuario denominado "usu2" que pertenezca (como grupo principal) al grupo "alum" (debe crearlo, si no existe) y que el directorio HOME sea /home/alum/usu2 (nota: tenga en cuenta que si el directorio /home/alum no existe tendrá que crearlo manualmente antes de añadir el usuario).
**Respuesta:**
```bash
sudo addgroup alum
sudo mkdir -p /home/alum
sudo adduser --ingroup alum --home /home/alum/usu2 usu2
```

---

### 134. Observará que ahora mismo tiene creados dos usuarios usu1 (creado en tarea105) y usu2 que pertenecen a dos grupos distintos (grupo principal) y que cuyos directorios de usuario están en dos ramas distintas (/home/profes y /home/alum). Esta clasificación permite, por ejemplo, establecer permisos de grupo en /home/profes y /home/alum de forma que los miembros de los grupos profes y alum tengan acceso a esos directorios y que el resto (others) no tenga acceso. Implemente esta característica.
**Respuesta:**
```bash
sudo chown root:profes /home/profes
sudo chmod 750 /home/profes

sudo chown root:alum /home/alum
sudo chmod 750 /home/alum
```
Con `750` (`rwxr-x---`), el usuario root tiene control total, los miembros del grupo respectivo (`profes` o `alum`) tienen permisos de lectura y paso, y `others` (`---`) tiene el acceso completamente bloqueado.

---

# TEMA 4: Procesos

### 135. Teclee "ps" y observe cuántos procesos se muestran, ¿qué información se muestra de cada proceso?
**Respuesta:**
```bash
ps
```
Muestra los procesos que comparten el mismo EUID del usuario y están asociados a la terminal actual (normalmente dos: la propia shell `bash` y el ejecutable `ps`).  
Columnas mostradas:
- `PID`: Identificador único del proceso.
- `TTY`: Terminal asociada al proceso.
- `TIME`: Tiempo acumulado de ejecución de CPU.
- `CMD`: Nombre de la orden o binario ejecutado.

---

### 136. "ps" sin parámetros muestra poca información sobre los procesos. Hay dos formas de obtener más información sobre los mismos, una de ellas es `ps -f` y la otra `ps u`. Anote qué información se muestra en cada una de las formas propuestas. Hay alguna relación paterno-filial entre los procesos que se muestran (indique cómo lo ha averiguado y por qué cree que ocurre esto).
**Respuesta:**
- `ps -f` (formato UNIX completo): Muestra `UID`, `PID`, `PPID` (PID del proceso padre), `C` (uso de CPU), `STIME` (hora de inicio), `TTY`, `TIME` y `CMD`.
- `ps u` (formato BSD orientado a usuario): Muestra `USER`, `PID`, `%CPU`, `%MEM`, `VSZ` (memoria virtual), `RSS` (memoria residente física), `TTY`, `STAT` (estado del proceso), `START`, `TIME` y `COMMAND`.
- **Relación paterno-filial:** Se comprueba que el `PPID` del proceso `ps` coincide de forma idéntica con el `PID` de la shell `bash`. Ocurre porque la shell es el intérprete padre que ejecuta la llamada al sistema `fork()` y `exec()` para lanzar el comando tecleado.

---

### 137. Indique qué programa se está ejecutando con PID=1 y el PID y PPID del proceso que ejecuta "cron".
**Respuesta:**
```bash
ps -ef | grep -E '^UID|[s]ystemd|[i]nit|[c]ron'
```
- **PID=1:** `/sbin/init` (enlace simbólico a `systemd`).
- **cron:** Se observa su línea en la salida de `ps -ef` para anotar su `PID` específico y su `PPID` (cuyo padre suele ser `1`).

---

### 138. Mostrar solo los procesos del usuario "syslog" (o cualquier otro usuario, si no hay ninguno).
**Respuesta:**
```bash
ps -u syslog
# O bien:
ps aux | grep '^syslog'
```

---

### 139. ¿De qué forma podemos mostrar los procesos del apartado anterior pero que, además, se muestre la cabecera? Pista: Usar "grep" con dos patrones.
**Respuesta:**
```bash
ps -ef | grep -E '^UID|syslog'
```

---

### 140. Establezca la variable LANG a valor C. Utilice el programa "top" para mostrar en tiempo real los procesos del sistema.
**Respuesta:**
```bash
export LANG=C
top
```
Se pulsa `h` para abrir la ayuda con todas las teclas interactivas de control.

---

### 141. En esta tarea vamos a practicar con el manejo del programa "top", use la ayuda para saber cómo hacer estas tareas:
**Respuesta:**
- **Ordenar por PID ascendente:** Pulsar `f`, desplazarse hasta `PID`, pulsar `s` (seleccionar como clave de ordenación), `R` (para conmutar orden ascendente/descendente) y `q` (volver a la vista principal).
- **Ordenar por tiempo de CPU (TIME+):** Pulsar `f`, seleccionar el campo `TIME+`, presionar `s` y pulsar `q`.
- **Intervalo de actualización a 0 segundos:** Pulsar `d` (o `s`), teclear `0` y pulsar Enter.
- **Intervalo de actualización a 5 segundos:** Pulsar `d`, teclear `5` y pulsar Enter.
- **Mostrar solo procesos de un usuario:** Pulsar `u`, escribir el nombre de usuario y pulsar Enter.
- **Mostrar de nuevo todos los usuarios:** Pulsar `u`, dejar el campo en blanco y pulsar Enter.
- **Guardar la configuración:** Pulsar `W` (Shift + w). Muestra que se ha escrito en `~/.toprc` (o `~/.config/procps/toprc`). Para resetearlo:
  ```bash
  rm ~/.toprc 2>/dev/null || rm ~/.config/procps/toprc 2>/dev/null
  ```

---

### 142. Realice un script (al que llamaremos "carga") que consista en un bucle infinito en el que se esté asignando b=10 continuamente. Ejecute una instancia de ese script y observe en "top" el %CPU. ¿Qué ocurre si se lanzan dos instancias? ¿Y con tres? Extraiga conclusiones respecto al número de CPU/core que tiene el sistema.
**Respuesta:**
1. Crear el script `carga`:
   ```bash
   cat << 'EOF' > carga
   #!/bin/bash
   while true; do
       b=10
   done
   EOF
   chmod +x carga
   ```
2. Lanzar instancias:
   ```bash
   ./carga &
   ./carga &
   ./carga &
   ```
- Cada proceso satura un núcleo al 100% de CPU.
- Si el equipo tiene 2 núcleos, dos instancias consumirán el 100% de cada uno (200% global en `top`); la tercera instancia obligará al planificador a repartir tiempo entre ellas (66% cada una). Si tiene 4 o más núcleos, cada una consumirá el 100% de un core sin interferencias.

---

### 143. Desactive todas las CPU/core excepto la cpu0 mediante `echo "0" > /sys/devices/system/cpu/cpuXX/online`.
**Respuesta:**
```bash
sudo -s
for cpu in /sys/devices/system/cpu/cpu[1-9]*/online; do
    echo "0" > "$cpu"
done
```

---

### 144. Termine la ejecución de los scripts anteriores. Arranque el script "carga" con el comando interno "time": `time ./carga`. En "top" compruebe %us y %sys. Calcule con "time" el reparto de CPU y compruebe los tiempos.
**Respuesta:**
- En `top`: `%us` (espacio de usuario) se sitúa cercano al **100%**, y `%sys` próximo a **0%**, ya que la asignación `b=10` se ejecuta estrictamente en la CPU sin llamadas al sistema.
- Al abortar con `Ctrl+C`:
  `real`: tiempo total de reloj transcurrido.  
  `user`: prácticamente idéntico al tiempo `real`.  
  `sys`: residual (~0.00s).  
  Se cumple que $t_{\text{real}} \approx t_{\text{user}} + t_{\text{sys}}$.

---

### 145. Pruebe a teclear `time dd if=/dev/zero of=/dev/null`. ¿Cree que se generará algún tipo de actividad de entrada/salida? Diferencias con la tarea anterior en "top".
**Respuesta:**
```bash
time dd if=/dev/zero of=/dev/null
```
- No genera E/S física a disco (todo opera en buffers de memoria virtual del kernel).
- En `top`: El uso pasa a estar casi al **100% en `%sys`** y el `%us` se sitúa cercano al 0%, dado que el proceso pasa todo su ciclo ejecutando llamadas al sistema (`read` sobre `/dev/zero` y `write` sobre `/dev/null`). Al cancelar, el tiempo `real` equivale prácticamente al tiempo `sys`.

---

### 146. ¿Qué cree que hace `time dd if=/dev/sda of=/dev/null`? ¿cree que generará actividad de entrada/salida? Compruebe empíricamente el valor de %us, %sys y %wa. Calcule tiempos y justifique si el tiempo real es superior a la suma de usuario y sistema.
**Respuesta:**
```bash
sudo time dd if=/dev/sda of=/dev/null bs=8k
```
- Lee los bloques físicos del disco duro `/dev/sda`. Sí genera intensa actividad de E/S.
- En `top`: El porcentaje `%wa` (*iowait*) sube de manera notable.
- En la salida de `time`: El tiempo `real` es **claramente muy superior** a la suma de $t_{\text{user}} + t_{\text{sys}}$. La diferencia corresponde al tiempo en que el proceso estuvo bloqueado en estado `D` (espera no interrumpible de hardware), durante el cual la CPU queda desocupada esperando a que el controlador de disco transfiera los datos.

---

### 147. Terminal 1: top. Terminal 2: `dd if=/dev/sda of=/dev/null`. Terminal 3: `./carga`. ¿Qué valor tiene ahora %wa? ¿Cómo se explica eso si el proceso "dd" sigue activo?
**Respuesta:**
- `%wa` cae prácticamente a **0%**, y `%us` se eleva al 100%.
- **Explicación:** La métrica `iowait` solo computa el tiempo en que la CPU está desocupada (*idle*) mientras existen procesos bloqueados en E/S. Al introducir el script `carga`, la CPU encuentra tareas listas para ejecutar en espacio de usuario, por lo que la CPU se encuentra al 100% ocupada y el contador de espera ociosa desaparece.

---

### 148. Arranque `firefox`. Pulse `Ctrl-C` en el terminal. ¿Qué señal se le envía? ¿Continúa "vivo"?
**Respuesta:**
- Se envía la señal **`SIGINT`** (señal 2).
- El proceso **no continúa vivo**; finaliza de forma inmediata al interceptar la interrupción de terminal.

---

### 149. Arranque `firefox`. Pulse `Ctrl-Z`. ¿Qué señal se envía? ¿Continúa "vivo"? Use "pkill" para terminarlo.
**Respuesta:**
- Se envía la señal **`SIGTSTP`** (señal 20).
- El proceso **sigue vivo en la tabla de procesos**, pero queda en estado detenido/congelado (`T`). La ventana gráfica deja de responder.
- Para terminarlo:
  ```bash
  pkill -f firefox
  ```

---

### 150. Arranque de nuevo el navegador y cierre la ventana de terminal pulsando la "X" gráfica. ¿Continúa ejecutándose Firefox?
**Respuesta:**
**No continúa ejecutándose**. Al cerrar la ventana, el emulador de terminal se destruye y envía la señal de colgado **`SIGHUP`** (señal 1) a todos los procesos dependientes del grupo de la terminal, lo que provoca la terminación de Firefox.

---

### 151. Usando ps, identifique el pts asociado a una ventana de terminal. Vuelva a arrancar "Firefox". En otra ventana de terminal compruebe el PID del proceso Firefox y si el PPID se corresponde con el de la shell del terminal desde donde abrió la aplicación.
**Respuesta:**
```bash
ps -ef | grep -i [f]irefox
```
Se observa que el `PPID` de Firefox coincide exactamente con el `PID` de la shell `bash` abierta en el terminal donde se invocó, compartiendo su mismo pseudo-terminal (`pts/X`).

---

### 152. Pulse Ctrl-Z sobre Firefox. Teclee `bg`. Compruebe con `jobs`. Salga con `exit`. ¿Continúa vivo? ¿Cuál es su PPID ahora?
**Respuesta:**
```bash
bg
jobs
exit
```
- Sí, continúa vivo.
- Su nuevo `PPID` pasa a ser **`1`** (o el PID del gestor de sesión `systemd --user`), al haber quedado huérfano tras la finalización de su shell padre.

---

### 153. Arranque en segundo plano `dd if=/dev/zero of=/dev/null &`. Compruebe con "jobs". Termínelo mediante `kill %n`.
**Respuesta:**
```bash
dd if=/dev/zero of=/dev/null &
jobs
kill %1
```

---

### 154. Arranque de nuevo el proceso "dd" en segundo plano. Compruebe PID, PPID y el directorio `/proc/<PID>/fd`. Inspeccione los descriptores 0, 1 y 2. Salga de la shell con exit y observe qué ocurre con el descriptor 2. Termine el proceso.
**Respuesta:**
```bash
dd if=/dev/zero of=/dev/null &
# Supongamos PID = 4567
ls -l /proc/4567/fd
```
- `0 -> /dev/zero`
- `1 -> /dev/null`
- `2 -> /dev/pts/X`
Al salir con `exit`, el proceso sigue vivo; si se examina de nuevo `/proc/4567/fd/2`, el enlace al pseudo-terminal queda apuntando a un dispositivo cerrado o inexistente (`/dev/pts/X (deleted)`). Para finalizarlo:
```bash
kill 4567
```

---

### 155. Con dos terminales abiertos, identifique el PID de la shell de cada uno y, usando kill, pruebe a terminar desde un terminal la shell del otro. ¿Ha bastado con SIGTERM o se ha tenido que recurrir a SIGKILL?
**Respuesta:**
1. En Terminal 1: `echo $$` (ej. PID 5001).
2. En Terminal 2:
   ```bash
   kill -15 5001   # SIGTERM
   ```
Bash ignora la señal `SIGTERM` por diseño cuando es interactiva para proteger la sesión del usuario.  
Es obligatorio enviar la señal ineludible **`SIGKILL`** (señal 9):
```bash
kill -9 5001
```

---

### 156. Arranque varios procesos "carga" y monitorícelos con "top". Modifique la prioridad con renice/top al máximo (-20) y al mínimo (+19). Termine todos excepto el de +19. ¿Continúa sin recibir CPU? ¿Por qué?
**Respuesta:**
- Con alta concurrencia, el proceso con prioridad `nice +19` recibe una fracción ínfima de CPU frente a los que tienen `nice -20`.
- Al matar el resto de procesos: el proceso con `nice +19` pasa a consumir de inmediato el **100% de la CPU**. Esto ocurre porque la prioridad o valor de *nice* es un mecanismo de reparto relativo: si no hay otros procesos compitiendo por la CPU, el planificador le otorga todos los ciclos disponibles.

---

### 157. Ejecute `dd if=/dev/zero of=/dev/null &` y averigüe su PID. Inspeccione en `/proc/<PID>`: a dónde apunta `cwd`, a dónde apunta `exe`, qué contiene `cmdline`, y a dónde apuntan `fd/0`, `fd/1` y `fd/2`.
**Respuesta:**
```bash
dd if=/dev/zero of=/dev/null &
PID=$!
ls -l /proc/$PID/cwd       # Apunta al directorio de trabajo desde el que se invocó
ls -l /proc/$PID/exe       # Apunta al binario ejecutable: /usr/bin/dd
cat /proc/$PID/cmdline     # Muestra los argumentos pasados: dd\0if=/dev/zero\0of=/dev/null
ls -l /proc/$PID/fd        # 0 -> /dev/zero, 1 -> /dev/null, 2 -> /dev/pts/X
kill $PID
```

---

### 158. Compruebe qué limites tiene establecidos el sistema en su configuración por defecto. Compruebe también si existe un fichero limits.conf a nivel global en el sistema.
**Respuesta:**
```bash
ulimit -a
cat /etc/security/limits.conf
```
`ulimit -a` muestra los límites actuales del proceso (tamaño de core dumps, número de descriptores abiertos con `-n`, tiempo de CPU, procesos concurrentes con `-u`, etc.).

---

### 159. Establezca en una shell un límite de tiempo de ejecución a 1m, arranque en segundo plano un proceso de suma y monitorícelo mediante "top" ¿qué ocurre cuando el tiempo de CPU consumido es de un minuto?
**Respuesta:**
```bash
ulimit -t 60
./carga &
```
Cuando el proceso acumula exactamente 60 segundos de cómputo en la CPU, el núcleo del sistema operativo le envía la señal **`SIGXCPU`** (*CPU time limit exceeded*) y el proceso es abortado automáticamente.

---

### 160. Cree una tarea con "at" para que dentro de 2m se inicie un proceso de "reinicio" del sistema. El proceso debe incluir un aviso, mediante "wall", 30 segundos antes. Compruebe con "atq".
**Respuesta:**
```bash
sudo at now + 2 minutes
```
Dentro del prompt interactivo de `at>` se introduce:
```bash
sleep 90
wall "El sistema se reiniciara en 30 segundos"
sleep 30
reboot
```
Se pulsa `Ctrl+D` para guardar. Para verificar la cola y borrarlo antes de que reinicie:
```bash
atq
atrm <numero_de_job>
```

---

### 161. Teclee en un terminal `bash` y compruebe con `ps` que es hija de la anterior. Teclee `exit` y pruebe con `exec bash`. ¿Qué cambios suceden?
**Respuesta:**
- Al teclear `bash`: Se ejecuta un `fork()`, creando un nuevo proceso cuyo `PPID` es la shell previa (ambas coexisten).
- Al teclear `exec bash`: La llamada al sistema `exec()` reemplaza la imagen de memoria del proceso actual por una nueva instancia de Bash conservando el **mismo PID**. No se crea ningún proceso hijo adicional.

---

### 162. ¿Qué ocurre si en la ventana de un terminal teclea `exec ls -la`? Compruebe que está en lo cierto.
**Respuesta:**
```bash
exec ls -la
```
Muestra el listado de archivos y la ventana de terminal **se cierra inmediatamente**. Ocurre porque `ls` sustituye al proceso de la shell; al finalizar `ls`, el proceso concluye y la terminal finaliza por no tener proceso que controlar.

---

### 163. Programe, como usuario normal, una tarea recurrente que consista en escribir, cada 2 minutos, el mensaje "Hola, probando a usar crontab" en uno de los terminales que tiene abierto. Compruebe que funciona y elimine la tarea.
**Respuesta:**
1. Obtener la terminal actual con `tty` (ej. `/dev/pts/2`).
2. Editar el cron:
   ```bash
   crontab -e
   ```
3. Añadir la siguiente línea:
   ```cron
   */2 * * * * echo "Hola, probando a usar crontab" > /dev/pts/2
   ```
4. Para eliminar la programación:
   ```bash
   crontab -r
   ```

---

### 164. Ver contenido de anacrontab.
**Respuesta:**
```bash
cat /etc/anacrontab
```
Muestra las directivas para tareas periódicas (`cron.daily`, `cron.weekly`, `cron.monthly`) indicando: período en días, retardo en minutos tras el arranque para no saturar el sistema, identificador del trabajo y comando a ejecutar mediante `run-parts`.

---

# TEMA 5: Arranque del Sistema

### 165. Implemente un mecanismo para generar un log en /var/log/arranque.log que registre el día y la hora a la que se arranca y para el sistema.
**Respuesta:**
Creando un servicio de `systemd`:
```bash
sudo tee /etc/systemd/system/registro-arranque.service << 'EOF'
[Unit]
Description=Registro de arranque y parada
DefaultDependencies=no
Before=shutdown.target
After=local-fs.target

[Service]
Type=oneshot
RemainAfterExit=true
ExecStart=/bin/sh -c 'echo "Sistema arrancado: $(date)" >> /var/log/arranque.log'
ExecStop=/bin/sh -c 'echo "Sistema parado: $(date)" >> /var/log/arranque.log'

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable registro-arranque.service
```

---

### 166. Pruebe a apagar el sistema desde un terminal.
**Respuesta:**
```bash
sudo poweroff
# O bien:
sudo shutdown -h now
# O bien:
sudo init 0
```

---

### 167. ¿systemd? ¿modo emergencia? ¿recuperación de contraseña?
**Respuesta:**
- **systemd:** Es el gestor de inicio del sistema y de servicios (PID 1) en las distribuciones GNU/Linux modernas, basado en unidades (`units`: `.service`, `.target`, `.mount`).
- **Modo emergencia:** Nivel de recuperación mínimo con el sistema de ficheros raíz en modo solo lectura (`emergency.target` o `rescue.target`). Se accede añadiendo `systemd.unit=emergency.target` en la línea del kernel en GRUB.
- **Recuperación de contraseña de root:** Al encender el equipo, pulsar la tecla `e` sobre la entrada en el menú de GRUB, localizar la línea que comienza por `linux /boot/vmlinuz...` y añadir al final:
  ```
  init=/bin/bash
  ```
  Pulsar `Ctrl+X` para arrancar. Una vez en el prompt directo de root:
  ```bash
  mount -o remount,rw /
  passwd root
  sync
  reboot -f
  ```

---

# TEMA 6: Almacenamiento, Sistemas de Ficheros y LVM

### 168. Use el comando "losetup" sin parámetros para ver qué dispositivos de loopback tiene configurado el sistema operativo.
**Respuesta:**
```bash
sudo losetup
# O bien:
sudo losetup -a
```
Lista todos los dispositivos `/dev/loopX` activos (frecuentes en paquetes snap) indicando el fichero imagen asociado y opciones de desplazamiento.

---

### 169. Cree un fichero de 100MB usando "dd". Utilice "losetup" para asociar ese fichero a un dispositivo de loopback que no esté en uso (ej: /dev/loop100). Compruebe que se ha asociado correctamente.
**Respuesta:**
```bash
dd if=/dev/zero of=disco_loop.img bs=1M count=100
sudo losetup /dev/loop100 disco_loop.img
sudo losetup /dev/loop100
```

---

### 170. A continuación use "file" para averiguar el contenido del fichero recién creado. Use "cfdisk" para crear una partición que ocupe todo el "disco" (ej: /dev/loop100) y, hecho esto, compruebe si ahora "file" le indica otra cosa.
**Respuesta:**
```bash
file disco_loop.img             # Muestra: "data"
sudo cfdisk /dev/loop100
```
Dentro de `cfdisk`, se selecciona tabla de particiones `dos` (MBR), se crea una partición primaria con todo el espacio y se guarda con `Write`.
```bash
file disco_loop.img
```
Ahora `file` detecta la estructura: `DOS/MBR boot sector; partition 1 : ID=0x83, active, start-CHS...`.

---

### 171. Borre el fichero creado en la tarea 169.
**Respuesta:**
```bash
sudo losetup -d /dev/loop100
rm disco_loop.img
```

---

### 172. Use "cfdisk" para crear una partición que ocupe todo el disco /dev/sdb. Comprobar si funciona tecleando `ls -la /dev/sdb*` y comprobar que hay un sdb1.
**Respuesta:**
```bash
sudo cfdisk /dev/sdb
```
Se selecciona `dos` (o `gpt`), nueva partición con el 100% del tamaño, se escribe (`Write`) y se sale (`Quit`).
```bash
ls -la /dev/sdb*
```
Se observa la existencia de `/dev/sdb` y de la nueva partición `/dev/sdb1`.

---

### 173. Pruebe si funciona "blkid" sobre el disco /dev/sdb.
**Respuesta:**
```bash
sudo blkid /dev/sdb
sudo blkid /dev/sdb1
```
Si la partición es nueva y no tiene sistema de ficheros formateado, no devuelve UUID ni tipo (`TYPE`).

---

### 174. Use "free" para averiguar la cantidad de memoria RAM que tiene el sistema y el tamaño del swap. Indique también la cantidad que queda libre.
**Respuesta:**
```bash
free -h
```
Muestra en formato legible (`-h`):
- Fila `Mem`: memoria física total, en uso, libre (`free`), compartida, buffers/caché y disponible.
- Fila `Swap`: tamaño total de intercambio y su espacio libre.

---

### 175. Cree un fichero "/swapfile0" de 500MB. Use "mkswap" para iniciar ese fichero como swap y compruebe, mediante "file", que ahora ese fichero es apto para usarse como swap. Use ahora "swapon" para activar ese fichero como parte de swap del sistema. Advertencia sobre permisos y solución. Compruebe con "free". Desactive y borre.
**Respuesta:**
```bash
sudo dd if=/dev/zero of=/swapfile0 bs=1M count=500
sudo mkswap /swapfile0
file /swapfile0                  # Linux swap file format ...
sudo swapon /swapfile0
```
- **Advertencia:** `swapon: /swapfile0: insecure permissions 0644, 0600 suggested`. Significa que usuarios locales podrían leer información volcada de la memoria en la swap.  
- **Solución:**
  ```bash
  sudo chmod 600 /swapfile0
  ```
- **Comprobación y eliminación:**
  ```bash
  free -h
  sudo swapoff /swapfile0
  sudo rm /swapfile0
  ```

---

### 176. Usar el programa "stress" para crear procesos que consuman mucha memoria y comprobar qué pasa con free. Comprobar que si un fichero de swap está en uso no se puede desactivar el swap.
**Respuesta:**
```bash
sudo apt-get install -y stress
stress --vm 2 --vm-bytes 80% --timeout 30s &
free -m
```
El consumo de swap se incrementa. Si se intenta `sudo swapoff -a` mientras la memoria física está saturada, el comando se bloquea o falla con `swapoff: cannot allocate memory` al no caber los datos del swap en la memoria RAM.

---

### 177. Partiendo del dispositivo /dev/sdb1 use "mkfs" para crear en ese dispositivo un sistema de ficheros tipo ext4 en la primera y única partición: `mkfs -v -t ext4 /dev/sdb1`. Responda a las preguntas sobre bloques, tamaños, reserva para root, inodos y copia del superbloque.
**Respuesta:**
```bash
sudo mkfs -v -t ext4 /dev/sdb1
```
En la salida detallada se localizan:
- **Tamaño de bloque:** Típicamente `4096 bytes` (4 KB).
- **Tamaño total:** Multiplicación de bloques totales por 4096 bytes.
- **Espacio reservado para root:** Por defecto el 5% de los bloques (`Block count * 0.05 * 4096 bytes`).
- **Número máximo de ficheros (inodos):** Indicado en la línea `Inodes: X`.
- **Primera copia del superbloque:** Indicada en la lista de backups: bloque `32768`.

---

### 178. Cree un directorio "/pto1" vacío y use "mount" para montar /dev/sdb1 en /pto1. Muestre su contenido. ¿Existe algo? Pruebe con `df -k .`. Cree un pequeño fichero de texto.
**Respuesta:**
```bash
sudo mkdir /pto1
sudo mount /dev/sdb1 /pto1
cd /pto1
ls -la
```
- ¿Existe algo?: Sí, la carpeta del sistema **`lost+found`** creada por `ext4` para recuperación tras fallos.
```bash
df -k .
echo "Prueba de escritura" | sudo tee prueba.txt
```

---

### 179. Cambie a su directorio de usuario y desmonte /pto1. ¿Contiene algo ahora /pto1? Cree un nuevo fichero en /pto1 y vuelva a montar /dev/sdb1. ¿Cuál de los dos se ve? ¿Y si desmonta?
**Respuesta:**
```bash
cd ~
sudo umount /pto1
ls -la /pto1                     # Queda completamente vacío
sudo touch /pto1/fichero_base.txt
sudo mount /dev/sdb1 /pto1
ls -la /pto1                     # Se ve "prueba.txt" (oculta al fichero base subyacente)
sudo umount /pto1
ls -la /pto1                     # Vuelve a ser visible "fichero_base.txt"
```

---

### 180. Monte el sistema de ficheros con la opción "ro" (solo lectura). ¿Se podría crear ahí un nuevo directorio? Compruebe si es posible. Desmonte.
**Respuesta:**
```bash
sudo mount -o ro /dev/sdb1 /pto1
cd /pto1
sudo mkdir subdir_prueba
```
Falla con el error: **`mkdir: cannot create directory 'subdir_prueba': Read-only file system`**, a pesar de que los permisos lógicos sean de escritura para root.
```bash
cd ~
sudo umount /pto1
```

---

### 181. Monte nuevamente el sistema de ficheros y vaya al directorio /pto1. Pruebe a desmontar /pto1. ¿Por qué obtiene error? Averigüe qué procesos lo usan mediante "lsof" o "fuser".
**Respuesta:**
```bash
sudo mount /dev/sdb1 /pto1
cd /pto1
sudo umount /pto1
```
- Devuelve: `umount: /pto1: target is busy`.
- Ocurre porque la shell actual tiene su directorio de trabajo (`CWD`) situado dentro del punto de montaje.
- Diagnóstico:
  ```bash
  fuser -v /pto1
  # O bien:
  lsof /pto1
  ```

---

### 182. Cuotas de disco en /dev/sdb1: Montaje con "quota", quotacheck, cuotas de bloques e inodos con edquota. Comprobación con `quota -v`.
**Respuesta:**
```bash
sudo mkfs.ext4 -F /dev/sdb1
sudo mount -o usrquota,grpquota /dev/sdb1 /pto1
sudo quotacheck -cugm /pto1
sudo quotaon -v /pto1
sudo mkdir /pto1/dir1
sudo chown usuario:usuario /pto1/dir1
```
1. Consulta inicial:
   ```bash
   quota -v
   ```
   No muestra límites establecidos (cuotas en 0).
2. Como root, editar cuotas:
   ```bash
   sudo edquota usuario
   ```
   Configurar: `blocks: soft=1000 hard=1500; inodes: soft=5 hard=6`.
3. Creación de ficheros:
   ```bash
   cd /pto1/dir1
   touch f1 f2 f3 f4 f5
   quota -v
   ```
   Supera el límite *soft* de inodos y se inicia el periodo de gracia (*grace period*).
   ```bash
   touch f6   # Se crea (alcanza el hard limit de 6 inodos)
   touch f7   # Falla: Disk quota exceeded
   ```

---

### 183. LVM: 5 discos (sdb a sdf). Particionar como tipo LVM y usar "pvcreate" para crear volúmenes físicos en sdb1..sdf1. Compruebe la salida de `pvdisplay /dev/sdc1`.
**Respuesta:**
```bash
for d in b c d e f; do
    sudo parted -s /dev/sd$d mklabel msdos mkpart primary ext2 1MiB 100% set 1 lvm on
    sudo pvcreate /dev/sd${d}1
done
sudo pvdisplay /dev/sdc1
```
Muestra `Allocatable: NO` porque aún no ha sido incorporado a ningún *Volume Group* (VG).

---

### 184. Mediante "vgcreate" cree un grupo de volúmenes llamado "ASILAB" con los 5 PVs. Compare `pvdisplay /dev/sdc1` y responda: tamaño de PE, cantidad de PE, tamaño real y tamaño total en `vgdisplay`.
**Respuesta:**
```bash
sudo vgcreate ASILAB /dev/sdb1 /dev/sdc1 /dev/sdd1 /dev/sde1 /dev/sdf1
sudo pvdisplay /dev/sdc1
sudo vgdisplay ASILAB
```
- **Tamaño de PE (Physical Extent):** 4.00 MiB por defecto.
- **Cantidad de PE:** Tamaño del disco dividido por 4 MiB (ej. en 100MB son 24 PEs de 4MB = 96MB).
- **Tamaño real utilizable:** $N_{\text{PE}} \times 4\text{ MiB}$ (el espacio sobrante menor a 4MB no se puede asignar).
- **Tamaño total:** Suma de todas las PEs de los 5 discos mostrada en la línea `VG Size` de `vgdisplay`.

---

### 185. Use "lvcreate" para crear "lvol0" (75MB) y "lvol1" (125MB) en ASILAB. Compruebe con `ls /dev/ASILAB`. ¿Por qué "lvcreate" redondea el tamaño?
**Respuesta:**
```bash
sudo lvcreate -L 75M -n lvol0 ASILAB
sudo lvcreate -L 125M -n lvol1 ASILAB
ls -l /dev/ASILAB/
```
Redondea el tamaño porque las particiones lógicas en LVM se asignan en múltiplos exactos del tamaño de extensión física (**PE**, 4 MiB). Un volumen de 75 MiB se redondea al múltiplo superior más cercano (19 PEs = 76 MiB).

---

### 186. Cree en los dos volúmenes lógicos sistemas de ficheros. Móntelos en /pto1 y /pto2. Use "df" para comprobar su tamaño.
**Respuesta:**
```bash
sudo mkfs.ext4 /dev/ASILAB/lvol0
sudo mkfs.ext4 /dev/ASILAB/lvol1
sudo mkdir -p /pto1 /pto2
sudo mount /dev/ASILAB/lvol0 /pto1
sudo mount /dev/ASILAB/lvol1 /pto2
df -h /pto1 /pto2
```

---

### 187. Aumente el volumen lógico más grande en 100MB con "lvresize" redimensionando también el sistema de ficheros.
**Respuesta:**
```bash
sudo lvresize -r -L +100M /dev/ASILAB/lvol1
df -h /pto2
```
La opción `-r` (`--resizefs`) invoca internamente a `resize2fs` para expandir el sistema de ficheros al nuevo tamaño en caliente.

---

### 188. Mediante `pvs` indique si hay algún volumen físico que no se esté usando y si alguno está completamente en uso.
**Respuesta:**
```bash
sudo pvs
```
En las columnas `PSize` y `PFree`:
- Si `PFree == PSize`, el disco no tiene ninguna PE asignada.
- Si `PFree == 0`, el disco está 100% ocupado.

---

### 189. Probar uso de snapshots en LVM (comprobar que al montar el snapshot es una imagen "congelada").
**Respuesta:**
```bash
echo "Datos originales" | sudo tee /pto1/fichero.txt
sudo lvcreate -L 20M -s -n lvol0_snap /dev/ASILAB/lvol0
echo "Modificacion posterior" | sudo tee -a /pto1/fichero.txt
sudo mkdir -p /mnt/snapshot
sudo mount -o ro /dev/ASILAB/lvol0_snap /mnt/snapshot
cat /mnt/snapshot/fichero.txt   # Muestra solo "Datos originales"
sudo umount /mnt/snapshot
```

---

# TEMA 7: Logs y Gestión de Paquetes

### 190. Mediante "logger" compruebe que se pueden generar eventos de log (use la "facility" y el nivel que quiera) que luego se pueden ver en alguno de los ficheros de log que se han visto.
**Respuesta:**
```bash
logger -p local0.notice "Mensaje de prueba enviado por logger para ASI"
sudo grep "Mensaje de prueba" /var/log/syslog
```

---

### 191. Use "apt-get" para actualizar el listado de los paquetes disponibles y, a continuación, actualice su sistema operativo.
**Respuesta:**
```bash
sudo apt-get update
sudo apt-get upgrade -y
```

---

# TEMA 11: Redes y Núcleo

### 192. Busque en sysctl.conf si hay alguna directiva que permita que su sistema Linux se comporte como un router (es decir, que tenga activo el reenvío de datagramas). Indique si cree que esa directiva está activa en este momento (compruébelo con el fichero correspondiente en /proc/sys).
**Respuesta:**
```bash
grep 'ip_forward' /etc/sysctl.conf
cat /proc/sys/net/ipv4/ip_forward
```
- Directiva: `net.ipv4.ip_forward = 1`.
- Si el fichero `/proc/sys/net/ipv4/ip_forward` devuelve `0`, el reenvío está **desactivado**. Si devuelve `1`, está activo.

---

### 193. Desactivar respuesta a ping. Comprobar `/proc/sys/net/ipv4/icmp_echo_ignore_all`. ¿Funciona `ping 127.0.0.1`? ¿Es persistente tras reinicio?
**Respuesta:**
```bash
cat /proc/sys/net/ipv4/icmp_echo_ignore_all   # Muestra 0 (activo)
sudo sh -c 'echo 1 > /proc/sys/net/ipv4/icmp_echo_ignore_all'
```
- Desde otro PC o localmente: los paquetes ICMP Echo Request son descartados silenciosamente sin contestar.
- `ping 127.0.0.1` **tampoco responde**, porque la regla ignora todo tráfico ICMP entrante sin distinción de interfaz.
- **No es persistente:** los valores en `/proc` residen en RAM. Para hacerlo permanente tras reinicios se añade a `/etc/sysctl.conf`:
  ```ini
  net.ipv4.icmp_echo_ignore_all = 1
  ```
  y se aplica con `sudo sysctl -p`.

---

# TEMA WINDOWS: Administración y Seguridad

### 194. Usar MMC para añadir una nueva herramienta a la consola. Se añadirán directivas de equipo local y de administradores.
**Respuesta:**
1. Pulsar `Win + R`, escribir `mmc` y pulsar Enter.
2. Archivo $\rightarrow$ Agregar o quitar complemento... (`Ctrl + M`).
3. Seleccionar **Editor de objetos de directiva de grupo**, pulsar *Agregar*, mantener "Equipo local" y pulsar *Finalizar*.
4. Volver a agregar otro complemento de directivas, pulsar *Examinar*, pestaña *Usuarios*, seleccionar el grupo **Administradores** y aceptar.

---

### 195. Añadir un usuario "normal" y probar con el editor de políticas de grupo a cambiar el modo de inicio de sesión e impedir que un usuario no administrador pueda cambiar la contraseña.
**Respuesta:**
1. Crear el usuario:
   ```cmd
   net user normal Password123! /add
   ```
2. En `gpedit.msc`:  
   Configuración del equipo $\rightarrow$ Configuración de Windows $\rightarrow$ Configuración de seguridad $\rightarrow$ Directivas locales $\rightarrow$ Opciones de seguridad.
3. Modificar la directiva de inicio interactivo para requerir pulsación de `Ctrl + Alt + Supr`.
4. En Configuración de usuario $\rightarrow$ Plantillas administrativas $\rightarrow$ Sistema $\rightarrow$ Opciones de Ctrl+Alt+Supr: habilitar **Quitar "Cambiar contraseña"**.

---

### 196. Acceda a Panel de Control -> Sistema y compruebe el nombre del equipo y si está conectado a un dominio. Pruebe a asociarse a un dominio llamado wgrp.es ¿qué pasa?
**Respuesta:**
Muestra el nombre NetBIOS del equipo y que pertenece a un grupo de trabajo por defecto (`WORKGROUP`).  
Al intentar unirse al dominio `wgrp.es`: Tras unos segundos intentando resolver el registro SRV DNS del controlador de dominio (`_ldap._tcp.dc._msdcs.wgrp.es`), muestra un error indicando que **no se pudo contactar con un controlador de Active Directory para el dominio**.

---

### 197. Abrir MMC y añadir: gestión de usuarios, editor de objetos de directiva de grupo. Ver listado de usuarios, estado de "Administrador" y SID de usuario practicas. Ver miembros de grupos "usuarios" y "administradores".
**Respuesta:**
En `mmc`: añadir el complemento **Usuarios y grupos locales** (`lusrmgr.msc`).
- Estado de "Administrador": Cuenta integrada deshabilitada por defecto (icono con flecha hacia abajo).
- SID de usuario: Se consulta abriendo PowerShell con:
  ```powershell
  Get-LocalUser -Name practicas | Select-Object Name, SID
  ```
- Miembros: En la carpeta *Grupos*, doble clic en `Administradores` y `Usuarios` para revisar sus cuentas integrantes.

---

### 198. Ver el listado de los usuarios que hay. Comprobar si el usuario "Administrador" aparece o no en la lista de usuarios y en qué estado está. Averiguar el SID del usuario "normal" creado anteriormente.
**Respuesta:**
```cmd
net user
```
El usuario `Administrador` sí figura en el listado. En sus propiedades (`net user Administrador`) indica `Cuenta activa: No`.  
Para consultar el SID del usuario `normal`:
```cmd
wmic useraccount where name='normal' get name,sid
```

---

### 199. Ver el listado de grupos. Comprobar los miembros de Usuarios, Usuarios Avanzados, Administradores y Operadores de backup.
**Respuesta:**
```cmd
net localgroup "Usuarios"
net localgroup "Usuarios avanzados"
net localgroup "Administradores"
net localgroup "Operadores de copia de seguridad"
```

---

### 200. Iniciar sesión como usuario "practica" y ver los permisos asociados a \Users\Practica. Probar a añadir algún permiso a "Mis Documentos" (para que lo pueda ver el usuario "normal").
**Respuesta:**
1. Clic derecho en `C:\Users\Practica\Documents` $\rightarrow$ Propiedades $\rightarrow$ pestaña **Seguridad**.
2. Pulsar *Editar...* $\rightarrow$ *Agregar...*, escribir `normal` y presionar *Comprobar nombres*.
3. Asignar permisos de **Lectura y ejecución** y pulsar *Aceptar*.

---

### 201. Intentar entrar como usuario práctica (que es administrador) en el directorio del usuario normal. No debe poderse. Ver que es necesario para acceder cambiar los permisos y que para eso es necesario tomar la propiedad de la carpeta.
**Respuesta:**
Al acceder a `C:\Users\normal`, Windows muestra un mensaje de acceso denegado requiriendo confirmación de administrador.  
Para acceder formalmente: Propiedades de la carpeta $\rightarrow$ Seguridad $\rightarrow$ Opciones avanzadas $\rightarrow$ Propietario $\rightarrow$ Cambiar $\rightarrow$ Asignar al usuario `practicas` (o a Administradores) y marcar la casilla **Reemplazar propietario en subcontenedores y objetos**.

---

### 202. Borrar (desde la MMC) la cuenta del usuario normal creado y ver cómo se quedan los permisos de la carpeta de ese usuario normal.
**Respuesta:**
Tras eliminar la cuenta en `lusrmgr.msc`:  
Al revisar la pestaña *Seguridad* de `C:\Users\normal`, el nombre del usuario desaparece de la lista de ACL y en su lugar se muestra una cadena de SID huérfana de la forma **`S-1-5-21-...`**, dado que el sistema ya no puede resolver el identificador en la base de datos SAM local.

---

### 203. Acceder con secpol.msc a los derechos de usuario. Cambiar algún permiso.
**Respuesta:**
1. Ejecutar `secpol.msc`.
2. Directivas locales $\rightarrow$ Asignación de derechos de usuario.
3. Localizar directivas críticas como **Apagar el sistema** o **Permitir el inicio de sesión local** y modificar la lista de usuarios o grupos autorizados.

---

### 204. Registro de Windows (HKEY_CURRENT_USER). Fichero ntuser.dat.
**Respuesta:**
1. Abrir `regedit`. La clave `HKEY_CURRENT_USER` mapea dinámicamente la configuración del perfil activo.
2. Cada directorio personal de usuario (`C:\Users\<usuario>`) almacena dicho subárbol en el fichero de sistema oculto **`NTUSER.DAT`**. Al iniciar sesión, el kernel de Windows monta (*hive load*) este archivo en `HKEY_USERS\<SID_del_usuario>` y genera el enlace simbólico a `HKEY_CURRENT_USER`.

---

### 205. Comprobar la configuración de UAC por defecto. Comportamiento en cuenta de administrador vs cuenta de usuario normal.
**Respuesta:**
- Configuración UAC por defecto: Notificar solo cuando las aplicaciones intenten realizar cambios en el equipo (atenuando el escritorio con el *Secure Desktop*).
- **Desde cuenta `practicas` (administrador):** Salta la pantalla atenuada de UAC solicitando únicamente confirmación mediante el botón **"Sí"** (*Consent Prompt*).
- **Desde cuenta `normal` (estándar):** Salta la pantalla atenuada exigiendo introducir las credenciales de una cuenta administrativa (*Credential Prompt*).

---

### 206. Iniciar sesión como usuario practica y ver diferencias entre arrancar una consola normal o "Ejecutar como administrador". Comprobar con `whoami /priv` y `whoami /groups`.
**Respuesta:**
- **Consola normal:**
  ```cmd
  whoami /groups
  ```
  El grupo `Administradores` aparece marcado con el atributo **Solo para denegar grupo** (*Group used only for deny*), y la mayoría de privilegios administrativos se encuentran ausentes.
- **Consola con botón derecho -> "Ejecutar como administrador":**
  ```cmd
  whoami /priv
  ```
  Se activa el token de acceso elevado completo con privilegios administrativos (`SeDebugPrivilege`, `SeShutdownPrivilege`, `SeTakeOwnershipPrivilege`, etc.).