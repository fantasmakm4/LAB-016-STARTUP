# Evidencias públicas — LAB-016 Startup

Se documentan aquí únicamente resultados que no exponen secretos:

- Nmap identificó FTP/21, SSH/22 y HTTP/80 con las versiones anotadas en el informe.
- FTP anónimo permitió escritura en `ftp/`, servida también por HTTP.
- El webshell devolvió `uid=33(www-data)`.
- La captura de incidentes expuso una credencial SSH reutilizada; su valor y el PCAP se mantienen privados.
- Se confirmó acceso como `lennie` y después `euid=0(root)` mediante el script de cron modificable.
- Se recuperó `user.txt`; los valores de todas las flags se omiten.

Las capturas, el PCAP, las credenciales y las flags deben guardarse solo en `05-Evidencias/privado/`, carpeta excluida de Git.
