```txt
#                                                                                                             
#    @@@  @@@   @@@@@@    @@@@@@@  @@@  @@@     @@@@@@@  @@@  @@@  @@@@@@@@     @@@@@@@    @@@@@@   @@@  @@@  
#    @@@  @@@  @@@@@@@@  @@@@@@@@  @@@  @@@     @@@@@@@  @@@  @@@  @@@@@@@@     @@@@@@@@  @@@@@@@@  @@@  @@@  
#    @@!  @@@  @@!  @@@  !@@       @@!  !@@       @@!    @@!  @@@  @@!          @@!  @@@  @@!  @@@  @@!  !@@  
#    !@!  @!@  !@!  @!@  !@!       !@!  @!!       !@!    !@!  @!@  !@!          !@   @!@  !@!  @!@  !@!  @!!  
#    @!@!@!@!  @!@!@!@!  !@!       @!@@!@!        @!!    @!@!@!@!  @!!!:!       @!@!@!@   @!@  !@!   !@@!@!   
#    !!!@!!!!  !!!@!!!!  !!!       !!@!!!         !!!    !!!@!!!!  !!!!!:       !!!@!!!!  !@!  !!!    @!!!    
#    !!:  !!!  !!:  !!!  :!!       !!: :!!        !!:    !!:  !!!  !!:          !!:  !!!  !!:  !!!   !: :!!   
#    :!:  !:!  :!:  !:!  :!:       :!:  !:!       :!:    :!:  !:!  :!:          :!:  !:!  :!:  !:!  :!:  !:!  
#    ::   :::  ::   :::   ::: :::   ::  :::        ::    ::   :::   :: ::::      :: ::::  ::::: ::   ::  :::  
#     :   : :   :   : :   :: :: :   :   :::        :      :   : :  : :: ::      :: : ::    : :  :    :   ::   
#                                                                                                             

===============================================================================
                       Hack The Box — Walkthrough Repository
===============================================================================


----[ 0x01 ]  ABSTRACT
-------------------------------------------------------------------------------
Benvenutə nel mio archivio underground di walkthrough per le macchine
Hack The Box (HTB). Qui trovi report tecnici in Markdown, strutturati
per macchina, con spiegazioni passo-passo, comandi chiave e flag.

Formato: stile Phrack (sezioni numerate, separatori, toni asciutti).


----[ 0x02 ]  STRUTTURA DEL REPO
-------------------------------------------------------------------------------
Ogni file `.md` è un writeup completo di una singola macchina HTB:

    <MachineName>.md     # walkthrough tecnico (enum → exploit → privesc → root)

----[ 0x03 ]  ELENCO WALKTHROUGH (AGGIORNATO)
-------------------------------------------------------------------------------
| Macchina     | Difficoltà | Categorie                          | Link                             |
|--------------|------------|------------------------------------|----------------------------------|
| CODE TWO     | Easy       | Web, JS2Py Escape, RCE, PrivEsc    | [CODE_TWO.md](./CodeTwo.md)      |
| CODE         | Easy       | Web, PrivEsc                       | [CODE.md](./Code.md)             |
| TITANIC      | Easy       | Web, LFI, RCE                      | [TITANIC.md](./Titanic.md)       |
| PLANNING     | Medium     | Web, Grafana, Docker, PrivEsc      | [PLANNING.md](./Planning.md)     |
| NOCTURNAL    | Medium     | Web, File Inclusion, RCE, PrivEsc  | [NOCTURNAL.md](./Nocturnal.md)   |
| DOG          | Medium     | Web, CMS, RCE, PrivEsc             | [DOG.md](./Dog.md)               |


----[ 0x04 ]  NOTE PERSONALI
-------------------------------------------------------------------------------
Questo archivio consolida quanto imparo e offre tracce ripetibili per chi
studia offensive security. Obiettivo: chiarezza operativa, rigore tecnico,
zero fumo. Dove possibile: spiegazione della root cause, non solo i comandi.


----[ 0x05 ]  DISCLAIMER
-------------------------------------------------------------------------------
Tutti i contenuti sono per scopi educativi ed ethical hacking. Utilizza le
informazioni responsabilmente e solo su sistemi per i quali hai autorizzazione.
Nessuna responsabilità per usi impropri.


----[ 0x06 ]  CREDITS & CONTATTI
-------------------------------------------------------------------------------
Autore:    NoFlyFre
GitHub:    https://github.com/NoFlyFre


----[ 0x07 ]  TAG / KEYWORDS
-------------------------------------------------------------------------------
#security  #htb  #walkthrough  #pentest  #NoFlyFre  #hacking  #writeup
#privesc   #web  #rce  #lfi  #docker  #grafana  #js2py  #backup


----[ 0x08 ]  LINGUA
-------------------------------------------------------------------------------
🇮🇹  ITA (Italiano)


===============================================================================
            .: Knowledge, Skillz, Root :.    .: Stay curious! :.
===============================================================================
```
