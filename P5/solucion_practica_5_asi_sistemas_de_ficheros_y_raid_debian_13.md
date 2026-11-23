# Administración de Sistemas de Información - Estudio Experimental Práctica 5
## Sistemas de ficheros y RAID (Resuelto para Debian 13)

> **Nota previa sobre herramientas en Debian 13:** Antes de comenzar con las partes correspondientes a LVM, RAID y Cuotas, es necesario actualizar los repositorios e instalar los paquetes requeridos como `root`:
> ```bash
> apt update
> apt install lvm2 mdadm quota
> ```

---

### Primera parte: Manejo básico del sistema de ficheros

**1. Inicie sesión directamente como "root" y teclee lo siguiente `ls -la /dev/sd*`. ¿Qué información se muestra? A la vista de lo observado, ¿puede indicar cuántos discos ha reconocido el SO y, de estos, el número de particiones de cada uno de ellos? Teniendo en cuenta que el disco que alberga el SO está particionado ¿Cuál cree que es el disco que alberga el SO? Anote el nombre del dispositivo de los discos. AVISE AL PROFESOR AHORA.**

*   **Respuesta:**
    *   **Información mostrada:** Se listan los ficheros especiales de bloque (identificados por la letra `b` al inicio de los permisos) correspondientes a los discos duros SATA/SCSI y sus particiones detectadas por el kernel. También muestra sus números *major* y *minor*.
    *   **Discos reconocidos:** El sistema ha reconocido **5 discos**: `/dev/sda`, `/dev/sdb`, `/dev/sdc`, `/dev/sdd` y `/dev/sde`.
    *   **Número de particiones:** El disco `/dev/sda` tiene particiones asociadas (por ejemplo, `/dev/sda1`, `/dev/sda2`, etc., típicamente 2 o 3 particiones en una instalación estándar de Debian 13 con partición EFI/boot, raíz y swap). Los otros 4 discos (`/dev/sdb`, `/dev/sdc`, `/dev/sdd`, `/dev/sde`) no tienen ningún número detrás, lo que indica que tienen **0 particiones** (están vacíos/sin particionar).
    *   **Disco del SO:** El disco que alberga el sistema operativo es **`/dev/sda`**, ya que es el único que se encuentra particionado.
    *   **Nombres de los dispositivos:**
        *   Disco del SO: `/dev/sda`
        *   Discos adicionales (4 discos de ~500 MiB): `/dev/sdb`, `/dev/sdc`, `/dev/sdd`, `/dev/sde`.

---

**2. Mediante `fdisk` se puede mostrar fácilmente las particiones que tiene un disco. Use la opción `-l` para que el programa simplemente muestre las particiones (en vez de intentar editarlas). Use `fdisk -l` para mostrar la información de partición de su disco que tiene el SO ¿Cuántas y de qué tipo son las particiones mostradas?**

*   **Respuesta:**
    *   **Comando:**
        ```bash
        fdisk -l /dev/sda
        ```
    *   **Análisis de la salida:** En una instalación típica de Debian 13 (con tabla de particiones GPT o DOS/MBR), se muestran generalmente **2 o 3 particiones** (según si es arranque BIOS o UEFI):
        *   Si es tabla DOS (MBR): suele haber una partición primaria (`/dev/sda1`) de tipo **`Linux`** (ID `83`) para el sistema raíz (`/`) y una partición extendida (`/dev/sda2`, tipo `Extended`) que contiene una unidad lógica (`/dev/sda5`) de tipo **`Linux swap / Solaris`** (ID `82`).
        *   Si es tabla GPT (UEFI): suele haber `/dev/sda1` de tipo **`EFI System`**, `/dev/sda2` de tipo **`Linux filesystem`** (raíz `/`) y `/dev/sda3` de tipo **`Linux swap`**.

---

**3. Usaremos otro programa (`cfdisk`) para crear particiones en los discos adicionales (los que no tienen el SO). Úselo (seleccione la opción "gpt") y cree, en cada disco adicional, una única partición primaria que sea del tipo "Linux" y que ocupe el disco completo. Una vez que haya terminado con los cuatro discos, teclee nuevamente `ls -la /dev/sd*` ¿qué diferencias hay respecto a lo que ocurrió anteriormente? ¿Por qué cree que ahora aparecen más ficheros de dispositivo?**

*   **Respuesta:**
    *   **Comandos ejecutados:**
        ```bash
        cfdisk /dev/sdb
        cfdisk /dev/sdc
        cfdisk /dev/sdd
        cfdisk /dev/sde
        ```
        *(En cada uno: seleccionar `gpt` -> `[ New ]` -> aceptar tamaño máximo por defecto -> confirmar tipo `Linux filesystem` -> `[ Write ]` -> escribir `yes` -> `[ Quit ]`).*
    *   **Diferencias observadas al ejecutar `ls -la /dev/sd*`:** Ahora aparecen cuatro nuevos ficheros de dispositivo de bloque: `/dev/sdb1`, `/dev/sdc1`, `/dev/sdd1` y `/dev/sde1`.
    *   **Motivo:** Al escribir la tabla de particiones GPT en cada disco, el kernel de Linux relee dicha tabla y genera dinámicamente en `/dev` los nodos/ficheros especiales de bloque correspondientes a cada nueva partición creada para poder acceder a ellas individualmente.

---

**4. Ahora ya tiene creadas las particiones, pero cada una de ellas sigue siendo un conjunto de bytes "en crudo". Es necesario crear en ellos un sistema de ficheros. Use `mkfs` para crear un sistema de ficheros de tipo "ext4" en la primera (y única) partición del primer disco `sdc` y para que le muestre información "extendida". ¿Qué ha tecleado? Observe la salida del programa: ¿Cuántos bloques tiene el sistema de ficheros? ¿Cuántos bytes ocupa cada bloque? En la salida del programa se le indica, aunque no con ese nombre, el número máximo de ficheros que puede tener el sistema de ficheros que acaba de crear ¿qué valor ha tenido que buscar? Anote el número máximo de ficheros que puede tener.**

*   **Respuesta:**
    *   **Comando tecleado:**
        ```bash
        mkfs -t ext4 -v /dev/sdc1
        ```
        *(Nota: La opción `-v` activa el modo verbose/extendido).*
    *   **Datos de la salida (para un disco de ~500 MiB, es decir, ~499 MiB útiles tras la cabecera GPT):**
        *   **Número de bloques:** Aproximadamente **`510976` bloques** (de 1 KiB) o **`127744` bloques** (si se formatea con bloques de 4 KiB; en particiones menores a 512 MiB, `mkfs.ext4` suele asignar por defecto bloques de **1024 bytes**).
        *   **Bytes por bloque:** **`1024` bytes** (`Block size=1024` en la salida).
        *   **Valor buscado para el número máximo de ficheros:** El número de **inodos** (`inodes`), ya que cada fichero o directorio en `ext4` consume exactamente un inodo.
        *   **Número máximo de ficheros (inodos):** Aproximadamente **`128016` inodos** (varía ligeramente según el tamaño exacto en sectores de la partición de 500 MiB).

---

**5. Una de las características que tiene este sistema de ficheros es que se reserva un espacio del disco para el superusuario, de forma que ese espacio solo lo puede usar "root" y no los usuarios normales (lo que, en definitiva, hace que el espacio disponible para los usuarios normales sea más pequeño). Monte el sistema de ficheros que acaba de crear en `/home`. Mediante `df -k` (Nota: puede que la salida sea más legible si teclea `df -k /home`) compruebe el espacio disponible que le muestra el programa. Dicho espacio es el disponible para los usuarios y, por tanto, no incluye el espacio reservado para el superusuario. A la vista de lo observado en este apartado, ¿qué porcentaje del espacio total del disco se ha reservado para el superusuario?**

*   **Respuesta:**
    *   **Comandos:**
        ```bash
        mount /dev/sdc1 /home
        df -k /home
        ```
    *   **Cálculo y porcentaje reservado:**
        En la salida de `df -k /home` se observan las columnas `1K-blocks` (bloques totales útiles), `Used` (usados por metadatos del sistema de ficheros) y `Available` (disponibles para usuarios normales).
        Si hacemos la diferencia: $\text{Reservado} = \text{1K-blocks} - \text{Used} - \text{Available}$, y calculamos $(\text{Reservado} / \text{1K-blocks}) \times 100$, obtenemos exactamente el **`5%`**.
        *(Por defecto, `mkfs.ext4` reserva el 5% de los bloques del sistema de ficheros para el usuario `root` y procesos críticos del sistema).*

---

**6. Intente (en otro tty) iniciar sesión como usuario "normal". Comprobará que puede hacerlo, pero se le muestra un error que le indica que el `$HOME` del usuario no existe ¿Por qué sucede esto? Cierre la sesión de usuario normal.**

*   **Respuesta:**
    *   **Motivo:** Al montar el nuevo sistema de ficheros (`/dev/sdc1`) sobre el punto de montaje `/home`, el contenido original del directorio `/home` (que residía en la partición raíz `/` y contenía la carpeta personal del usuario normal, por ejemplo `/home/asiuser`) ha quedado **oculto/tapado** temporalmente. Como el nuevo sistema de ficheros está vacío (solo contiene `lost+found`), el sistema no encuentra el directorio `$HOME` del usuario y le asigna `/` como directorio de trabajo inicial.
    *   **Cierre de sesión:** Teclear `exit` o pulsar `Ctrl+D`.

---

**7. Desmonte el sistema de ficheros y muestre el contenido de `/home` ¿qué ocurre?**

*   **Respuesta:**
    *   **Comandos:**
        ```bash
        umount /home
        ls -la /home
        ```
    *   **¿Qué ocurre?:** Al desmontar `/dev/sdc1`, vuelve a ser visible el contenido original que estaba almacenado en el directorio `/home` del sistema de ficheros raíz (`/`), reapareciendo el directorio personal del usuario normal intacto.

---

### Segunda parte: Uso de LVM

**8. Nota: En este paso se indica lo que se va a hacer en la segunda parte, pero en este momento no tiene que hacer nada, léalo entero y siga con el paso 9.**

*   **Respuesta:** *(Paso informativo de lectura).*

---

**9. Usando `pvcreate` cree los cuatro PV para los cuatro discos nuevos del sistema (use la primera partición de cada disco).**

*   **Respuesta:**
    ```bash
    pvcreate /dev/sdb1 /dev/sdc1 /dev/sdd1 /dev/sde1
    ```
    *(Nota: Si pide confirmación para limpiar la firma `ext4` de `/dev/sdc1` creada en la parte 1, responder `y`).*

---

**10. Cree un único grupo de volúmenes (llámele "Lab") que agrupe los 4 PV recién creados. Use `vgdisplay` para averiguar el tamaño del PE del grupo recién creado y anótelo. Compruebe que el tamaño del grupo se corresponde (aproximadamente) con el tamaño de los 4 discos agregados y que el grupo está formado, efectivamente, por 4 PV. Indique de dónde (qué ha mirado, qué campo) ha sacado esa información.**

*   **Respuesta:**
    *   **Comandos:**
        ```bash
        vgcreate Lab /dev/sdb1 /dev/sdc1 /dev/sdd1 /dev/sde1
        vgdisplay Lab
        ```
    *   **Información obtenida en `vgdisplay Lab`:**
        *   **Tamaño del PE (Physical Extent):** **`4.00 MiB`**, consultado en el campo **`PE Size`**.
        *   **Tamaño total del grupo:** Aproximadamente **`1.94 GiB`** (o ~`1984.00 MiB`, equivalente a $4 \times 496\text{ MiB}$ útiles tras restar las cabeceras GPT y metadatos de LVM), consultado en el campo **`VG Size`**.
        *   **Número de PV que lo forman:** **`4`**, consultado en los campos **`Cur PV`** y **`Act PV`**.

---

**11. ¿Se ha creado algún fichero de dispositivo con el nombre de "Lab" en `/dev`?**

*   **Respuesta:**
    *   **No.** Si ejecutamos `ls -la /dev/Lab`, el fichero o directorio no existe todavía.

---

**12 y 13. Explicación conceptual sobre VG y LV.**

*   **Respuesta:** *(Pasos informativos de lectura).*

---

**14. Cree, mediante `lvcreate`, un nuevo LV de tamaño 750MB y asígnele el nombre "lv1".**

*   **Respuesta:**
    ```bash
    lvcreate -L 750M -n lv1 Lab
    ```
    *(Nota: En LVM moderno de Debian 13, `-L 750M` crea un volumen de 750 MiB exactos).*

---

**15. Mediante `lvdisplay` puede ver los datos asociados al volumen que se ha creado. Compruebe si existe el fichero de dispositivo indicado y anótelo.**

*   **Respuesta:**
    *   **Comandos:**
        ```bash
        lvdisplay /dev/Lab/lv1
        ls -la /dev/Lab/lv1
        ```
    *   **Fichero de dispositivo indicado:** En el campo **`LV Path`** de `lvdisplay` aparece **`/dev/Lab/lv1`** (que en Debian es un enlace simbólico hacia `/dev/mapper/Lab-lv1`, el cual apunta al dispositivo de bloque `/dev/dm-0`). Comprobando con `ls -la`, el fichero existe.

---

**16. Use `pvdisplay` para mostrar el uso de cada uno de los PV (quizá tenga que usar `more` para paginar la salida). ¿Se han usado (total o parcialmente) todos los PV al crear el LV? Anote los PV que no se han usado para nada e indique en qué se ha fijado para hacer esa afirmación.**

*   **Respuesta:**
    *   **Comando:**
        ```bash
        pvdisplay | more
        ```
    *   **¿Se han usado todos los PV?:** **No.** Solo se han utilizado los dos primeros volúmenes físicos (`/dev/sdb1` al 100% con ~496 MiB / 124 PEs, y `/dev/sdc1` parcialmente con ~254 MiB / 64 PEs).
    *   **PV no usados:** **`/dev/sdd1`** y **`/dev/sde1`**.
    *   **Campo observado:** En `pvdisplay`, para `/dev/sdd1` y `/dev/sde1`, el campo **`Allocated PE`** vale **`0`** (y **`Free PE`** es igual a **`Total PE`**).

---

**17. Use `lvcreate` para crear un nuevo LV, esta vez de 450MiB, y asígnele el nombre "lv2". Compruebe ahora si hay algún PV que no se esté usando y anote la respuesta.**

*   **Respuesta:**
    *   **Comandos:**
        ```bash
        lvcreate -L 450M -n lv2 Lab
        pvdisplay | more
        ```
    *   **PV sin usar:** Ahora el único PV que no se está usando (`Allocated PE` igual a `0`) es **`/dev/sde1`**, ya que `lv2` ocupa el espacio libre restante de `/dev/sdc1` (~242 MiB) y parte de `/dev/sdd1` (~208 MiB).

---

**18. Cree dos puntos de montaje (`/pto1` y `/pto2`), cree un sistema de ficheros tipo ext4 en los dos LV nuevos y móntelos.**

*   **Respuesta:**
    ```bash
    mkdir /pto1 /pto2
    mkfs -t ext4 /dev/Lab/lv1
    mkfs -t ext4 /dev/Lab/lv2
    mount /dev/Lab/lv1 /pto1
    mount /dev/Lab/lv2 /pto2
    ```

---

**19. Compruebe (con `df -k /pto1 /pto2`) que los sistemas de ficheros tienen el tamaño esperado. NOTA: Recuerde que la opción "h" de df y otros programas permite una salida más fácil de leer. Anote la salida.**

*   **Respuesta:**
    *   **Comando:**
        ```bash
        df -kh /pto1 /pto2
        ```
    *   **Salida aproximada:**
        ```text
        Filesystem           Size  Used Avail Use% Mounted on
        /dev/mapper/Lab-lv1  720M   24K  668M   1% /pto1
        /dev/mapper/Lab-lv2  426M   24K  393M   1% /pto2
        ```
        *(Los tamaños útiles son ligeramente inferiores a 750 MiB y 450 MiB debido a los metadatos de `ext4` y el 5% reservado para `root`).*

---

**20. Cree bajo `/pto1` algún fichero.**

*   **Respuesta:**
    ```bash
    touch /pto1/fichero_prueba.txt
    ```

---

**21. En el paso 17 pudo comprobar que existe un PV que no se está usando. Use `vgreduce` para intentar eliminar ese PV del grupo. Compruebe que la salida de `vgdisplay` muestra que el VG está formado por solo tres discos.**

*   **Respuesta:**
    *   **Comandos:**
        ```bash
        vgreduce Lab /dev/sde1
        vgdisplay Lab
        ```
    *   **Comprobación:** En `vgdisplay Lab`, los campos `Cur PV` y `Act PV` muestran ahora **`3`** (y `VG Size` se ha reducido a ~`1.45 GiB`).

---

**22. Ahora vamos a intentar eliminar el segundo LV que hemos creado (`lv2`). Para eso se usará `lvremove` (mire las transparencias para un ejemplo). Inténtelo. ¿Ha funcionado? ¿Qué ocurre?**

*   **Respuesta:**
    *   **Comando:**
        ```bash
        lvremove /dev/Lab/lv2
        ```
    *   **¿Ha funcionado? ¿Qué ocurre?:** **No funciona.** El comando devuelve un error indicando que el volumen lógico está en uso/abierto (`Logical volume Lab/lv2 contains a filesystem in use`), ya que actualmente se encuentra montado en `/pto2`.

---

**23. Desmonte el sistema de ficheros de `lv2` y elimine ese LV.**

*   **Respuesta:**
    ```bash
    umount /pto2
    lvremove /dev/Lab/lv2
    ```
    *(Confirmar con `y` cuando pregunte).*

---

**24. Una vez que haya eliminado ese LV, vamos a intentar quitar el PV correspondiente con la primera partición del primer disco adicional del VG (use `vgreduce`). Inténtelo, ¿puede? (la respuesta debe ser "no"). ¿Por qué no se puede?**

*   **Respuesta:**
    *   **Comando:**
        ```bash
        vgreduce Lab /dev/sdb1
        ```
    *   **¿Se puede?:** **No.** Devuelve el error `Physical volume "/dev/sdb1" still in use`.
    *   **Motivo:** Porque `/dev/sdb1` contiene extensiones físicas (PE) asignadas al volumen lógico `lv1` (que sigue activo y ocupando 750 MiB distribuidos entre `/dev/sdb1` y `/dev/sdc1`).

---

**25. Use `pvmove` para liberar ese disco e inténtelo de nuevo (intente quitar el disco del VG). Si funciona indique qué ha tecleado y dónde cree que han ido a parar los datos del disco que se ha eliminado.**

*   **Respuesta:**
    *   **Comandos tecleados:**
        ```bash
        pvmove /dev/sdb1
        vgreduce Lab /dev/sdb1
        ```
    *   **Destino de los datos:** Los bloques/PE de `lv1` que estaban almacenados en `/dev/sdb1` se han migrado en caliente (sin desmontar `/pto1`) al espacio libre disponible en **`/dev/sdd1`** (y al resto libre de `/dev/sdc1`), que son los otros volúmenes físicos activos dentro del grupo `Lab`.

---

**26. ¿Sigue funcionando el sistema de ficheros instalado en `/pto1`? (intente crear un directorio dentro de `/pto1`).**

*   **Respuesta:**
    *   **Comando:**
        ```bash
        mkdir /pto1/dir_prueba
        ls -la /pto1
        ```
    *   **Resultado:** **Sí, sigue funcionando perfectamente** y sin interrupciones, ya que LVM realiza la reubicación de bloques de forma transparente para el sistema de ficheros.

---

**27. Cree (usando `dd`) un fichero de 100MiB en `/pto1`. Compruebe que se ha creado correctamente el fichero en el sistema de ficheros deseado.**

*   **Respuesta:**
    ```bash
    dd if=/dev/zero of=/pto1/fichero100M bs=1M count=100
    ls -lh /pto1/fichero100M
    df -h /pto1
    ```

---

**28. Mediante `echo "offline" > /sys/block/sdd/device/state` provocará que el sistema operativo ponga "offline" el disco `sdd`. ¿Qué cree que le pasará al sistema de ficheros de `/pto1` si hace eso?**

*   **Respuesta:**
    *   Como `lv1` (montado en `/pto1`) está formado por la concatenación/agregación lineal sin redundancia de `/dev/sdc1` y `/dev/sdd1`, al desconectar `/dev/sdd` desaparecerá una parte física esencial del volumen lógico. El sistema de ficheros `ext4` perderá acceso a sus bloques/inodos almacenados en `sdd1`, producirá errores graves de entrada/salida (I/O error) y el kernel lo remontará en modo **solo lectura (`ro`)** o quedará corrupto/inaccesible.

---

**29. Compruébelo. Pare el disco `sdd` y acceda a continuación a `/pto1`. Haga un `ls -la`. ¿Sigue funcionando? Intente ahora hacer una copia del fichero de 100MiB que creó antes (establezca como destino el propio directorio `/pto1`). Anote lo que ocurre. Nota: Puede que transcurran algunos segundos antes de que pase "algo".**

*   **Respuesta:**
    *   **Comandos:**
        ```bash
        echo "offline" > /sys/block/sdd/device/state
        ls -la /pto1
        cp /pto1/fichero100M /pto1/copia100M
        ```
    *   **¿Qué ocurre?:**
        *   El comando `ls -la /pto1` puede funcionar inicialmente porque los metadatos del directorio raíz están en `sdc1` o cacheados en memoria RAM.
        *   Al intentar copiar `fichero100M` con `cp`, el sistema intenta leer/escribir bloques ubicados físicamente en `sdd1`. Tras unos segundos de espera (*timeout* de I/O), aparecen mensajes de error de entrada/salida (`Input/output error`) y el sistema de ficheros `ext4` pasa a modo **solo lectura** (`Read-only file system`) para protegerse, abortando la copia.

---

**30. Desmonte `/pto1` y, a continuación, intente montarlo de nuevo ¿ocurre algo? ¿es posible montarlo?**

*   **Respuesta:**
    *   **Comandos:**
        ```bash
        umount /pto1
        mount /dev/Lab/lv1 /pto1
        ```
    *   **¿Ocurre algo? ¿Es posible montarlo?:** **No es posible montarlo.** El comando `mount` falla (indicando error de lectura del superbloque o dispositivo inexistente/corrupto, ej. `can't read superblock on /dev/mapper/Lab-lv1`), ya que el volumen lógico `lv1` está incompleto al faltar uno de sus volúmenes físicos subyacentes (`sdd1`).

---

**31. Con esto terminamos la parte 2. A fin de realizar la siguiente parte es conveniente que dejemos todos los discos activos y, además, para no interferir con la configuración del RAID, eliminaremos la configuración de LVM de todos los discos:**
*   **a. Reactive el disco que hemos desactivado: `echo "running" > /sys/block/sdd/device/state`.**
*   **b. Elimine el último LV.**
*   **c. Elimine el VG.**
*   **d. Elimine (`pvremove`) todos los PV (puede que sea necesario usar la opción "force").**

*   **Respuesta:**
    ```bash
    # a. Reactivar el disco sdd
    echo "running" > /sys/block/sdd/device/state

    # b. Eliminar el último LV (lv1)
    lvremove -f /dev/Lab/lv1

    # c. Eliminar el VG (Lab)
    vgremove Lab

    # d. Eliminar todos los PV (incluyendo los que se sacaron del VG antes)
    pvremove -f /dev/sdb1 /dev/sdc1 /dev/sdd1 /dev/sde1
    ```

---

### Tercera parte: Uso de `mdadm`

**32. Teniendo en cuenta que vamos a usar 4 discos de 500MiB y que en esta parte de la práctica vamos a usar un RAID5. ¿Cuál cree que será el tamaño del disco resultante?**

*   **Respuesta:**
    *   En un **RAID 5** formado por $N$ discos de tamaño $T$, se pierde el equivalente a 1 disco para almacenar la paridad distribuida, por lo que la capacidad útil es:
        $$\text{Capacidad útil} = (N - 1) \times T = (4 - 1) \times 500\text{ MiB} = \mathbf{1500\text{ MiB}} \text{ (aprox. } 1.46\text{ GiB)}$$
        *(En la práctica será ligeramente inferior, alrededor de **1496 MiB**, debido al espacio que reserva `mdadm` en cada partición para su superbloque de metadatos).*

---

**33. Usando `mdadm` cree un RAID5 que esté formado por la primera partición de los 4 discos que ha creado. El nombre del dispositivo raid debe ser `/dev/md0`.**

*   **Respuesta:**
    ```bash
    mdadm --create /dev/md0 --level=5 --raid-devices=4 /dev/sdb1 /dev/sdc1 /dev/sdd1 /dev/sde1
    ```

---

**34. Cree un sistema de ficheros en el RAID. Nota: si teclea `cat /proc/mdstat` podrá ver el estado del RAID.**

*   **Respuesta:**
    ```bash
    mkfs -t ext4 /dev/md0
    cat /proc/mdstat
    ```

---

**35. Monte el recién creado sistema de ficheros en `/pto1`. ¿Qué capacidad tiene? ¿Se corresponde con la que calculó antes?**

*   **Respuesta:**
    *   **Comandos:**
        ```bash
        mount /dev/md0 /pto1
        df -h /pto1
        ```
    *   **Capacidad y correspondencia:** `df -h /pto1` muestra una capacidad total de aproximadamente **`1.5G`** (o **`1454M`** útiles tras metadatos de `ext4`), lo cual **sí se corresponde** con los $3 \times 500\text{ MiB} = 1500\text{ MiB}$ calculados teóricamente.

---

**36 y 37. Registro de configuración del RAID y actualización de initramfs.**

*   **Respuesta:**
    ```bash
    mdadm --detail --scan
    mdadm --detail --scan >> /etc/mdadm/mdadm.conf
    update-initramfs -u
    ```

---

**38, 39 y 40. Reinicio del equipo, inicio de sesión como `root`, comprobación de montaje y creación de fichero de 100MiB.**

*   **Respuesta:**
    ```bash
    reboot
    # (Tras reiniciar e iniciar sesión como root)
    mount /dev/md0 /pto1
    dd if=/dev/zero of=/pto1/fichero100M bs=1M count=100
    ```
    *(Nota: Si se añadió previamente a `/etc/fstab`, ya estará montado; si no, `mount /dev/md0 /pto1` lo monta directamente).*

---

**41. Además de ver el contenido de `/proc/mdstat`, existen más formas de monitorizar lo que ocurre en el RAID que hemos creado. Una de esas formas es teclear: `mdadm --detail /dev/md0`. Ejecute ese comando y anote el estado general del RAID (campo "State", justo debajo de "Update Time") y el estado de los 4 discos.**

*   **Respuesta:**
    *   **Comando:**
        ```bash
        mdadm --detail /dev/md0
        ```
    *   **Estado general (`State`):** **`clean`** (limpio, operativo al 100% y sincronizado).
    *   **Estado de los 4 discos:** Los cuatro dispositivos (`/dev/sdb1`, `/dev/sdc1`, `/dev/sdd1`, `/dev/sde1`) aparecen en la tabla inferior con estado **`active sync`**.

---

**42. Lea este apartado completamente antes de hacer nada, dado que todo va a suceder muy rápido. Pare (de forma similar a lo que hizo en el paso 28) el primer disco adicional. Usando `mdadm --detail /dev/md0` vuelva a mostrar información sobre el RAID. ¿En qué estado está? (debe indicarse un fallo, si no es así espere unos segundos o fuerce a que exista actividad en el sistema de ficheros) ¿Cómo está marcado el disco que ha fallado? ¿Cuántos dispositivos activos/trabajando/fallidos/reserva tiene?**

*   **Respuesta:**
    *   **Comandos:**
        ```bash
        echo "offline" > /sys/block/sdb/device/state
        touch /pto1/forzar_escritura
        mdadm --detail /dev/md0
        ```
    *   **Estado del RAID (`State`):** **`clean, degraded`** (limpio pero degradado).
    *   **Marca del disco fallido (`/dev/sdb1`):** Aparece marcado como **`faulty`** (fallido).
    *   **Contadores de dispositivos:**
        *   `Active Devices` (Activos): **3**
        *   `Working Devices` (Trabajando): **3**
        *   `Failed Devices` (Fallidos): **1**
        *   `Spare Devices` (Reserva): **0**

---

**43. A continuación vamos a eliminar el disco que ha fallado del RAID. Eso lo puede hacer mediante `mdadm` tal y como se mostraba en el anexo del estudio teórico. Hágalo.**

*   **Respuesta:**
    ```bash
    mdadm --manage /dev/md0 --remove /dev/sdb1
    ```
    *(También válido de forma abreviada: `mdadm /dev/md0 -r /dev/sdb1`).*

---

**44. Compruebe que el sistema de ficheros sigue funcionando (ej: copiar algo ahí), a pesar del fallo en el disco.**

*   **Respuesta:**
    ```bash
    cp /pto1/fichero100M /pto1/copia_en_degradado
    ls -lh /pto1
    ```
    *(La operación finaliza sin ningún error de E/S).*

---

**45. ¿Por qué ahora el sistema de ficheros sigue funcionando y no sucedía eso en la parte anterior de la práctica?**

*   **Respuesta:**
    *   Porque en la parte anterior usábamos **LVM en modo lineal (sin redundancia)**, donde la pérdida de un disco implica la pérdida irrecuperable de los bloques almacenados en él. En cambio, **RAID 5** almacena información de **paridad distribuida** entre todos los discos del conjunto; gracias a esa paridad y a los 3 discos restantes, el controlador RAID calcula al vuelo los datos que faltan del disco caído y mantiene el sistema de ficheros completamente operativo.

---

**46. Una vez que ha eliminado el disco fallido. ¿Qué cambios hay en la salida de `mdadm --detail /dev/md0`? ¿Cuál es el estado ("state" en la salida del programa) del multidispositivo en este momento?**

*   **Respuesta:**
    *   **Cambios en la salida:** El disco `/dev/sdb1` desaparece de la lista de dispositivos inferiores (apareciendo en su lugar una línea que indica `0 - 0 - removed`). El contador `Failed Devices` pasa de `1` a **`0`**.
    *   **Estado (`State`):** Sigue siendo **`clean, degraded`**, ya que el array continúa operando con 3 de los 4 discos configurados (sin tolerancia a un segundo fallo).

---

**47. Active nuevamente el primer disco adicional (ver paso 31.a) y, hecho esto, añádalo al RAID.**

*   **Respuesta:**
    ```bash
    echo "running" > /sys/block/sdb/device/state
    mdadm --manage /dev/md0 --add /dev/sdb1
    ```

---

**48. Compruebe la salida de `mdadm --detail /dev/md0` ¿Cuál es el estado según `mdadm`? ¿Cuántos discos activos y de reserva hay? ¿Qué dice el programa respecto al disco que se acaba de añadir al RAID?**

*   **Respuesta:**
    *   **Comando:**
        ```bash
        mdadm --detail /dev/md0
        ```
    *   **Estado (`State`):** **`clean, degraded, recovering`** (mientras se reconstruye) y posteriormente **`clean`** al finalizar.
    *   **Discos activos y de reserva (durante la reconstrucción):**
        *   `Active Devices`: **3**
        *   `Working Devices`: **4**
        *   `Spare Devices`: **1**
    *   **Estado del disco recién añadido (`/dev/sdb1`):** Aparece marcado como **`spare rebuilding`** (disco de reserva reconstruyéndose). Una vez finaliza la reconstrucción al 100%, pasa a **`active sync`**, `Active Devices` pasa a **4** y `Spare Devices` a **0**.

---

**49 y 50. Sincronización y prueba de fallo doble (opcional).**

*   **Respuesta:** *(En RAID 5, si fallan 2 discos simultáneamente se supera la tolerancia de 1 disco, el RAID queda inactivo/corrupto y se pierden los datos).*

---

**51. Antes de seguir a la cuarta parte asegúrese de lo siguiente: Pare el RAID mediante `mdadm --stop /dev/md0`, vuelva a arrancar los discos parados y elimine todo resto de `mdadm` con `mdadm --zero-superblock <partición_dispositivo>` para cada uno de los cuatro discos que ha usado.**

*   **Respuesta:**
    ```bash
    umount /pto1
    mdadm --stop /dev/md0
    mdadm --zero-superblock /dev/sdb1 /dev/sdc1 /dev/sdd1 /dev/sde1
    ```
    *(Nota: Es recomendable también comentar o eliminar la línea `ARRAY /dev/md/0 ...` añadida en `/etc/mdadm/mdadm.conf` y ejecutar `update-initramfs -u` para que no intente ensamblarlo en futuros reinicios).*

---

### Extra time. Cuarta parte: Moviendo `/home` a otra ubicación y estableciendo cuotas de disco

**52. Instale el sistema de cuotas de disco mediante `apt-get install quota`.**

*   **Respuesta:**
    ```bash
    apt update
    apt-get install quota
    ```

---

**53. Cree un sistema de ficheros `ext4` en `sdb1` y móntelo en `/pto1`.**

*   **Respuesta:**
    ```bash
    mkfs -t ext4 /dev/sdb1
    mount /dev/sdb1 /pto1
    ```

---

**54. Ahora mismo debemos tener montado en `/pto1` el nuevo sistema de ficheros y el directorio `/home` debe estar ubicado en el mismo sistema de ficheros que `/`. Esto lo puede comprobar (hágalo) mediante `df -k /home` y `df -k /pto1`. ¿Qué tiene que mirar para comprobar que está haciendo bien las cosas?**

*   **Respuesta:**
    *   **Comandos:**
        ```bash
        df -k /home
        df -k /pto1
        ```
    *   **Qué hay que mirar:** Hay que fijarse en la primera columna (**`Filesystem`**) y en la última (**`Mounted on`**):
        *   Para `/home`, `Filesystem` debe ser la partición raíz (ej. `/dev/sda1` o `/dev/sda2`) y `Mounted on` debe indicar `/`.
        *   Para `/pto1`, `Filesystem` debe ser **`/dev/sdb1`** y `Mounted on` debe indicar **`/pto1`**.

---

**55, 56 y 57. Preparación de ficheros de prueba con usuario normal (`FICH1`, enlace simbólico `FICH2` y enlace duro `FICH3`).**

*   **Respuesta:**
    ```bash
    # Iniciamos sesión como usuario normal (ej. asiuser) y en su $HOME:
    echo "Contenido de prueba" > FICH1
    ln -s FICH1 FICH2
    ln FICH1 FICH3
    ls -li
    ```
    *(Al ejecutar `ls -li`, se verifica que `FICH1` y `FICH3` comparten exactamente el mismo número de inodo en la primera columna y tienen contador de enlaces `2`, mientras que `FICH2` tiene un inodo distinto y tipo `l`).*

---

**58. Sitúese en `/home` y teclee (como superusuario): `cp -r . /pto1`. Con ello le estamos indicando al programa `cp` que copie recursivamente. Vaya al directorio `/pto1/asiuser` y haga `ls -la`. ¿Observa algún problema (en realidad hay dos: el propietario/grupo de los ficheros y también con los enlaces duros)? ¿Cuál es el problema? Borre el contenido de la COPIA (repito: de la COPIA, NO BORRE EL ORIGINAL, SOLO LA COPIA). AVISE AL PROFESOR.**

*   **Respuesta:**
    *   **Comandos:**
        ```bash
        cd /home
        cp -r . /pto1
        ls -lia /pto1/asiuser
        ```
    *   **Problemas observados:**
        1.  **Propietario y grupo:** Todos los ficheros y directorios copiados pertenecen ahora a **`root:root`** en lugar de `asiuser:asiuser`, porque `cp -r` sin `-p` asigna la propiedad al usuario que ejecuta la copia (`root`).
        2.  **Enlaces duros (y simbólicos):** `FICH1` y `FICH3` ya **no comparten el mismo número de inodo** (se han copiado como dos ficheros independientes ocupando el doble de espacio), y `FICH2` ha dejado de ser un enlace simbólico para convertirse en una copia regular del fichero al que apuntaba.
    *   **Borrado de la copia en `/pto1`:**
        ```bash
        rm -rf /pto1/asiuser
        ```

---

**59. En Internet, a veces se sugiere usar `cp -rp` para solventar el problema anterior. Pruébelo y comprobará que ahora sí se respetan los permisos y propietarios... pero... ¿Se conservan los enlaces simbólicos como tales? ¿Y los enlaces duros? Borre el contenido de la COPIA.**

*   **Respuesta:**
    *   **Comandos:**
        ```bash
        cd /home
        cp -rp . /pto1
        ls -lia /pto1/asiuser
        ```
    *   **¿Se conservan los enlaces simbólicos?:** **No.** Al usar `-r` (en lugar de `-R` o `-P`), `cp` sigue los enlaces simbólicos (*dereference*) y copia el contenido del fichero destino como un fichero regular.
    *   **¿Y los enlaces duros?:** **Tampoco.** `FICH1` y `FICH3` siguen teniendo inodos diferentes (se duplican los datos).
    *   **Borrado de la copia:**
        ```bash
        rm -rf /pto1/asiuser
        ```

---

**60. En vez de usar las alternativas anteriores, usaremos (sitúese en `/home`) `tar cfv - . | (cd /pto1; tar xf -)` para copiar el contenido del directorio `/home` correctamente. Compruebe que funciona. NOTA: Para este caso se podría haber usado `cp -a`.**

*   **Respuesta:**
    ```bash
    cd /home
    tar cfv - . | (cd /pto1; tar xf -)
    ls -lia /pto1/asiuser
    ```
    *(Comprobación: Se preservan los propietarios `asiuser:asiuser`, los permisos originales, el enlace simbólico `FICH2 -> FICH1` como tipo `l`, y `FICH1` y `FICH3` vuelven a compartir el **mismo número de inodo** como enlace duro).*

---

**61 y 62. Renombrado de `/home` a `/home.old`, montaje de `/dev/sdb1` en `/home` y configuración permanente en `/etc/fstab`.**

*   **Respuesta:**
    *   **Comandos realizados:**
        ```bash
        cd /
        mv /home /home.old
        mkdir /home
        umount /pto1
        mount /dev/sdb1 /home
        ```
    *   **Cambios en `/etc/fstab`:** Añadir al final del fichero `/etc/fstab` la siguiente línea (usando `/dev/sdb1` o su UUID con `defaults`):
        ```text
        /dev/sdb1   /home   ext4    defaults    0   2
        ```
    *   **Comprobación tras reiniciar (`reboot`):** Al iniciar sesión como usuario normal (`asiuser`), accede sin errores a su carpeta `/home/asiuser` con todos sus ficheros intactos.

---

**63. Las cuotas de disco se activan con `quotaon`, ¿qué ocurre si intenta activar las cuotas en `/home`? Solucione el problema activando la opción `usrquota` (o `quota`) al montar el sistema de ficheros, pero eso debe hacerlo sin desmontar previamente el sistema de ficheros (pista: opción `remount`).**

*   **Respuesta:**
    *   **¿Qué ocurre al ejecutar `quotaon /home`?:** Devuelve un error indicando que no encuentra los ficheros de cuotas o que el punto de montaje no tiene habilitado el soporte de cuotas (`quotaon: Mountpoint (or device) /home not found or has no quota enabled`).
    *   **Solución sin desmontar (`remount`):**
        ```bash
        mount -o remount,usrquota /home
        ```
        *(Nota: En `ext4` de Debian 13, la opción estándar es `usrquota`, equivalente a `quota`).*

---

**64 y 65. Comprobación con `quota -v`, inicialización con `quotacheck` y activación con `quotaon`.**

*   **Respuesta:**
    *   Al ejecutar `quotaon /home` antes de `quotacheck`, falla porque aún no existe el fichero índice `aquota.user` en `/home`.
    *   **Secuencia correcta:**
        ```bash
        quotacheck -cu /home
        quotaon /home
        quotaon -p /home
        ```
        *(La salida de `quotaon -p /home` muestra `user quota on /home (/dev/sdb1) is on`).*

---

**66. Compruebe la salida de `quota -v <usuario>` (donde `<usuario>` es el nombre del usuario normal de su sistema). ¿Cuántos ficheros tiene ese usuario? ¿Cuántos bytes ocupan? ¿Tiene alguna cuota ese usuario?**

*   **Respuesta:**
    *   **Comando:**
        ```bash
        quota -v asiuser
        ```
    *   **Análisis de la salida:**
        *   **Número de ficheros (`files`):** Muestra el total de inodos usados por el usuario (por ejemplo, `8` o `9`, contando su directorio `$HOME`, `.bashrc`, `.profile`, `.bash_logout`, `FICH1` y `FICH2`).
        *   **Espacio ocupado (`blocks`):** Muestra los bloques de 1 KiB ocupados (por ejemplo, `28` o `32` bloques = `28672` o `32768` bytes).
        *   **¿Tiene alguna cuota?:** **No**, los campos `quota` (límite *soft*) y `limit` (límite *hard*) aparecen a **`0`** tanto para bloques como para inodos (`files`).

---

**67. Establezca una cuota de uso de disco mediante `edquota`. Haga que la cuota de ficheros "soft" sea inferior (en una unidad) al uso actual de ficheros, que la cuota "hard" sea de un fichero más que el número actual de ficheros, que la cuota "soft" de uso de espacio en disco sea de 10MB o, lo que es lo mismo, 10000 bloques (o algo superior, si ya está ocupando el usuario ese espacio) y que la cuota "hard" de uso de espacio en disco sea igual a la cuota "soft". Indique qué pasos ha dado y cómo lo ha configurado (indique qué campos del fichero de cuota ha cambiado).**

*   **Respuesta:**
    *   **Comando ejecutado:**
        ```bash
        edquota -u asiuser
        ```
    *   **Configuración editada (suponiendo que el usuario tiene actualmente `8` ficheros/inodos y `28` bloques):**
        En el editor que se abre, se modifican los valores `soft` y `hard` de bloques e inodos de la siguiente manera:
        ```text
        Disk quotas for user asiuser (uid 1000):
          Filesystem                   blocks       soft       hard     inodes     soft     hard
          /dev/sdb1                        28      10000      10000          8        7        9
        ```
        *   **Campos cambiados:**
            *   `soft` (de `blocks`): puesto a **`10000`** (10000 bloques de 1 KiB $\approx$ 10 MB).
            *   `hard` (de `blocks`): puesto a **`10000`** (igual al *soft*).
            *   `soft` (de `inodes`): puesto a **`7`** ($8 - 1$, una unidad inferior al uso actual).
            *   `hard` (de `inodes`): puesto a **`9`** ($8 + 1$, una unidad superior al uso actual).
        *   Se guarda el fichero y se sale del editor.

---

**68. ¿Qué muestra ahora `quota -v` para el usuario normal de su sistema? Indíquelo detalladamente, explicando lo que cree que significa cada campo.**

*   **Respuesta:**
    *   **Comando:**
        ```bash
        quota -v asiuser
        ```
    *   **Salida y significado de cada campo:**
        ```text
        Disk quotas for user asiuser (uid 1000): 
             Filesystem  blocks   quota   limit   grace   files   quota   limit   grace
              /dev/sdb1      28   10000   10000               8*      7       9   6days
        ```
        *   `Filesystem`: Dispositivo/partición sobre la que se aplica la cuota (`/dev/sdb1`).
        *   `blocks`: Cantidad actual de espacio en disco ocupado por el usuario (en bloques de 1 KiB).
        *   `quota` (1ª aparición): Límite *soft* (blando) de espacio en disco (`10000` KiB). Se puede superar temporalmente durante el periodo de gracia.
        *   `limit` (1ª aparición): Límite *hard* (duro) de espacio en disco (`10000` KiB). Límite estricto infranqueable.
        *   `grace` (1ª aparición): Tiempo de gracia restante si se supera el límite *soft* de bloques (vacío porque no se ha superado).
        *   `files`: Cantidad actual de ficheros/directorios (inodos) propiedad del usuario (`8*`). El **asterisco (`*`)** indica que el usuario ha superado el límite *soft* de ficheros.
        *   `quota` (2ª aparición): Límite *soft* de número de ficheros (`7`).
        *   `limit` (2ª aparición): Límite *hard* de número de ficheros (`9`).
        *   `grace` (2ª aparición): Tiempo de gracia restante (`6days` / 7 días por defecto) antes de que el límite *soft* de ficheros pase a comportarse como un límite *hard*.

---

**69. Como usuario normal, compruebe si puede ver su propia cuota de usuario. NOTA: Si ve que no es posible, modifique los permisos del fichero que considere conveniente Y AVISE AL PROFESOR.**

*   **Respuesta:**
    *   Al ejecutar `quota -v` como usuario normal (`asiuser`), puede ocurrir que devuelva un error de permiso denegado (`Permission denied`) al intentar leer `/home/aquota.user` si este tiene permisos `-rw-------` (`600`) de `root`.
    *   **Solución (como `root`):** Dar permiso de lectura para otros usuarios al fichero `aquota.user`:
        ```bash
        chmod o+r /home/aquota.user
        ```
    *   Tras esto, `asiuser` ya puede ejecutar `quota -v` sin problemas.

---

**70. Pruebe, como usuario normal, a crear un fichero más, ¿puede? ¿y otro más?**

*   **Respuesta:**
    *   **Crear un primer fichero (`touch prueba1`):** **Sí puede.** El contador pasa de `8` a `9` ficheros (alcanzando el límite *hard* de `9`). Muestra un aviso por pantalla (`sdb1: warning, user file quota exceeded.`), pero crea el fichero.
    *   **Crear un segundo fichero (`touch prueba2`):** **No puede.** El sistema bloquea la operación y muestra el error: `touch: cannot touch 'prueba2': Disk quota exceeded`, ya que ha alcanzado el límite *hard* de `9` inodos.

---

**71. Borre uno de los ficheros creados y pruebe, como usuario normal, a intentar llenar el disco duro (use `dd`, por ejemplo). ¿Ocurre algún error? ¿Permanece el fichero creado? (la respuesta debe ser sí) ¿qué tamaño tiene?**

*   **Respuesta:**
    *   **Comandos (como `asiuser`):**
        ```bash
        rm prueba1
        dd if=/dev/zero of=fichero_grande bs=1M count=50
        ls -lh fichero_grande
        ```
    *   **¿Ocurre algún error?:** **Sí**, `dd` se detiene antes de terminar y muestra el error: `dd: error writing 'fichero_grande': Disk quota exceeded`.
    *   **¿Permanece el fichero creado?:** **Sí**, el fichero `fichero_grande` permanece en el directorio.
    *   **¿Qué tamaño tiene?:** Tiene un tamaño aproximado de **`9.7 MiB`** (`9972` bloques de 1 KiB), es decir, el límite *hard* de `10000` bloques menos los `28` bloques que ya ocupaban los ficheros previos del usuario.

---

**72 y 73. Deshacer los cambios de la cuarta parte y restaurar `/home` original.**

*   **Respuesta:**
    ```bash
    # (Como root, sin sesiones abiertas del usuario normal)
    quotaoff /home
    umount /home
    rmdir /home
    mv /home.old /home
    ```
    *   Además, se edita **`/etc/fstab`** para eliminar la línea `/dev/sdb1 /home ext4 defaults 0 2` que añadimos en el paso 62.

---

### Extra time: Quinta parte. Combinando LVM y MDADM, redimensionando sistemas de ficheros y usando snapshots

**74 y 75. Desmontar sistemas previos y crear un RAID 5 con los tres primeros discos adicionales (`sdb1` a `sdd1`).**

*   **Respuesta:**
    ```bash
    # Asegurarnos de que no hay nada montado en /pto1 o /pto2
    umount /pto1 /pto2 2>/dev/null

    # Crear RAID 5 de 3 discos
    mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sdb1 /dev/sdc1 /dev/sdd1
    ```

---

**76, 77 y 78. Comprobar estado inicial del RAID5, crear sistema de ficheros y volver a comprobar.**

*   **Respuesta:**
    ```bash
    mdadm --detail /dev/md0
    mkfs -t ext4 /dev/md0
    mdadm --detail /dev/md0
    ```
    *(Al crear un RAID 5 de 3 discos, `mdadm` suele iniciar con 2 discos activos y 1 en reconstrucción (`degraded, recovering`). Al escribir los metadatos con `mkfs -t ext4`, se fuerza la actividad/sincronización y al poco tiempo pasa a estado `clean` con los 3 discos en `active sync`).*

---

**79. Añada el disco que le faltaba al RAID5. Compruebe el estado mediante `mdadm` ¿ha cambiado el tamaño utilizable del RAID? ¿Cómo (de qué tipo) aparece ahora el disco `sde1`?**

*   **Respuesta:**
    *   **Comandos:**
        ```bash
        mdadm --manage /dev/md0 --add /dev/sde1
        mdadm --detail /dev/md0
        ```
    *   **¿Ha cambiado el tamaño utilizable del RAID?:** **No**, el tamaño utilizable (`Array Size`) sigue siendo exactamente el mismo (~`998 MiB` o `1000 MiB`, equivalente a $(3 - 1) \times 500\text{ MiB}$).
    *   **¿De qué tipo aparece ahora `sde1`?:** Aparece como **`spare`** (disco de reserva en caliente / *hot-spare*), listo para sustituir automáticamente a cualquiera de los 3 discos activos en caso de fallo.

---

**80. Use LVM para crear un VG llamado "Labextra" que contenga (como único PV) el multidispositivo creado. Indique qué es lo que ha tecleado. ¿Qué tamaño tiene ese VG?**

*   **Respuesta:**
    *   **Comandos tecleados:**
        ```bash
        pvcreate /dev/md0
        vgcreate Labextra /dev/md0
        vgdisplay Labextra
        ```
        *(Responder `y` si pide limpiar la firma `ext4` previa de `/dev/md0`).*
    *   **Tamaño del VG (`VG Size`):** Aproximadamente **`992.00 MiB`** (el tamaño útil de los 2 discos de datos del RAID 5 menos la cabecera de metadatos de LVM).

---

**81. Use `lvcreate` para crear dos LV. Uno de 20MiB y el otro del tamaño máximo posible (mire el espacio que le ha sobrado después de crear el primero).**

*   **Respuesta:**
    ```bash
    lvcreate -L 20M -n lv_peq Labextra
    vgdisplay Labextra
    # Mirando el campo "Free PE / Size" (ej. 243 PE / 972.00 MiB), usamos -l 100%FREE:
    lvcreate -l 100%FREE -n lv_grande Labextra
    ```

---

**82. Cree y monte dos sistemas de ficheros en `/pto1` y `/pto2`.**

*   **Respuesta:**
    ```bash
    mkfs -t ext4 /dev/Labextra/lv_peq
    mkfs -t ext4 /dev/Labextra/lv_grande
    mount /dev/Labextra/lv_peq /pto1
    mount /dev/Labextra/lv_grande /pto2
    ```

---

**83. Cree en el sistema de ficheros "grande" un archivo de 50MiB. Intente copiarlo en el pequeño y verá que no es posible. Compruebe con `df` que el disco más pequeño está, efectivamente, lleno.**

*   **Respuesta:**
    ```bash
    dd if=/dev/zero of=/pto2/fich50M bs=1M count=50
    cp /pto2/fich50M /pto1/
    # Salida: cp: error writing '/pto1/fich50M': No space left on device
    df -h /pto1
    # Muestra Use% = 100% (Avail = 0)
    ```

---

**84. Anote el espacio del disco grande (use `df`).**

*   **Respuesta:**
    ```bash
    df -h /pto2
    ```
    *(Muestra un tamaño total `Size` de aproximadamente **`942M`**).*

---

**85. Use `lvresize` para reducir en 100MiB el tamaño del LV grande. Utilice (además de otras opciones) la opción `-r`. ¿Qué ocurre? ¿Por qué le hace esa pregunta? Diga que sí y continúe. Anote brevemente y comente los mensajes que aparecen.**

*   **Respuesta:**
    *   **Comando:**
        ```bash
        lvresize -r -L -100M /dev/Labextra/lv_grande
        ```
    *   **¿Qué ocurre y por qué pregunta?:** Como el sistema de ficheros `ext4` de `/pto2` está actualmente **montado** y los sistemas de ficheros `ext4` **no soportan reducción en caliente (online)**, `lvresize -r` pregunta si deseamos desmontarlo (`Do you want to unmount "/pto2"? [Y/n]`).
    *   **Comentarios de los mensajes tras responder `Y`:**
        1.  Desmonta `/pto2`.
        2.  Ejecuta automáticamente `e2fsck -f` para comprobar la integridad del sistema de ficheros antes de encogerlo.
        3.  Ejecuta `resize2fs` reduciendo el sistema de ficheros `ext4` en 100 MiB.
        4.  Reduce el volumen lógico `lv_grande` en 100 MiB (de 972 MiB a 872 MiB).
        5.  Vuelve a montar automáticamente `/pto2`.

---

**86. ¿Qué muestra ahora la salida de `df` respecto a ese sistema de ficheros?**

*   **Respuesta:**
    ```bash
    df -h /pto2
    ```
    *(El tamaño total `Size` ha disminuido en 100 MiB, pasando de ~`942M` a aproximadamente **`844M`**).*

---

**87 y 88. Aumentar el tamaño del volumen pequeño en 100MiB usando `-r` y comprobar con `df`.**

*   **Respuesta:**
    ```bash
    lvresize -r -L +100M /dev/Labextra/lv_peq
    df -h /pto1
    ```
    *   **¿Qué ocurre ahora?:** Esta vez **no pide desmontar `/pto1`**, porque `ext4` sí permite la **expansión en caliente (online)**. El volumen lógico pasa de 20 MiB a 120 MiB y `df -h /pto1` confirma que el tamaño útil del sistema de ficheros ha aumentado a ~**`112M`**.

---

**89 y 90. Notas explicativas sobre `lvresize -r`, `resize2fs` y redimensionado online/offline.**

*   **Respuesta:** *(Pasos informativos de lectura).*

---

**91. Ahora vamos a probar la característica de snapshot de LVM. Para ello desmontaremos el sistema de ficheros del volumen pequeño y eliminaremos ese LV.**

*   **Respuesta:**
    ```bash
    umount /pto1
    lvremove -f /dev/Labextra/lv_peq
    ```

---

**92. Cree ahora un LV que sea un snapshot de 100MiB del LV grande que creó anteriormente (ver transparencias) y móntelo (directorio `/snapshot`) sin crear ahí ningún sistema de ficheros. ¿Por qué no ha sido necesario crear un sistema de ficheros?**

*   **Respuesta:**
    *   **Comandos:**
        ```bash
        lvcreate -L 100M -s -n snap_grande /dev/Labextra/lv_grande
        mkdir /snapshot
        mount /dev/Labextra/snap_grande /snapshot
        ```
    *   **¿Por qué no ha sido necesario crear un sistema de ficheros?:** Porque un *snapshot* (instantánea) de LVM es una copia lógica punto-en-el-tiempo (*Copy-on-Write*) del volumen origen (`lv_grande`), por lo que hereda y refleja exactamente la misma estructura de bloques y el mismo sistema de ficheros `ext4` que ya existía en el volumen original.

---

**93. ¿Tiene el mismo contenido el sistema de ficheros origen (el grande) y el sistema de ficheros del snapshot? De hecho, ¿qué tamaño dice `df` que tienen los dos?**

*   **Respuesta:**
    *   **Comandos:**
        ```bash
        ls -la /pto2 /snapshot
        df -h /pto2 /snapshot
        ```
    *   **Contenido y tamaño:** **Sí**, ambos muestran exactamente el mismo contenido (el fichero `fich50M` y `lost+found`) y `df -h` reporta que **ambos tienen el mismo tamaño total (~`844M`)**, aunque físicamente el snapshot solo tenga reservados 100 MiB para almacenar los bloques que vayan cambiando en el original.

---

**94. Cree en el sistema de ficheros original otro fichero de 50MB. ¿Aparece ese fichero en `/snapshot`? Compruebe, mediante `lvdisplay`, el estado del snapshot. Anote el % de uso. Pruebe a crear otro fichero de 60MiB y vea qué ocurre.**

*   **Respuesta:**
    *   **Creación del fichero de 50 MiB:**
        ```bash
        dd if=/dev/zero of=/pto2/fich50M_nuevo bs=1M count=50
        ls -la /snapshot
        lvdisplay /dev/Labextra/snap_grande
        ```
    *   **¿Aparece en `/snapshot`?:** **No.** `/snapshot` conserva la foto fija del momento en que se creó (solo tiene `fich50M`).
    *   **Porcentaje de uso en `lvdisplay` (`Allocated to snapshot`):** Alrededor del **`50.2%`** (ya que se han escrito ~50 MiB nuevos en el origen, obligando al mecanismo *Copy-on-Write* a guardar esos 50 MiB de bloques originales en los 100 MiB del snapshot).
    *   **Creación de otro fichero de 60 MiB (`dd if=/dev/zero of=/pto2/fich60M bs=1M count=60`):**
        Al superar los $50 + 60 = 110\text{ MiB} > 100\text{ MiB}$ de capacidad del snapshot, **el snapshot se llena por completo y se invalida/desactiva** (`LV snapshot status: INACTIVE` en `lvdisplay`), perdiendo el acceso a `/snapshot` (errores de E/S) para no bloquear las escrituras en el volumen original.

---

**95. Desmonte `/snapshot` y elimine su LV.**

*   **Respuesta:**
    ```bash
    umount /snapshot
    lvremove -f /dev/Labextra/snap_grande
    ```

---

**96 y 97. Explicación conceptual de LVM sobre RAID 5 con Hot-Spare.**

*   **Respuesta:** *(Pasos informativos de lectura).*

---

**98, 99 y 100. Provoque el fallo en un disco activo del RAID. Use `mdadm` para monitorizar el estado y espere a que se detecte el fallo. ¿Qué ocurre ahora que no sucedía antes? Mientras el sistema RAID se recupera, elimine el disco que ha fallado del RAID, vuelva a activarlo e insértelo nuevamente en el RAID. Una vez insertado el disco ¿Pasa a estar activo o se queda como reserva?**

*   **Respuesta:**
    *   **Provocar fallo y monitorizar:**
        ```bash
        echo "offline" > /sys/block/sdb/device/state
        touch /pto2/test_fallo
        mdadm --detail /dev/md0
        ```
    *   **¿Qué ocurre ahora que no sucedía antes?:** Al existir un disco de reserva en caliente (*hot-spare*, `/dev/sde1`), en cuanto `mdadm` detecta que `/dev/sdb1` ha fallado (`faulty`), **inicia automáticamente la reconstrucción/sincronización (`spare rebuilding`) sobre `/dev/sde1`** sin intervención manual del administrador. Además, los volúmenes lógicos de LVM siguen funcionando sin interrupción.
    *   **Reinserción del disco fallido:**
        ```bash
        mdadm --manage /dev/md0 --remove /dev/sdb1
        echo "running" > /sys/block/sdb/device/state
        mdadm --manage /dev/md0 --add /dev/sdb1
        mdadm --detail /dev/md0
        ```
    *   **Estado tras insertarlo:** El disco recién insertado (`/dev/sdb1`) **se queda como reserva (`spare`)**, ya que el antiguo disco de reserva (`/dev/sde1`) ha pasado a ocupar su puesto como disco activo (`active sync`) dentro del RAID 5.