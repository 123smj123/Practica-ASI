# Administración de Sistemas de Información
## Estudio Experimental - Práctica 4: Procesos
### Entorno: Debian 13

---

### Primera parte: Árbol de procesos

#### 1. Antes de comenzar, es necesario que esté instalada la aplicación "psmisc" en Debian. Si no lo tiene instalada puede hacerlo (como superusuario) tecleando "apt update" (esto permite actualizar la base de datos de paquetes de Debian) y luego "apt install psmisc"

**Respuesta:**  
Para instalar el paquete se accede a una sesión de administración con `su -` (o ejecutando con privilegios de superusuario) e introduciendo los comandos indicados:
```bash
apt update
apt install psmisc
```

---

#### 2. "pstree" es una aplicación que permite visualizar, en modo "gráfico", la relación paterno-filial de los procesos. Inicie sesión como usuario normal en el terminal tty1 y ejecute ese programa con la opción "-p" (para que muestre también los PID de los procesos). Puede comprobar que una de las ramas del árbol contiene el proceso "bash" que está usando la Shell. Comprobará que hay un proceso inicial que se llama "systemd" con PID=1. Este proceso hace las mismas funciones que el proceso "init" explicado en clase. ¿Por qué "systemd" no tiene padre?

**Respuesta:**  
Comando ejecutado:
```bash
pstree -p
```
`systemd` (PID=1) no tiene padre dentro del espacio de usuario (*user space*) porque es el primer proceso lanzado directamente por el núcleo del sistema operativo (*kernel*) una vez terminada la fase de arranque e inicialización del hardware. Actúa como el ancestro común o raíz de todos los demás procesos y asume el rol de proceso `init`, adoptando además a cualquier proceso cuyos padres hayan finalizado antes que ellos (evitando procesos huérfanos). En las estructuras internas del kernel, su identificador de padre (`PPID`) figura como 0 (el propio núcleo).

---

#### 3. Observará que en una de las ramas de pstree hay un proceso llamado "bash" cuyo proceso padre es "login". Eso se debe a que ese proceso es el que realmente controla el inicio de sesión (verificando el nombre de usuario y contraseña). Dado que el proceso "login" se encarga de verificar la contraseña del usuario (recuerde los permisos que tiene establecido /etc/shadow), ¿qué EUID cree que tiene el proceso "login" que es padre de la shell que está ejecutando?

**Respuesta:**  
El proceso `login` tiene un `EUID = 0` (correspondiente al usuario `root`).  
**Justificación:** El archivo `/etc/shadow`, que contiene los hashes de las contraseñas del sistema, tiene permisos restringidos (generalmente `-rw-r-----` o `0640` con propietario `root:shadow`), de modo que solo un proceso con privilegios efectivos de superusuario (`EUID=0`) puede leerlo para validar las credenciales introducidas.

---

#### 4. Compruebe (usando ps) que lo que contestó en el apartado anterior es correcto y anote lo que ha hecho. NOTA: Necesitará consultar la página de manual de "ps" para saber cómo ejecutar "ps" para que muestre los parámetros de los procesos que desea.

**Respuesta:**  
Se ejecuta `ps` seleccionando las columnas de PID, PPID, UID, EUID y comando:
```bash
ps -eo pid,ppid,uid,euid,comm | grep login
```
*(O de forma directa: `ps -o pid,ppid,user,euser,comm -C login`)*.  
Al observar la salida, en la columna correspondiente al `EUID` (o `EUSER`) del proceso `login` aparece el valor `0` (o `root`), confirmando lo contestado en el apartado anterior.

---

#### 5. Compruebe cuál es el EUID y UID de la bash que está usando (proceso hijo del proceso "login" que observó anteriormente). ¿Ha ejecutado el proceso padre (login) alguna llamada a sistema antes de ejecutar el programa "bash"?

**Respuesta:**  
Para comprobar los identificadores de la shell en uso, se ejecuta `ps` pasando como filtro el PID de la propia shell (`$$`):
```bash
ps -o pid,ppid,uid,euid,comm -p $$
```
- **UID y EUID:** Ambos coinciden y corresponden al identificador del usuario regular autenticado (por ejemplo, `UID=1000` y `EUID=1000`).
- **Llamadas al sistema:** Sí. El proceso `login` se ejecutaba inicialmente con privilegios de `root` (`EUID=0`). Tras verificar la autenticación en `/etc/shadow`, `login` ejecutó llamadas al sistema como `setgid()` y `setuid()` para cambiar su identidad a la del usuario normal, despojándose de los privilegios de root. Posteriormente ejecutó `exec()` (o familia `execve`) para reemplazar el binario de `login` por el de `/bin/bash`.

---

#### 6. En tty1, teclee "su -" para adquirir privilegios de superusuario, teclee ahora "pstree -p" en ese terminal. ¿Por qué aparece ahora que el proceso "pstree" tiene un proceso padre distinto del de antes?

**Respuesta:**  
Al ejecutar `su -`, la shell original del usuario no se reemplaza, sino que crea un proceso hijo mediante `fork()` para ejecutar `su`. A su vez, `su` autentica al superusuario y crea un nuevo proceso hijo `bash` con entorno y privilegios de `root`. Como este nuevo `bash` es la consola interactiva activa desde la que se lanza `pstree`, el padre de `pstree` pasa a ser esta nueva shell de `root`, y no la shell del usuario normal que la originó.

---

#### 7. Inicie ahora sesión en tty2 como usuario normal. Compruebe, mediante "pstree", que ahora hay dos ramas del árbol para cada una de las sesiones en las que ha iniciado sesión. En tty2, teclee "exec su -" y compruebe ahora la salida de "pstree"¿Por qué la rama de procesos del tty1 hay un proceso "bash" adicional si lo comparo con el de tty2?

**Respuesta:**  
- En **tty1**, se ejecutó `su -` directamente, lo que provocó una llamada al sistema `fork()`. Esto mantiene vivo el `bash` original del usuario como padre y añade un nuevo proceso `bash` para `root` en la jerarquía.
- En **tty2**, se ejecutó `exec su -`. La orden interna `exec` sustituye el ejecutable del proceso actual por el nuevo comando (`su`) manteniendo el mismo PID, sin hacer `fork`. La shell de usuario original desaparece de la memoria al ser reemplazada por `su`, la cual a su vez arranca la sesión de `root`. En consecuencia, en tty2 hay un proceso `bash` menos que en tty1.

---

### Segunda parte: Prioridades

#### 8. NOTA: A partir de ahora, a no ser que sea necesario, todos los pasos se realizarán como usuario normal.

*Instrucción operativa tenida en cuenta para las siguientes preguntas.*

---

#### 9. En esta parte de la práctica, usaremos el script de "carga" que se explicó en clase. Deberá crear ese script en la máquina virtual. Se trata de este:
```bash
#!/bin/bash
while [ 1 ];
do
i=10
done
```

**Respuesta:**  
Se crea el archivo `carga` y se le asignan permisos de ejecución:
```bash
cat << 'EOF' > carga
#!/bin/bash
while [ 1 ];
do
i=10
done
EOF
chmod +x carga
```

---

#### 10. Ejecute "time ./carga" durante 30s y teclee "ctrl-C". ¿Qué porcentaje (sobre el total) ha estado el proceso usando la CPU? ¿Cómo lo ha calculado? ¿Debe considerar en ese tiempo (el de uso de CPU) el tiempo que se señala en sys (aunque sea muy pequeño)?

**Respuesta:**  
Comando ejecutado:
```bash
time ./carga
```
*(Se espera ~30 segundos y se pulsa `Ctrl-C`)*.

- **Porcentaje de uso:** El proceso ha estado usando prácticamente el **100%** de un núcleo de CPU.
- **Cálculo:** La métrica de uso efectivo de CPU es la suma del tiempo consumido en espacio de usuario (`user`) y en espacio de sistema (`sys`), dividida entre el tiempo total de reloj transcurrido (`real`):
  $$\% \text{CPU} = \frac{\text{user} + \text{sys}}{\text{real}} \times 100$$
- **Consideración de `sys`:** Sí, debe considerarse obligatoriamente. Aunque el bucle no haga llamadas al sistema explícitas y la mayoría del tiempo sea `user`, el campo `sys` representa el tiempo real durante el cual la CPU estuvo ejecutando código del núcleo en nombre de dicho proceso (gestión de interrupciones de reloj, asignación de páginas, etc.).

---

#### 11. A partir de ahora, se usarán varios terminales. En tty1 se estará ejecutando "top" u otros programas para monitorizar procesos. En tty2, tty3, etc se ejecutarán distintos programas que hacen uso de CPU (por ejemplo, el programa "carga").

**Respuesta:**  
En `tty1` se lanza la herramienta de monitorización:
```bash
top
```
El resto de terminales virtuales (`tty2`, `tty3`, ...) quedan disponibles para lanzar las cargas de trabajo.

---

#### 12. Repita el apartado 10, pero en esta ocasión hágalo en dos terminales distintos (tty2 y tty3, por ejemplo). NOTA: Procure que no transcurra mucho tiempo entre la ejecución de ambos programas (el objetivo es que estén ejecutándose simultáneamente la mayor parte del tiempo). ¿Coincide la suma de los tiempos de usuario y sistema con el tiempo real? ¿Por qué no coinciden ahora?

**Respuesta:**  
- **Resultado:** No coincide. En un sistema monoprocesador (1 solo núcleo o vCPU), para cada proceso la suma `user + sys` será aproximadamente la mitad (~50%) del tiempo `real` transcurrido.
- **Justificación:** Ambos procesos compiten por el único procesador disponible. El planificador de tareas de Linux (*scheduler*) tiene que repartir los ciclos de la CPU equitativamente mediante rodajas de tiempo (*time slicing*). Por tanto, en 30 segundos de tiempo real, cada proceso sólo habrá tenido oportunidad de cómputo en la CPU durante unos 15 segundos.

---

#### 13. Sobre el tty2, lance (sin usar time) 4 procesos "carga" (procure, nuevamente, no tardar mucho tiempo en hacerlo). En tty1 compruebe, usando "top", que los 4 procesos se están ejecutando. Espere unos minutos y compruebe a que valor tiende el primer campo de "load average" ¿qué indica este valor?

**Respuesta:**  
En `tty2`, se lanzan en segundo plano usando `&`:
```bash
./carga & ./carga & ./carga & ./carga &
```
- **Valor al que tiende:** El primer valor de *load average* (promedio de carga del último minuto) tiende progresivamente a **4.0** (o ligeramente superior).
- **Qué indica:** Representa el número promedio de procesos que se encuentran activos compitiendo por recursos: bien ejecutándose o en la cola listos para usar la CPU (estado `R`), o bien bloqueados a la espera de operaciones de entrada/salida no interrumpibles (estado `D`).

---

#### 14. Observará que los 4 procesos usan el 25% de la CPU. Avise al profesor si no es así. De los pasos necesarios para que uno de los procesos "carga" tenga la menor prioridad posible, es decir, que se le asigne muy poco tiempo de CPU. ¿Qué ha hecho? ¿Qué % de CPU tienen ahora los otros tres procesos? ¿Y el cuarto?

**Respuesta:**  
- **Pasos dados:** Se identifica el PID de uno de los procesos mediante `top` o `pgrep carga` y se le asigna la menor prioridad posible (el valor de cortesía máximo, `nice = +19`) con el comando:
  ```bash
  renice +19 -p <PID>
  ```
  *(También se puede pulsar `r` en `top`, introducir el PID y escribir `19`)*.
- **Distribución de % CPU:**
  - El cuarto proceso (con `nice +19`) pasa a consumir un porcentaje mínimo de CPU, entre un **1% y un 3%**.
  - Los otros tres procesos (que conservan su prioridad estándar `nice 0`) se reparten equitativamente el 97%-99% restante de la CPU, obteniendo aproximadamente un **32% - 33% de CPU cada uno**.

---

#### 15. Mientras continúa monitorizando el uso del sistema, anote el % de tiempo que la CPU está en espacio de usuario (us) y en espacio de sistema (sy). Compruebe ahora el valor de la columna TIME. ¿Se observan diferencias notables entre los valores de la columna TIME de los 4 procesos? A tenor de lo observado, ¿cree que ese campo muestra el tiempo REAL desde el que se empezó a ejecutar el proceso?

**Respuesta:**  
- **Valores %us y %sy:** `%us` se mantiene en valores muy altos (~98% - 100%), mientras que `%sy` está prácticamente en 0%, ya que los scripts ejecutan bucles simples de variables en memoria sin apenas requerir servicios del núcleo.
- **Diferencias en TIME:** Sí existen diferencias muy notables. La columna `TIME` (o `TIME+`) avanza rápidamente en los tres procesos con prioridad estándar y avanza de forma mucho más lenta en el proceso con `renice +19`.
- **Conclusión sobre TIME:** La columna `TIME` **no** muestra el tiempo real (tiempo transcurrido de reloj), sino el **tiempo acumulado de CPU** (*CPU time*) que dicho proceso ha estado efectivamente ejecutándose dentro del procesador.

---

#### 16. Parece obvio que uno de los procesos "carga" tiene menos % de uso de CPU que el resto. ¿Se ve eso reflejado en la carga del sistema ("load average")? ¿Por qué?

**Respuesta:**  
- **No se ve reflejado:** El *load average* se mantiene en torno a 4.0.
- **Por qué:** Porque el cálculo de la carga del sistema cuenta cuántos procesos están en la cola de ejecución listos para correr (estado `R`). Aunque el proceso con `nice +19` reciba menos rodajas de tiempo de la CPU, sigue estando activo y queriendo procesar instrucciones, por lo que suma exactamente 1 a la métrica de procesos en demanda de CPU.

---

#### 17. Envíe la señal "TERM" a los tres procesos "carga" que están usando la mayor parte de la CPU. Observará que los procesos terminan, tal y como se establece en el comportamiento "por defecto" para esta señal. Una vez que han finalizado esos tres procesos, ¿cuál es el % de CPU que se le asigna al restante? ¿Cómo es eso posible si se le asignó una prioridad muy baja?

**Respuesta:**  
Comando ejecutado para los tres procesos prioritarios:
```bash
kill -TERM <PID1> <PID2> <PID3>
```
- **% CPU asignado al restante:** Se le asigna prácticamente el **100% de la CPU**.
- **Cómo es posible:** El valor de *nice* es una prioridad relativa que actúa cuando hay competencia entre varios procesos listos para ejecutarse. Si no existen otros procesos demandando el uso de la CPU, el planificador no deja la CPU ociosa e invierte todos los ciclos disponibles en el único proceso ejecutable, aun cuando tenga asignada la prioridad más baja (`nice +19`).

---

#### 18. Compruebe ahora el valor de "%id". Anótelo. A la vista del resultado ¿cree que se está usando más de un procesador/core en la máquina virtual?

**Respuesta:**  
El valor de `%id` (*idle*, tiempo de CPU desocupado) es prácticamente **0.0%**.  
A la vista del resultado, se deduce que la máquina virtual tiene asignado **un único procesador/núcleo** (1 core/vCPU). Si dispusiera de 2 o más núcleos, un único proceso de un solo hilo sólo podría saturar uno de ellos (100% de un core), y el monitor `top` mostraría al menos un 50% de inactividad global en `%id`.

---

#### 19. Termine con el proceso "carga" que queda.

**Respuesta:**  
Comando ejecutado:
```bash
killall carga
```
*(O alternativamente: `kill -TERM <PID_restante>`)*.

---

### Segunda parte: %sy

#### 20. Haga una copia el script "carga" y póngale el nombre "carga_y_mas". Añada la línea "echo $i > /dev/null" dentro del bucle "while". ¿Para qué sirve esto?

**Respuesta:**  
Se duplica el script y se edita su contenido:
```bash
cp carga carga_y_mas
```
El script resultante es:
```bash
#!/bin/bash
while [ 1 ];
do
i=10
echo $i > /dev/null
done
```
- **Para qué sirve:** Sirve para forzar al script a realizar constantes llamadas al sistema (concretamente `write` sobre el descriptor abierto de `/dev/null`) en cada iteración del bucle, provocando frecuentes cambios de contexto entre el modo usuario y el modo kernel (espacio de sistema).

---

#### 21. Ejecute el nuevo script (en tty2) y compruebe (mediante "top") cuánto %CPU tiene el proceso es el mismo que tenía en el apartado 17? Observará que el valor de %sy es distinto, ¿por qué? ¿Por qué ese proceso no genera %wa (tiempo de espera E/S) si está escribiendo en un fichero de dispositivo?

**Respuesta:**  
En `tty2` se ejecuta:
```bash
./carga_y_mas
```
- **%CPU global:** El uso total del proceso sigue rondando el **100%**, igual que en el apartado 17.
- **Por qué `%sy` es distinto:** En el apartado 17 el porcentaje de sistema `%sy` era prácticamente 0% porque solo se ejecutaban instrucciones internas del intérprete. Ahora, `%sy` sube notablemente (pudiendo alcanzar 30%–50% o más) debido al tiempo que la CPU pasa dentro del kernel gestionando las llamadas al sistema requeridas por las operaciones de escritura continua.
- **Por qué no genera `%wa`:** No genera *iowait* (`%wa`) porque `/dev/null` es un dispositivo especial de caracteres puramente virtual gestionado directamente en memoria por el núcleo. Como no implica esperar a que un medio de almacenamiento físico secundario (disco mecánico o SSD) complete bloques de lectura/escritura, el proceso nunca se bloquea en espera de E/S síncrona (estado `D`).

---

#### 22. Termine el proceso creado en el apartado anterior

**Respuesta:**  
Comando ejecutado:
```bash
killall carga_y_mas
```
*(O pulsando `Ctrl-C` directamente en el terminal `tty2`)*.

---

### Tercera parte: ¿Procesos inmortales?

#### 23. Copie el script "carga" y póngale el nombre "inmortal". Añada, antes del bucle "while", lo siguiente: trap 'echo "No puedes matarme"' TERM INT. ¿Para qué sirve esto?

**Respuesta:**  
Se crea el fichero `inmortal`:
```bash
cp carga inmortal
```
El script queda con la siguiente estructura:
```bash
#!/bin/bash
trap 'echo "No puedes matarme"' TERM INT
while [ 1 ];
do
i=10
done
```
- **Para qué sirve:** La instrucción `trap` establece un manejador de señales (*signal handler*) personalizado en el script. En lugar de ejecutar la acción por defecto para las señales `TERM` (15) e `INT` (2) —que provocarían la muerte o cancelación del proceso—, el script intercepta ambas señales, ejecuta la acción asociada (imprime por pantalla `"No puedes matarme"`) y continúa ejecutando su bucle con normalidad.

---

#### 24. Lance (en primer plano) el proceso "inmortal" en tty2 y desde el programa "top" envíe la señal TERM a ese proceso. ¿Termina ahora el proceso? ¿Se muestra algún mensaje en tty2? ¿Qué es lo que está ocurriendo?

**Respuesta:**  
Pasos ejecutados:
1. En `tty2`: `./inmortal`
2. En `tty1` (dentro de `top`): se presiona `k`, se teclea el PID de `inmortal` y se indica la señal `15` (o `TERM`).

- **¿Termina el proceso?:** No, no finaliza.
- **¿Qué mensaje se muestra?:** En `tty2` se imprime la cadena: `No puedes matarme`.
- **Qué ocurre:** La señal `SIGTERM` es una señal capturable por software. El comando `trap` interceptó la señal y ejecutó el bloque definido en vez de permitir que el núcleo aplique la acción por defecto de finalización.

---

#### 25. Intente parar el proceso "inmortal" mediante la combinación "Ctrl-C" desde el terminal donde fue abierto. ¿Qué ocurre? ¿A qué señal se refiere ese "Ctrl-C"?

**Respuesta:**  
- **Qué ocurre:** El proceso no se detiene. En el terminal aparece una vez más el texto `No puedes matarme` y el bucle sigue corriendo indefinidamente.
- **A qué señal se refiere:** La combinación de teclado `Ctrl-C` envía la señal de interrupción `SIGINT` (señal número 2), la cual también fue interceptada explícitamente en el manejador del `trap`.

---

#### 26. ¿Existe alguna forma de terminar con el proceso "inmortal" sin lugar a dudas, "pase lo que pase"? Nota: No vale reiniciar el sistema (Pista: La respuesta es sí) Hágalo e indique qué ha hecho y qué se muestra entonces en tty2.

**Respuesta:**  
Sí, enviando la señal `SIGKILL` (señal número 9).  
**Justificación:** A nivel de arquitectura de procesos en Linux/UNIX, las señales `KILL` (9) y `STOP` no pueden ser capturadas por manejadores (`trap`), ignoradas ni bloqueadas por ningún proceso. Son procesadas de forma directa e incontestable por el propio núcleo, el cual destruye el proceso de inmediato.

Comando ejecutado (desde otro terminal o desde `top` con la opción `k` -> señal `9`):
```bash
kill -9 <PID>
```
*(O mediante `killall -9 inmortal`)*.

- **Qué se muestra en tty2:** El proceso es destruido bruscamente y la terminal muestra el mensaje:
  ```text
  Killed
  ```
  devolviendo el control al intérprete de comandos.

---

### Cuarta parte: Tareas

#### 27. Para instalar "at", debe teclear "apt update" y "apt install at". Eso instalará no solo el programa "at" sino también otras utilidades adicionales tales como un gestor de correo sencillo.

**Respuesta:**  
Accediendo con privilegios de superusuario (`su -`), se instalan las utilidades requeridas:
```bash
apt update
apt install at
```

---

#### 28. Inicie sesión como usuario "normal". Antes de continuar, vamos a limpiar el buzón de correo del usuario (por si no está vacío). Para ello se ejecutará "mail". La forma más fácil de eliminar todos los correos del buzón es tecleando, dentro del programa mail, "d*". Hágalo. NOTA: Si le sale un mensaje indicando que no tiene correo, ignórelo y continúe al paso siquiente

**Respuesta:**  
Iniciando sesión como usuario normal, se abre la utilidad de correo:
```bash
mail
```
Si existen correos, en el prompt `?` se introduce:
```text
? d*
? q
```
Si mostraba `No mail for <usuario>`, se continúa directamente al siguiente apartado.

---

#### 29. Antes de continuar, asegúrese de que no ha iniciado sesión en el tty6.

**Respuesta:**  
Se comprueba mediante la orden `who` o `w` que ninguna consola activa tenga una sesión abierta en `tty6`.

---

#### 30. ¿Cuál es el fichero de dispositivo asociado a tty6? Compruebe que existe y sus permisos. Anótelo.

**Respuesta:**  
Comando ejecutado:
```bash
ls -l /dev/tty6
```
- **Fichero de dispositivo:** Es `/dev/tty6` (dispositivo de caracteres).
- **Permisos por defecto en Debian:**
  ```text
  crw--w---- 1 root tty 4, 6 ... /dev/tty6
  ```
  Los permisos son de lectura y escritura para su propietario (`root`), permiso exclusivo de escritura para el grupo (`tty`), y sin ningún tipo de permiso para el resto de usuarios (`otros: ---`).

---

#### 31. Use las herramientas adecuadas para lanzar una tarea recurrente como usuario sin privilegios. Dicha tarea consistirá en mostrar el mensaje "Hola" en el tty6. Ese mensaje se tendrá que mostrar cada minuto de lunes a viernes. De los pasos necesarios e indique qué ha hecho.

**Respuesta:**  
Pasos realizados:
1. Como usuario normal, se edita la tabla periódica de cron mediante:
   ```bash
   crontab -e
   ```
2. Se añade la siguiente especificación cron al final del archivo:
   ```text
   * * * * 1-5 echo "Hola" > /dev/tty6
   ```
   - Primer asterisco: cada minuto (`*`).
   - Segundo asterisco: cada hora (`*`).
   - Tercer asterisco: cada día del mes (`*`).
   - Cuarto asterisco: cada mes (`*`).
   - Rango `1-5`: de lunes a viernes.
3. Se guarda y se cierra el editor.

---

#### 32. Observará que lo que está intentado hacer no funciona (mire si en tty6 sale algún mensaje). Puede comprobar lo que ocurre si lee el correo (ejecute mail y teclee el número del mensaje de correo que desea leer). Dado que el proceso que controla las tareas (cron) le enviará un correo si la tarea produce alguna salida por stderr o stdout. ¿Por qué no funciona? Nota: Puede que tenga que esperar 1m para recibir ese correo. Puede usar el programa "date" para ver la hora del sistema.

**Respuesta:**  
Al cabo de un minuto se abre el gestor de correo ejecutando `mail` y se consulta el mensaje entrante generado por el demonio cron. El cuerpo del mensaje muestra el error:
```text
/bin/sh: line 1: /dev/tty6: Permission denied
```
- **Por qué no funciona:** Porque la tarea recurrente fue configurada en el crontab personal del usuario normal. El dispositivo `/dev/tty6` tiene permisos `crw--w----` con propietario `root` y grupo `tty`. El usuario normal no es root ni pertenece al grupo secundario `tty`, por lo que el sistema de ficheros deniega el permiso de escritura necesario para redirigir la salida al terminal `tty6`.

---

#### 33. Borre los mensajes de correo tecleando "d" en el prompt y salga del programa "mail" con "q".

**Respuesta:**  
Dentro del programa `mail`, se introduce `d` (o `d*`) para marcar los mensajes como borrados y posteriormente `q` para aplicar los cambios y salir a la shell:
```text
? d*
? q
```

---

#### 34. Elimine esa tarea recurrente. ¿Cómo lo ha hecho?

**Respuesta:**  
Comando ejecutado:
```bash
crontab -r
```
*(O accediendo con `crontab -e` y borrando manualmente la línea introducida)*.

---

#### 35. Dado que lo que hemos intentado no funciona, optaremos por otra estrategia. De lo que se trata ahora es de crear una tarea global del sistema que permita llevar a cabo lo que se señalaba en el punto 31 (el usuario que se debe usar para esa tarea global es root). Indique los pasos que ha dado y compruebe que ahora sí que funciona.

**Respuesta:**  
Pasos realizados:
1. Se accede a una consola con privilegios de superusuario (`su -`).
2. Se edita el archivo de configuración global del sistema `/etc/crontab`:
   ```bash
   nano /etc/crontab
   ```
3. Se añade la regla indicando explícitamente el campo de usuario como `root`:
   ```text
   * * * * 1-5 root echo "Hola" > /dev/tty6
   ```
4. Se guarda el archivo.
5. Al conmutar al terminal `tty6` (mediante `Ctrl+Alt+F6`), se comprueba que el mensaje `"Hola"` aparece impreso cada minuto, ya que `root` sí dispone de permisos de escritura sobre `/dev/tty6`.

---

#### 36. Cree una tarea no recurrente para que se ejecute el próximo 10 de diciembre a las 02.30. Dicha tarea consistirá en apagar el sistema ("poweroff") sin avisar previamente. Indique qué es lo que ha hecho y como qué usuario (normal o root) lo ha hecho.

**Respuesta:**  
- **Usuario:** Debe ejecutarse obligatoriamente como **root**, ya que la orden `poweroff` requiere privilegios de administración del sistema para alterar el estado de la máquina.
- **Pasos dados:** Como superusuario (`root`), se utiliza el comando `at` indicando la hora y fecha:
  ```bash
  at 02:30 10.12.26
  ```
  *(O usando el formato alternativo: `at 02:30 Dec 10`)*.  
  En el prompt interactivo de `at>` se teclea la orden `poweroff` y se finaliza con `Ctrl-D` (`<EOT>`):
  ```text
  at> poweroff
  at> <EOT>
  ```

---

#### 37. Compruebe que la tarea se ha programado correctamente y eliminela (no vaya a ser que interfiera con su trabajo en el futuro). Indique qué pasos ha dado.

**Respuesta:**  
Pasos realizados como `root`:
1. Consultar la cola de tareas programadas de `at` para comprobar que está activa y obtener su número de identificación (*job ID*):
   ```bash
   atq
   ```
2. Eliminar la tarea de la cola mediante `atrm` indicando su identificador (suponiendo que fue asignado el identificador `1`):
   ```bash
   atrm 1
   ```
3. Verificar que la tarea ha sido retirada ejecutando de nuevo:
   ```bash
   atq
   ```
   Comprobando que la orden no devuelve ninguna tarea pendiente.