# Informe técnico — LAB-016 Startup

## Alcance y resultado

Sala autorizada de TryHackMe Startup. IP de la instancia en esta sesión: `10.64.180.147` (temporal). El objetivo fue enumerar servicios, obtener acceso inicial y escalar privilegios. Se confirmó ejecución como `www-data`, acceso SSH como `lennie` y shell efectiva de root. Se recuperó `user.txt`; sus valores y las credenciales observadas se omiten. No se mostró el valor de `root.txt` ni se verificó el envío de las respuestas del cuestionario.

## Plataforma y sistema

- Plataforma: TryHackMe
- Sistema operativo remoto confirmado por SSH: Ubuntu 16.04.7 LTS
- Kernel informado por el sistema: Linux 4.4.0-190-generic x86_64
- Dificultad: no verificada durante esta sesión
- CVE: no se confirmó una CVE aplicable

## Reconocimiento

Escaneo inicial de los 100 puertos TCP más comunes:

```bash
nmap -Pn --top-ports 100 -T4 10.64.180.147
```

Puertos observados: 21/tcp FTP, 22/tcp SSH y 80/tcp HTTP.

Detección de versiones y scripts por defecto:

```bash
nmap -Pn -sV -sC -p21,22,80 -T4 10.64.180.147
```

| Puerto | Servicio | Versión observada |
|---|---|---|
| 21/tcp | FTP | vsftpd 3.0.3 |
| 22/tcp | SSH | OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 |
| 80/tcp | HTTP | Apache 2.4.18 (Ubuntu) |

FTP permitió autenticación anónima. La respuesta de `STAT` marcó el directorio `ftp` como escribible; también aparecieron `important.jpg` y `notice.txt`.

## Enumeración de FTP y web

`notice.txt` mencionaba archivos meme en el recurso y señalaba a Maya como sospechosa. `important.jpg`, descargado con ese nombre, resultó ser PNG de 735 × 458 según ExifTool. No se hallaron pistas útiles en sus metadatos; la imagen era un meme de Among Us.

Gobuster encontró `/files/` (301) y el índice web mostró `ftp/`, `important.jpg` y `notice.txt`. La carpeta FTP escribible era accesible también por HTTP en `/files/ftp/`.

## Acceso inicial como www-data

Se creó y subió un pequeño webshell PHP al directorio FTP. La ejecución por HTTP confirmó `uid=33(www-data)`. La variante final recibía el comando por el parámetro `cmd`; los detalles están en el registro de comandos, sin secretos.

La raíz del sistema contenía `/incidents/suspicious.pcapng` y `/recipe.txt`. El segundo archivo aportaba contexto para una pregunta de la sala; no se publica la respuesta. La captura fue copiada a la carpeta publicada por HTTP, descargada a Parrot y analizada localmente con TShark. El flujo TCP 7 por el puerto 4444 mostró una sesión de shell donde el usuario `www-data` intentaba entrar a `lennie` con una contraseña. El valor de la contraseña se omite.

## Acceso SSH y user flag

Se inició sesión por SSH como `lennie`. Se verificó el usuario con `id` y `whoami`, y se encontró `user.txt` en su directorio personal. El valor de la flag no se publica.

## Escalada a root

`/home/lennie/scripts/planner.sh` contenía una llamada a `/etc/print.sh`. El segundo archivo era ejecutable y propiedad de `lennie`, que podía modificarlo. El daemon cron estaba activo; la tarea programada no figuraba en el `/etc/crontab` global ni en los archivos visibles de `/etc/cron.d`.

Se reemplazó `/etc/print.sh` con instrucciones para copiar Bash a `/tmp/rootbash` y asignar permisos SUID. Tras esperar 60 segundos, `ls -l` mostró un binario `/tmp/rootbash` propiedad de root con el bit SUID. Al ejecutar `/tmp/rootbash -p`, `whoami` devolvió `root` y `id` confirmó `euid=0(root)`.

## Hallazgos y mitigación

| Hallazgo | Impacto | Recomendación |
|---|---|---|
| FTP anónimo escribible | Carga de contenido controlado por atacante | Deshabilitar anónimo y aplicar permisos mínimos |
| Archivos cargados ejecutables por Apache | Ejecución remota como `www-data` | Separar almacenamiento y bloquear interpretación PHP en uploads |
| Captura PCAP accesible y credencial en claro | Acceso SSH como `lennie` | Restringir evidencias, cifrar protocolos y rotar secretos |
| Script de cron modificable por usuario | Ejecución de código como root | Scripts root-owned no modificables por usuarios; revisar cron |
| Creación de binario SUID privilegiado | Escalada a root | Evitar SUID innecesario y alertar ante cambios de permisos |

## Evidencias y privacidad

La carpeta `05-Evidencias/privado/` está excluida de Git. No se deben publicar credenciales, contenidos de PCAP, capturas con flags ni los valores de `user.txt` o `root.txt`. La carpeta pública contiene solamente evidencia narrativa sin datos sensibles.
