```txt

    @@@  @@@   @@@@@@    @@@@@@@  @@@  @@@     @@@@@@@  @@@  @@@  @@@@@@@@     @@@@@@@    @@@@@@   @@@  @@@
    @@@  @@@  @@@@@@@@  @@@@@@@@  @@@  @@@     @@@@@@@  @@@  @@@  @@@@@@@@     @@@@@@@@  @@@@@@@@  @@@  @@@
    @@!  @@@  @@!  @@@  !@@       @@!  !@@       @@!    @@!  @@@  @@!          @@!  @@@  @@!  @@@  @@!  !@@
    !@!  @!@  !@!  @!@  !@!       !@!  @!!       !@!    !@!  @!@  !@!          !@   @!@  !@!  @!@  !@!  @!!
    @!@!@!@!  @!@!@!@!  !@!       @!@@!@!        @!!    @!@!@!@!  @!!!:!       @!@!@!@   @!@  !@!   !@@!@!
    !!!@!!!!  !!!@!!!!  !!!       !!@!!!         !!!    !!!@!!!!  !!!!!:       !!!@!!!!  !@!  !!!    @!!!
    !!:  !!!  !!:  !!!  :!!       !!: :!!        !!:    !!:  !!!  !!:          !!:  !!!  !!:  !!!   !: :!!
    :!:  !:!  :!:  !:!  :!:       :!:  !:!       :!:    :!:  !:!  :!:          :!:  !:!  :!:  !:!  :!:  !:!
    ::   :::  ::   :::   ::: :::   ::  :::        ::    ::   :::   :: ::::      :: ::::  ::::: ::   ::  :::
     :   : :   :   : :   :: :: :   :   :::        :      :   : :  : :: ::      :: : ::    : :  :    :   ::

```

# Hack The Box — Walkthrough Repository

Stile asciutto, taglio tecnico, vibe Phrack. Writeup in Markdown per macchine HTB, con focus su root cause e catena di exploit riproducibile.

## Indice

- [0x01 ABSTRACT](#0x01--abstract)
- [0x02 STRUTTURA](#0x02--struttura-del-repo)
- [0x03 ELENCO](#0x03--elenco-walkthrough-aggiornato)
- [0x04 USO](#0x04--uso)
- [0x05 NOTE](#0x05--note-personali)
- [0x06 DISCLAIMER](#0x06--disclaimer)
- [0x07 CREDITI](#0x07--credits--contatti)
- [0x08 LINGUA](#0x08--lingua)

## ----[ 0x01 ] ABSTRACT

Archivio di walkthrough per Hack The Box. Ogni documento tratta enum → exploit → privesc → root, con spiegazioni essenziali e comandi pronti all’uso.

## ----[ 0x02 ] STRUTTURA DEL REPO

Ogni file `.md` è il writeup di una singola macchina HTB:

    <MachineName>.md     # walkthrough tecnico (enum → exploit → privesc → root)

## ----[ 0x03 ] ELENCO WALKTHROUGH (AGGIORNATO)

| Macchina  | Difficoltà | Categorie                         | Link                           |
| --------- | ---------- | --------------------------------- | ------------------------------ |
| Code Two  | Easy       | Web, JS2Py Escape, RCE, PrivEsc   | [CodeTwo.md](./CodeTwo.md)     |
| Code      | Easy       | Web, PrivEsc                      | [Code.md](./Code.md)           |
| Titanic   | Easy       | Web, LFI, RCE                     | [Titanic.md](./Titanic.md)     |
| Planning  | Medium     | Web, Grafana, Docker, PrivEsc     | [Planning.md](./Planning.md)   |
| Nocturnal | Medium     | Web, File Inclusion, RCE, PrivEsc | [Nocturnal.md](./Nocturnal.md) |
| Dog       | Medium     | Web, CMS, RCE, PrivEsc            | [Dog.md](./Dog.md)             |

## ----[ 0x04 ] USO

- Leggi il `.md` della macchina d’interesse e segui le sezioni numerate.
- I blocchi comando sono pensati per copy/paste; adatta IP/interfacce.
- Le sezioni “Root Cause” chiariscono il perché dietro l’exploit, non solo il come.

## ----[ 0x05 ] NOTE PERSONALI

Focus su chiarezza operativa e rigore. Preferenza per spiegare le cause profonde, documentare gli assunti e rendere la catena d’attacco ripetibile.

## ----[ 0x06 ] DISCLAIMER

Materiale a scopo educativo ed ethical hacking. Usa le informazioni solo su sistemi per cui hai autorizzazione. L’autore declina ogni responsabilità per usi impropri.

## ----[ 0x07 ] CREDITS & CONTATTI

- Autore: NoFlyFre
- GitHub: https://github.com/NoFlyFre

## ----[ 0x08 ] LINGUA

🇮🇹 Italiano

## ----[ TAG / KEYWORDS ]

`#security` `#htb` `#walkthrough` `#pentest` `#hacking` `#writeup` `#privesc` `#web` `#rce` `#lfi` `#docker` `#grafana` `#js2py` `#backup`

<sub>Last update: 25/11/2025</sub>
