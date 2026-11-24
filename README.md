![Portada profesional FANTASMA — LAB-016 Startup. Infografía panorámica con ilustración cyberpunk de una calavera demoníaca encapuchada, cuernos, ojos rojos, alas de murciélago, luna roja y portátil. Paleta negro, carmesí, cian, ámbar y verde. Paneles legibles muestran TryHackMe, Ubuntu 16.04.7, IP dinámica de sesión, dificultad no verificada, servicios y versiones reales, cadena FTP anónimo a webshell PHP, análisis PCAP, SSH como lennie y escalada cron con script modificable a root. Incluye herramientas y mitigaciones; confirma root y omite respuestas, credenciales y flags.](04-Imagenes/LAB-016-STARTUP.png)

# 😈 FANTASMA | LAB-016 — STARTUP

**TryHackMe | FANTASMA LAB-016 STARTUP | Parrot OS, Nmap, FTP anónimo escribible, Gobuster, webshell PHP, análisis PCAP, SSH como lennie, escalada cron a ROOT | Máquina comprometida, ROOT confirmado | User flag recuperada; valores omitidos, root.txt no mostrado**

## Descripción de la portada

Infografía panorámica de **LAB-016 Startup** con el avatar FANTASMA —calavera demoníaca encapuchada, cuernos, ojos rojos, alas de murciélago, luna roja y portátil— sobre un entorno urbano tecnológico. La portada especifica plataforma, máquina, IP dinámica, sistema operativo, dificultad no verificada y estado. Incluye versiones de FTP/21 (vsftpd 3.0.3), SSH/22 (OpenSSH 7.2p2 Ubuntu 4ubuntu2.10) y HTTP/80 (Apache 2.4.18); hallazgos y ausencia de CVE confirmada; fases, comandos realmente ejecutados, herramientas, resultados, documentación, estructura del repositorio, referencia TryHackMe y mitigaciones. La ruta muestra FTP anónimo escribible, ejecución del webshell PHP como `www-data`, análisis de `suspicious.pcapng`, SSH como `lennie` y escalada por cron y `/etc/print.sh` hasta root con Bash SUID. Se confirma root y la recuperación de `user.txt`; no se mostró `root.txt`. No se publican contraseñas, respuestas ni valores de flags.

## Información del laboratorio

- Plataforma: TryHackMe
- Máquina: Startup
- Sala: [tryhackme.com/room/startup](https://tryhackme.com/room/startup)
- IP observada durante esta sesión: `10.64.180.147` (dinámica)
- Sistema operativo confirmado por SSH: Ubuntu 16.04.7 LTS, kernel `4.4.0-190-generic`
- Dificultad: no verificada durante esta sesión
- Estado: acceso efectivo como root confirmado mediante `whoami` y `id`
- User flag: recuperada; valor omitido
- Root flag: no se mostró en la conversación; no se afirma su recuperación
- Cuestionario de TryHackMe: envío de respuestas no verificado
- CVE: no se confirmó una CVE aplicable

## Descripción

El laboratorio presenta servicios FTP, SSH y HTTP. La ruta observada combinó un directorio FTP anónimo escribible y servido también por HTTP, ejecución de comandos PHP, una captura de tráfico con credenciales expuestas y una escalada local causada por una tarea programada que ejecutaba un script modificable por `lennie`.

## Ruta de compromiso

1. **Reconocimiento:** Nmap identificó FTP, SSH y HTTP.
2. **Enumeración:** FTP permitió acceso anónimo y expuso `important.jpg`, `notice.txt` y un directorio escribible. Gobuster encontró `/files/`, cuyo listado reveló el subdirectorio `ftp/`.
3. **Acceso inicial:** se subió un webshell PHP al directorio FTP y se invocó por HTTP. La ejecución confirmó el usuario `www-data`.
4. **Análisis de la captura:** `/incidents/suspicious.pcapng` fue copiado al directorio web expuesto y descargado. TShark permitió reconstruir el flujo TCP 7; la captura exponía credenciales de `lennie`. Los valores se omiten.
5. **Acceso de usuario:** SSH permitió entrar como `lennie`; se recuperó `user.txt`.
6. **Escalada:** `scripts/planner.sh` invocaba `/etc/print.sh`, propiedad de `lennie` y modificable por esa cuenta. Tras ejecutarse la tarea programada como root, apareció `/tmp/rootbash` con bit SUID. `/tmp/rootbash -p` abrió una shell con `euid=0`.

## Servicios identificados

| Puerto | Servicio | Versión observada |
|---|---|---|
| 21/tcp | FTP | vsftpd 3.0.3; acceso anónimo permitido |
| 22/tcp | SSH | OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 |
| 80/tcp | HTTP | Apache httpd 2.4.18 (Ubuntu) |

El directorio `/files/` permitía listado HTTP. Dentro se observó `ftp/`, además de `important.jpg` y `notice.txt`. El archivo con extensión `.jpg` era realmente PNG de 735 × 458; no se encontraron pistas en sus metadatos. La nota FTP mencionaba memes de Among Us y a Maya como sospechosa.

## Hallazgos

- **Escritura anónima en FTP:** la carpeta `ftp/` admitía cargas sin autenticación.
- **Ejecución remota de comandos en PHP:** el archivo subido ejecutaba `shell_exec` con un parámetro controlado por la petición.
- **Exposición de datos sensibles:** la captura PCAP accesible al usuario web contenía credenciales reutilizables de SSH. No se reproducen.
- **Escalada por tarea programada:** el flujo `planner.sh` → `/etc/print.sh` permitía a `lennie` influir en código ejecutado como root; el binario SUID generado confirmó la escalada.
- **CVE:** ninguna CVE se verificó como causa necesaria de esta cadena.

## Resultados

- Acceso web como `www-data` confirmado.
- Acceso SSH como `lennie` confirmado.
- User flag recuperada; valor no publicado.
- Acceso efectivo como root confirmado (`whoami` devolvió `root`; `id` mostró `euid=0`).
- No se mostró el valor de `root.txt` ni se verificó el envío de todas las respuestas en TryHackMe.

## Herramientas utilizadas

Nmap, Gobuster, curl, ExifTool, TShark, `strings`, SSH, `printf`, `shell_exec`, `grep`, `ls`, `cat`, `id`, `whoami` y cron.

El registro depurado de comandos está en [06-Comandos/COMANDOS.md](06-Comandos/COMANDOS.md). No contiene contraseñas ni valores de flags.

## Recomendaciones de seguridad

1. Deshabilitar el acceso FTP anónimo y retirar permisos de escritura a usuarios no autenticados.
2. No servir directorios de intercambio con listado público; impedir que archivos cargados se ejecuten como PHP.
3. Eliminar la ejecución de comandos del sistema desde parámetros HTTP; validar entradas y evitar pasar datos a un shell.
4. Proteger capturas de red y retirar credenciales en claro; rotar credenciales expuestas y evitar su reutilización.
5. No permitir que usuarios de bajo privilegio modifiquen scripts que ejecuta cron como root. Auditar propietario, permisos y rutas completas de cada tarea.
6. Evitar crear binarios SUID durante tareas programadas y monitorizar cambios en archivos privilegiados.

## Estructura del repositorio

- `01-Informes/`: informe técnico del laboratorio.
- `02-Reconocimiento/`: escaneos, servicios y enumeración.
- `03-Explotacion/`: acceso inicial y escalada.
- `04-Imagenes/`: portada FANTASMA.
- `05-Evidencias/publico/`: evidencia descriptiva sin secretos.
- `05-Evidencias/privado/`: espacio local ignorado por Git para capturas y flags.
- `06-Comandos/`: comandos utilizados, depurados para publicación.

---
**FANTASMA | LAB-016 | STARTUP | ROOT CONFIRMADO**
