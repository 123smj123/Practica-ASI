# ESTUDIO EXPERIMENTAL - PRÁCTICA 3
## Usuarios, Ficheros y Permisos
**Administración de Sistemas de Información (ASI)**  
*Dpto. Tecnología Electrónica - Universidad de Sevilla*  
*Curso 2026/27*

---

# Primera parte: Warming up

### 1. Indique a qué grupo principal y a qué grupos secundarios pertenecen tanto el usuario "normal" (es decir, asiuser) como el usuario "root".

**Respuesta:**

Para comprobar los grupos a los que pertenecen ambos usuarios se utiliza la orden `id`:

```bash
# Para el usuario normal:
id asiuser

# Para el superusuario:
id root
```

* **Usuario normal (`asiuser`):**
  * **Grupo principal:** `asiuser` (típicamente con GID 1000, según la política UPG (*User Private Group*)).
  * **Grupos secundarios:** Grupos administrativos o de hardware como `adm`, `dialout`, `cdrom`, `sudo`, `dip`, `plugdev`, `lxd`, entre otros.
* **Usuario superusuario (`root`):**
  * **Grupo principal:** `root` (GID 0).
  * **Grupos secundarios:** En instalaciones base suele pertenecer únicamente a su grupo principal `root` (GID 0).

---

### 2. En el directorio de dispositivos existe un fichero llamado "ttyS0" que permite acceder al primer puerto serie del PC. Ese fichero en concreto, ¿Es un fichero de dispositivo? Justifíquelo.

**Respuesta:**

Se comprueba ejecutando un listado largo en `/dev`:

```bash
ls -l /dev/ttyS0
```

Salida típica:
```text
crw-rw---- 1 root dialout 4, 64 sep 30 10:00 /dev/ttyS0
```

**Justificación:**  
Sí, es un fichero de dispositivo de tipo **carácter**. Se justifica mediante dos observaciones en los metadatos:
1. El primer carácter en la cadena de permisos es una **`c`** (*character device*), lo que indica al kernel que los datos se transfieren byte a byte en flujo serie sin almacenamiento en bloques de caché.
2. En lugar del tamaño en bytes que muestra un archivo regular, figuran dos números separados por coma (`4, 64`), correspondientes al **Major number** (identifica el controlador/driver en el kernel) y al **Minor number** (identifica el puerto o instancia física concreta gestionada por dicho controlador).

---

### 3. Indique, de forma justificada, si el usuario "normal" tiene acceso al primer puerto serie del PC.

**Respuesta:**

Consultando los permisos obtenidos en la tarea anterior:
* Propietario: `root` con permisos de lectura y escritura (`rw-`).
* Grupo asignado: `dialout` con permisos de lectura y escritura (`rw-`).
* Resto del sistema (*others*): sin ningún permiso (`---`).

**Justificación:**  
El usuario `asiuser` tendrá acceso **única y exclusivamente si es miembro del grupo `dialout`**. Si en la salida de `id asiuser` figura `dialout`, podrá leer y escribir en el puerto serie. De no pertenecer a dicho grupo secundario, al evaluarse los permisos para *others* (`0`), el kernel le denegará el acceso devolviendo `Permission denied`.

---

### 4. NOTA: El resto de las tareas de la primera parte se deben llevar a cabo como usuario normal.

**Respuesta:**  
Indicación metodológica: todas las órdenes de los apartados 5, 6 y 7 deben ejecutarse bajo la shell de `asiuser` (prompt con `$`).

---

### 5. Como usuario "normal", intente crear (en su directorio de usuario) un enlace simbólico y un enlace duro a /bin/ls. ¿Es posible?

**Respuesta:**

```bash
cd ~
# Intento de enlace simbólico:
ln -s /bin/ls ./enlace_simb

# Intento de enlace duro:
ln /bin/ls ./enlace_duro
```

* **Enlace simbólico:** **Sí es posible**. El usuario únicamente necesita permisos de escritura en su propio `$HOME` para crear la nueva entrada de tipo enlace que contiene como texto la ruta `/bin/ls`.
* **Enlace duro:** **Depende del esquema de sistemas de ficheros y de la configuración de seguridad:**
  1. Si `/bin` (o `/usr/bin`) y `/home` residen en sistemas de ficheros/particiones diferentes, `ln` fallará con el error `Invalid cross-device link`, ya que los enlaces duros no pueden cruzar diferentes sistemas de ficheros.
  2. Aunque residan en la misma partición raíz, los kernels modernos de Linux (incluido Ubuntu) implementan la protección `fs.protected_hardlinks = 1`. Esta directiva del núcleo impide que un usuario cree enlaces duros a ficheros sobre los que no tiene propiedad, a menos que ya tenga acceso de lectura y escritura directo sobre ellos, impidiendo enlazar directamente a binarios del sistema como `/bin/ls`.

---

### 6. El comando "ps" permite a un usuario saber los procesos asociados a ese terminal y le indica, entre otras cosas, el terminal que está usando. Dicho terminal se corresponde con un fichero bajo el directorio /dev. Teclee ese comando e indique los permisos que tiene el fichero de dispositivo que se corresponde con el terminal que está usando.

**Respuesta:**

```bash
ps
```
En la columna `TTY` se observa la terminal asignada, por ejemplo `pts/0` (o `pts/1`). Para consultar sus atributos:

```bash
ls -l /dev/pts/0
```

Salida típica:
```text
crw--w---- 1 asiuser tty 136, 0 oct 15 11:20 /dev/pts/0
```

* **Tipo:** Carácter (`c`).
* **Permisos:** En formato numérico **`620`** (`crw--w----`).
  * Propietario (`asiuser`): lectura y escritura (`rw-`).
  * Grupo (`tty`): solo escritura (`-w-`).
  * Otros (*others*): sin permisos (`---`).

---

### 7. ¿Qué ocurre si tecleo echo "Hola" > /dev/XXX (XXX se corresponde con el terminal que descubrió en el apartado anterior). ¿Qué ocurre? A la vista de lo que ha pasado y teniendo en cuenta la respuesta que dio en el apartado anterior ¿cree que otros usuarios podrían escribir en su terminal?

**Respuesta:**

```bash
echo "Hola" > /dev/pts/0
```

* **Qué ocurre:** La cadena `"Hola"` se redirige al flujo de entrada del dispositivo y se imprime directamente por pantalla en la ventana de terminal activa.
* **¿Podrían otros usuarios escribir en su terminal?:**  
  **No de forma directa**, dado que el campo *others* tiene permisos `---` (cero). Ningún usuario ajeno ordinario puede redirigir flujos hacia nuestro pseudoterminal.  
  *Excepción:* Aquellos procesos o usuarios que pertenezcan al grupo del sistema `tty` sí cuentan con permiso `-w-` (escritura). Este mecanismo es el que emplean órdenes de comunicación local entre usuarios como `write` o `wall` (que cuentan con el bit SGID para el grupo `tty`).

---

# Segunda parte: Creando múltiples cuentas y grupos

### 8. Elabore un esquema de usuarios, grupos, directorios y permisos que le permita cumplir los siguientes requisitos (NOTA: Debe hacer el diseño "en papel" y mostrárselo al profesor antes de implementarlo):
* **Tanto los directivos como los administradores de sistemas deben pertenecer también al grupo de "otros". Nota: "adduser usuario grupo" permite hacer esto.**
* **Se debe establecer una estructura de directorios de usuarios (/home) tal que asegure que los ficheros que estén ubicados en el $HOME de los usuarios del grupo "dire" no puedan ser accedidos por los usuarios que NO pertenece a ese grupo. NOTA: Esto debe cumplirse incluso si los usuarios de ese grupo establecen incorrectamente los permisos de su $HOME. PISTA: El directorio de los usuarios no tiene por qué "colgar" directamente de /home.**

**Respuesta:**

#### Esquema de diseño:

1. **Grupos del sistema a crear:**
   * `dire`: Grupo para el equipo directivo.
   * `utec`: Grupo para administradores de sistemas.
   * `otros`: Grupo común al que pertenecerán todos los usuarios del sistema.

2. **Estructura de Directorios Jerárquica:**
   En lugar de alojar las carpetas personales directamente bajo `/home`, se crea un contenedor intermedio exclusivo para la directiva:
   * `/home/dire`: Directorio intermedio contenedor.
     * Propietario: `root:dire`
     * Permisos: **`750`** (`drwxr-x---`)
   * `/home/utec`: Contenedor para técnicos (opcional o ubicados en `/home`).
   * `/home/otros`: Contenedor para el resto de usuarios ordinarios.

3. **Justificación del mecanismo de seguridad:**
   Para acceder a cualquier fichero o directorio hijo en Linux, el kernel exige disponer obligatoriamente de permiso de paso o ejecución (**`x`**) en **todos** los directorios ancestros que conforman la ruta absoluta.  
   Al configurar `/home/dire` con permisos `750` (`rwxr-x---`) y grupo `dire`:
   * Los usuarios de `dire` tienen permiso de paso (`x`).
   * El resto de usuarios (*others*) tienen denegado todo acceso (`---`).  
   **Consecuencia:** Aunque un directivo modifique por error los permisos de su propio `$HOME` (por ejemplo `/home/dire/usu1`) poniéndolo a `777`, ningún usuario fuera de `dire` podrá llegar hasta él, pues la comprobación de paso fallará de manera insalvable en el directorio padre `/home/dire`.

---

### 9. Cree 3 cuentas (una de cada tipo de usuario). Asigne los permisos a los directorios de forma que se cumplan los requisitos señalados en el apartado anterior. Indique la salida del comando "ls -la /home" (Nota: Basta con indicar los permisos en formato numérico, el propietario, el grupo y el nombre del directorio).

**Respuesta:**

#### 1. Creación de grupos y estructura:
```bash
sudo addgroup dire
sudo addgroup utec
sudo addgroup otros

sudo mkdir -p /home/dire
sudo chown root:dire /home/dire
sudo chmod 750 /home/dire
```

#### 2. Creación de los tres usuarios:
* **Directivo (`usu1`):**
  ```bash
  sudo adduser --home /home/dire/usu1 --ingroup dire usu1
  sudo adduser usu1 otros
  ```
* **Técnico (`usu2`):**
  ```bash
  sudo adduser --ingroup utec usu2
  sudo adduser usu2 otros
  ```
* **Usuario estándar (`usu3`):**
  ```bash
  sudo adduser --ingroup otros usu3
  ```

#### 3. Salida de `ls -la /home`:
```text
drwxr-xr-x  root   root   /home          (755)
drwxr-x---  root   dire   /home/dire     (750)
drwxr-xr-x  usu2   utec   /home/usu2     (755)
drwxr-xr-x  usu3   otros  /home/usu3     (755)
drwxr-xr-x  asiuser asiuser /home/asiuser (755)
```

---

### 10. Escriba la salida de "id -a usu1" donde usu1 es el nombre del usuario que pertenece al grupo "dire".

**Respuesta:**

```bash
id -a usu1
```

Salida devuelta:
```text
uid=1001(usu1) gid=1001(dire) groups=1001(dire),1003(otros)
```
*(Los identificadores numéricos UID y GID pueden variar según las cuentas previas existentes).*

---

# Tercera parte: Compartiendo información

### 11. Para facilitar la tarea de compartir información, se va a crear un directorio (/home/repositorio) que contenga una estructura de directorios que permita compartir información entre los distintos grupos de usuarios que existen en el sistema. Para ello, se establecen los siguientes requisitos:
* **Se creará un usuario llamado "repo" (grupo principal "nogroup") que tendrá la cuenta bloqueada (pero con una shell válida) y su $HOME será el directorio indicado anteriormente.**
* **En el directorio $HOME de ese usuario, existirá un directorio para los usuarios del grupo "dire" y otro adicional para todos los usuarios. Deben cumplirse los siguientes requisitos:**
  * **Los directorios serán propiedad del usuario "repo".**
  * **En el directorio destinado a todos los usuarios, debe ser posible que cualquier usuario del sistema cree un fichero o directorio, pero solo el propietario debe poder borrarlo.**
  * **En el directorio destinado al grupo "dire", solo debe permitirse que los usuarios que pertenecen a ese grupo puedan ver el contenido del directorio y crear ficheros o directorios. Los usuarios que no pertenezcan a ese grupo no deben tener acceso al directorio. Debe permitirse que un usuario (del grupo "dire") pueda borrar un fichero o directorio incluso si no es de su propiedad.**

**Respuesta:**

Análisis y definición de permisos para la estructura bajo `/home/repositorio`:

1. **Directorio base `/home/repositorio`:**
   * Propietario: `repo:nogroup`
   * Permisos: **`755`** (`drwxr-xr-x`). Debe permitir el acceso y paso a todos los usuarios del sistema para que puedan alcanzar los subdirectorios.

2. **Directorio para todos (`/home/repositorio/todos`):**
   * Propietario: `repo:nogroup`
   * Permisos: **`1777`** (`drwxrwxrwt`).
   * *Justificación:* Al tener `rwx` para *others*, cualquier usuario puede entrar, listar y crear ficheros. La activación del **Sticky Bit (`+t` / 1000)** garantiza que únicamente el propietario del fichero (o `root`) pueda renombrarlo o borrarlo.

3. **Directorio exclusivo para directivos (`/home/repositorio/directivos`):**
   * Propietario: `repo:dire`
   * Permisos: **`770`** (`drwxrwx---`).
   * *Justificación:* Propietario `repo` y grupo `dire`. Los miembros de `dire` tienen `rwx` (listar, crear y borrar). Al tener permiso de escritura en el directorio contenedor y **no tener el sticky bit**, cualquier directivo puede borrar archivos creados por otros miembros del mismo grupo. *Others* tiene `---` (cero acceso).

---

### 12. Lleve a cabo la tarea señalada en el apartado anterior. Además, debe indicar:
* **Qué ha tecleado para crear el usuario "repo".**
* **Describa, de forma razonada, los permisos que se han establecido en cada uno de los directorios del repositorio.**
* **Escriba la salida de "ls -lad /home/repositorio". Nota: Basta con indicar los permisos en formato numérico, el propietario, el grupo y el nombre del directorio.**

**Respuesta:**

#### 1. Creación del usuario `repo`:
```bash
sudo adduser --home /home/repositorio --ingroup nogroup --shell /bin/bash --disabled-login repo
```
*(Opcionalmente, para bloquearlo con passwd: `sudo passwd -l repo`).*

#### 2. Creación y asignación de permisos a los subdirectorios:
```bash
sudo mkdir /home/repositorio/todos
sudo chown repo:nogroup /home/repositorio/todos
sudo chmod 1777 /home/repositorio/todos

sudo mkdir /home/repositorio/directivos
sudo chown repo:dire /home/repositorio/directivos
sudo chmod 770 /home/repositorio/directivos
```

#### 3. Razonamiento de permisos:
* `/home/repositorio/todos` $\rightarrow$ **`1777`**: `rwx` generalizado para permitir libre creación, combinado con el **Sticky Bit** (`1...`) para impedir que los usuarios borren trabajos ajenos.
* `/home/repositorio/directivos` $\rightarrow$ **`0770`**: Restricción total a *others* (`0`) y pleno control colaborativo al grupo `dire` (`7`), permitiendo el borrado colectivo al omitirse el sticky bit.

#### 4. Salida de `ls -lad /home/repositorio`:
```text
755  repo  nogroup  /home/repositorio
```
*(En formato largo estándar: `drwxr-xr-x 4 repo nogroup 4096 oct 15 11:30 /home/repositorio`).*

---

### 13. Realice las siguientes pruebas con el objetivo de comprobar que todo se ha configurado correctamente:
* **Compruebe que el usuario asiuser (que no pertenece a ninguno de los grupos creados antes), puede crear un fichero en el directorio destinado a todos los usuarios. Compruebe que este usuario no puede acceder al otro directorio que está en /home/repositorio.**
* **Compruebe que un usuario con grupo principal "dire" no puede borrar el fichero creado en el apartado anterior.**
* **Compruebe que un usuario con grupo principal "dire" puede crear un fichero en el directorio del repositorio creado para esos usuarios específicamente.**

**Respuesta:**

#### Prueba 1: Acciones con `asiuser`
```bash
# Como asiuser:
touch /home/repositorio/todos/fich_asiuser.txt    # Tiene éxito
ls -ld /home/repositorio/directivos              # Se comprueba
cd /home/repositorio/directivos                  # Error: Permiso denegado
```

#### Prueba 2: Intento de borrado por parte de `usu1` (grupo `dire`)
```bash
# Como usu1:
rm /home/repositorio/todos/fich_asiuser.txt
```
* **Resultado:** Falla con el mensaje: `rm: cannot remove 'fich_asiuser.txt': Operation not permitted`.
* **Causa:** El sticky bit en `/home/repositorio/todos` restringe el borrado al dueño del archivo (`asiuser`).

#### Prueba 3: Creación de fichero en zona exclusiva por `usu1`
```bash
# Como usu1:
touch /home/repositorio/directivos/acuerdo.txt
ls -l /home/repositorio/directivos/acuerdo.txt
```
* **Resultado:** Se crea satisfactoriamente.

---

# Cuarta parte: Permisos especiales

### 14. Inicie sesión con el usuario "root" ¿está establecido el bit setuid en el fichero /usr/bin/su"? ¿En qué ha tenido que fijarse para saber la respuesta?

**Respuesta:**

```bash
ls -l /usr/bin/su
```
Salida típica:
```text
-rwsr-xr-x 1 root root 67816 mar 23  2024 /usr/bin/su
```

**Respuesta:** **Sí está establecido**.  
Nos hemos fijado en la tríada de permisos del propietario (`u`): aparece una **`s`** minúscula en lugar de la habitual `x` (`-rws...`). Al ser una `s` minúscula, indica además que el bit de ejecución subyacente también está presente.

---

### 15. Modifique los permisos de /usr/bin/su para que el bit setuid está desactivado y que el resto de los permisos no cambie. Indique lo que ha tecleado.

**Respuesta:**

```bash
chmod u-s /usr/bin/su
# O en formato numérico:
chmod 0755 /usr/bin/su
```

---

### 16. Inicie sesión en otro terminal como usuario "normal" e intente usar el programa "su" para abrir una Shell de superusuario ¿funciona? Explique por qué no funciona.

**Respuesta:**

```bash
su -
```

* **Resultado:** **No funciona**. Muestra un fallo de autenticación o el error `su: Authentication failure` / `su: must be suid to work on this system`.
* **Explicación:** `su` requiere leer el archivo `/etc/shadow` para validar la contraseña de root introducida. Dado que `/etc/shadow` tiene permisos `640` o `000` y pertenece a root, un binario ejecutado con el UID de un usuario normal no tiene privilegios para abrirlo. El bit **SUID** es indispensable para que el proceso adopte el UID efectivo 0 (`root`) durante la ejecución y pueda consultar las credenciales.

---

### 17. Vuelva a establecer los permisos de "su" de forma que tenga de nuevo el bit setuid activo. Indique lo que ha tecleado.

**Respuesta:**

```bash
chmod u+s /usr/bin/su
# O en modo numérico:
chmod 4755 /usr/bin/su
```

---

### 18. Compruebe que "su" funciona ahora correctamente.

**Respuesta:**

```bash
su -
```
Se introduce la clave de root y la elevación a superusuario se efectúa inmediatamente con éxito (comprobable mediante `id`, que devolverá `uid=0(root)`).

---

### 19. En Debian existe la orden "/usr/bin/systemctl poweroff" que nos permite hacer una parada ordenada del sistema y apagarlo una vez que se ha producido esto. ¿Puede un usuario no privilegiado apagar el sistema de esta forma? Compruébelo y anote el mensaje de error.

**Respuesta:**

Si se ejecuta desde una sesión no interactiva, vía SSH sin sesión local abierta o en terminales virtuales secundarias sin permisos en Polkit:
```bash
systemctl poweroff
```
* **Resultado:** No se permite el apagado directo.
* **Mensaje de error habitual:**  
  `Failed to set wall message, ignoring: Access denied`  
  `Failed to power off system via logind: Access denied`  
  o bien: `Interactive authentication required`.

---

### 20. ¿De qué forma se podría lograr que los usuarios pertenecientes al grupo "utec" puedan apagar el sistema sin necesidad de conocer la contraseña del superusuario (y sin usar "sudo" o "su"). Nota: Fuera del grupo "utec" solo el superusuario de poder apagar el sistema. Realice los cambios pertinentes, anote lo que ha hecho y compruebe que un usuario que pertenece al grupo "utec" puede apagar el sistema mientras que uno que no pertenece a ese grupo no puede.

**Respuesta:**

Para implementar este control mediante los permisos estándar de UNIX y permisos especiales vistos en la asignatura:

1. Localizar el ejecutable real del apagado (`/bin/systemctl` o el binario tradicional `/sbin/poweroff`).
2. Asignar el binario al grupo `utec`:
   ```bash
   sudo chown root:utec /sbin/poweroff
   ```
3. Configurar permisos con **SUID** (`4...`), restringiendo la ejecución solo al dueño y al grupo:
   ```bash
   sudo chmod 4750 /sbin/poweroff
   ```

* **Comprobación:**
  * Propietario (`root`): `rws` (ejecuta con privilegios de root).
  * Grupo (`utec`): `r-x` (puede ejecutarlo; gracias al bit SUID se eleva a root y apaga la máquina).
  * Otros (*others*): `---` (sin permiso de ejecución; si un usuario ajeno teclea `/sbin/poweroff`, el kernel le responde `Permission denied`).

---

### 21. Suponga que ahora desea que cualquier usuario pueda apagar el sistema sin necesidad de conocer la contraseña del superusuario y sin necesidad de pertenecer al grupo "utec". La idea es que se le pueda suministrar a un usuario un nombre de usuario (ej: "apagar") y contraseña que le permita apagar el sistema, pero que ese usuario no pueda hacer nada más. ¿Cómo se podría hacer esto? (Pista: Shell del usuario "apagar").

**Respuesta:**

Se crea una cuenta de usuario cuya **shell de inicio de sesión no sea un intérprete interactivo** (`/bin/bash`), sino directamente el binario o script encargado de apagar el equipo.

1. Se crea un script de apagado:
   ```bash
   echo '#!/bin/sh' | sudo tee /usr/local/bin/shell_apagar
   echo 'exec /sbin/poweroff' | sudo tee -a /usr/local/bin/shell_apagar
   sudo chmod 755 /usr/local/bin/shell_apagar
   ```
2. Se añade la ruta a `/etc/shells` para validarla como shell del sistema:
   ```bash
   echo '/usr/local/bin/shell_apagar' | sudo tee -a /etc/shells
   ```
3. Se crea el usuario `apagar` asignándole esta shell:
   ```bash
   sudo adduser --shell /usr/local/bin/shell_apagar apagar
   ```
4. Se asegura de que `/sbin/poweroff` disponga de los permisos necesarios (ej. SUID si no lo tuviera).

Al iniciar sesión como `apagar`, el sistema ejecuta la shell asignada, ordena el apagado y cierra el proceso, impidiendo que el usuario pueda escribir órdenes o interactuar con el sistema operativo.

---

### 22. Compruebe que lo que ha hecho en el apartado anterior funciona y anote los pasos que ha tenido que dar.

**Respuesta:**

1. Iniciar sesión desde una consola de terminal (`tty2` mediante `Ctrl+Alt+F2` o vía `su - apagar`).
2. Introducir el nombre de usuario `apagar` y su clave establecida.
3. El sistema valida las credenciales, invoca de inmediato `/usr/local/bin/shell_apagar` e inicia la secuencia de parada (`Powering off...`), sin presentar en ningún momento el prompt de la shell.

---

# Quinta parte: Listas de control de acceso (ACLs)

### 23. Compruebe si existen los ejecutables "getfacl" y "setfacl". Si no existen, el sistema de control de acceso a fichero basado en ACL no está instalado. Instale esta funcionalidad en el sistema ejecutado "apt-get install acl" como superusuario.

**Respuesta:**

```bash
which getfacl setfacl
```
Si el comando no devuelve ninguna ruta:
```bash
sudo apt-get update
sudo apt-get install -y acl
```

---

### 24. Lea la página de manual de "setfacl" a la hora de resolver el apartado 23.

**Respuesta:**

```bash
man setfacl
```
Sintaxis principal revisada:
* `setfacl -m u:usuario:permisos fichero`: Modifica la ACL añadiendo o alterando reglas para un usuario concreto.
* `setfacl -x u:usuario fichero`: Elimina la regla de ACL de un usuario.
* `setfacl -b fichero`: Elimina todas las entradas extendidas de ACL.

---

### 25. ¿Qué tendría que hacer si quisiera que el usuario "asiuser" pudiera tener acceso al directorio de los directivos que se creó en la parte segunda? Anote lo que ha hecho.

**Respuesta:**

Dado que `/home/dire` tenía permisos base `750` (`root:dire`) denegando el paso a *others*, se aplica una entrada de control de acceso extendida (ACL) sobre dicho directorio para otorgar permisos de lectura y paso a `asiuser`:

```bash
sudo setfacl -m u:asiuser:r-x /home/dire
```
*(Si se desea que también acceda al interior del home de un directivo concreto como `/home/dire/usu1`, se aplica igualmente: `sudo setfacl -m u:asiuser:r-x /home/dire/usu1`).*

---

### 26. Compruebe, mediante getfacl, que está establecido el permiso extendido.

**Respuesta:**

```bash
getfacl /home/dire
```

Salida devuelta:
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
Asimismo, en la salida de `ls -ld /home/dire`, se comprueba que aparece un signo más (**`+`**) al final de los permisos tradicionales (`drwxr-x---+`), indicando la presencia de atributos ACL.

---

### 27. Compruebe empíricamente que el usuario usado en el apartado 25 tiene, efectivamente, acceso al directorio de los directivos.

**Respuesta:**

Iniciando sesión o ejecutando como `asiuser`:
```bash
cd /home/dire
ls -la /home/dire
```
El cambio de directorio y el listado de las entradas se completan satisfactoriamente sin mensaje de error.

---

# Sexta parte: Algo de shell script y de scripts de inicio de sesión

### 28. Cree un script (/usr/local/bin/change_umask) que establezca los permisos por defecto de los ficheros de forma distinta dependiendo del grupo principal al que pertenece el usuario:
* **Debemos asegurar que para los usuarios del grupo "dire" los permisos por defecto no permitan ningún tipo de acceso para "resto de usuarios" y permiso de lectura/ejecución para el grupo.**
* **Para el resto de los usuarios del sistema no deben hacerse cambios.**
* **El script debe mostrar un mensaje indicando que se han cambiado los permisos y a qué valor. Pista: se puede usar "groups" (que devuelve los grupos a los que pertenece un usuario).**

**Respuesta:**

#### Análisis de la máscara umask:
* Permiso deseado para ficheros base (máx 666): Propietario `rw-`, Grupo `r--`, Otros `---` $\rightarrow$ Permisos resultantes: `640`.
* Permiso deseado para directorios base (máx 777): Propietario `rwx`, Grupo `r-x`, Otros `---` $\rightarrow$ Permisos resultantes: `750`.
* Cálculo de la máscara:  
  $$777 - 750 = 027$$ (o $666 - 640 = 026$, redondeado a `027` para mantener ejecución de grupo en carpetas).

#### Creación del script `/usr/local/bin/change_umask`:
```bash
sudo tee /usr/local/bin/change_umask << 'EOF'
#!/bin/bash
# Obtenemos el grupo principal del usuario mediante id -gn
GRUPO_PRINCIPAL=$(id -gn)

if [ "$GRUPO_PRINCIPAL" = "dire" ]; then
    umask 027
    echo "Se ha modificado la mascara de permisos (umask) a 027 para el grupo dire."
fi
EOF

sudo chmod 755 /usr/local/bin/change_umask
```

---

### 29. Anote lo que ha hecho antes. Ejecute el script (teclee: /usr/local/bin/change_umask) para comprobar si funciona. ¿Se ha cambiado la máscara de permisos? ¿Por qué cree que no se ha cambiado?

**Respuesta:**

```bash
/usr/local/bin/change_umask
umask
```

* **¿Se ha cambiado la máscara?:** **No**, al ejecutar `umask` tras la llamada, el valor en la terminal sigue siendo el anterior (ej. `0002` o `0022`).
* **Por qué no se ha cambiado:**  
  Al invocar el script directamente por su ruta (`/usr/local/bin/change_umask`), la shell del sistema realiza un `fork()` y `exec()`, creando un nuevo **proceso hijo** (una subshell) para ejecutar el script. La orden `umask 027` modificó la máscara de dicho proceso hijo; al terminar el script, el proceso hijo muere y el proceso padre (nuestra shell interactiva) mantiene intacta su propia tabla de atributos y su máscara original.

---

### 30. Si se deseara que se ejecutara ese script para que nos aseguráramos de que todos los usuarios tienen correctamente establecida la máscara de permisos ¿qué tendríamos que modificar? (Pista: Busque esa información en el tema 2).

**Respuesta:**

Para que el cambio se aplique a todos los usuarios del sistema al iniciar sesión, se debe incluir la llamada al script en el fichero de configuración global de inicio de sesión:
* **/etc/profile** (para *login shells*).
* **/etc/bash.bashrc** (para *non-login shells* interactivas).

---

### 31. Incorpore en los ficheros de configuración del apartado anterior, una llamada al script que realizó en el apartado 26. Inicie sesión como un usuario del grupo "dire" y comprobará que no funciona, a pesar de que se muestra el mensaje de que se ha modificado la máscara de permisos. Ello es debido a que ocurre algo parecido a lo que sucedía en el apartado 27.

**Respuesta:**

Si en `/etc/profile` simplemente se escribe:
```bash
/usr/local/bin/change_umask
```
Al hacer login con `usu1`, aparece por pantalla el mensaje informativo de que se ha modificado la máscara, pero al teclear inmediatamente `umask` en la terminal, se comprueba que **sigue sin estar establecida a 027**.  
Ocurre exactamente lo mismo: `/etc/profile` lanzó el script como un binario en un proceso hijo independiente, sin alterar la memoria de la shell principal que toma el control del usuario.

---

### 32. Para evitar lo que sucede en el apartado anterior, debe invocarse el script de esta forma: ". /usr/local/bin/change_umask". De esta forma, el script se incorporará al fichero de configuración como si fuera un "include" lo que permitirá que la máscara de permisos se configure correctamente. Compruebe que ahora funcione correctamente. Describa lo sucedido.

**Respuesta:**

#### 1. Configuración correcta en `/etc/profile`:
```bash
sudo tee -a /etc/profile << 'EOF'
if [ -f /usr/local/bin/change_umask ]; then
    . /usr/local/bin/change_umask
fi
EOF
```

#### 2. Comprobación:
Se inicia sesión como `usu1` y se verifica la máscara activa:
```bash
su - usu1
umask
```
La salida devuelve efectivamente **`0027`**.

#### 3. Descripción de lo sucedido:
El operador punto (`.`) o comando integrado `source` le indica a la shell que **no debe realizar un `fork()`**, sino leer, interpretar y ejecutar cada una de las líneas del archivo directamente sobre el contexto y espacio de memoria del **proceso actual**. Por tanto, la orden interna `umask 027` se aplica sobre la propia shell interactiva del usuario directivo, haciendo efectiva la configuración de seguridad requerida.