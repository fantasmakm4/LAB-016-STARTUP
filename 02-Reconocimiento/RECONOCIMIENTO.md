# Reconocimiento y enumeración — LAB-016 Startup

## Nmap

```bash
nmap -Pn --top-ports 100 -T4 10.64.180.147
nmap -Pn -sV -sC -p21,22,80 -T4 10.64.180.147
```

| Puerto | Servicio | Versión | Observación |
|---|---|---|---|
| 21/tcp | FTP | vsftpd 3.0.3 | Login anónimo; carpeta `ftp/` escribible |
| 22/tcp | SSH | OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 | Servicio disponible para acceso válido |
| 80/tcp | HTTP | Apache 2.4.18 (Ubuntu) | Sitio “Maintenance” y directorio `/files/` |

## FTP

- Login anónimo permitido.
- Archivos en la raíz: `important.jpg` (251631 bytes) y `notice.txt` (208 bytes).
- Directorio `ftp/` marcado como escribible.
- `important.jpg` era una imagen PNG, 735 × 458; ExifTool no mostró datos útiles.
- `notice.txt` hablaba de memes de Among Us y mencionaba a Maya.

## HTTP

```bash
gobuster dir -u http://10.64.180.147 -w /usr/share/wordlists/dirb/common.txt
curl -sS http://10.64.180.147/files/
```

Gobuster halló `/files/` con estado 301. El índice `/files/` listaba `ftp/`, `important.jpg` y `notice.txt`. El contenido subido en FTP podía solicitarse por HTTP bajo `/files/ftp/`.

## Archivos de interés descubiertos tras acceso web

- `/recipe.txt`: pista narrativa asociada a una pregunta de la sala; la respuesta se omite.
- `/incidents/suspicious.pcapng`: captura de red accesible como `www-data`.
- `/home/lennie/scripts/planner.sh`: script que invoca `/etc/print.sh`.
- `/etc/print.sh`: ejecutable modificable por `lennie`.

## Precaución de publicación

No publicar contraseña recuperada del PCAP, valores de flags, ni el PCAP íntegro. La IP de la sala es dinámica y corresponde solo a esta sesión.
