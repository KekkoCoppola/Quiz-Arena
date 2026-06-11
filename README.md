<p align="center">
  <img src="resources/img/QuizArenaLogo.png" alt="Quiz Arena Logo" width="300" style="border-radius: 10px; box-shadow: 0 4px 15px rgba(0,0,0,0.15);">
</p>

<h1 align="center">Quiz Arena</h1>

<p align="center">
  <strong>Il motore di gioco a quiz multiplayer locale e in tempo reale per Raspberry Pi.</strong><br>
  Un'alternativa a Kahoot completamente offline, autonoma e self-hosted, progettata per eventi, aule scolastiche, pub e fiere.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white" alt="NodeJS">
  <img src="https://img.shields.io/badge/Socket.io-010101?style=for-the-badge&logo=socketdotio&logoColor=white" alt="Socket.io">
  <img src="https://img.shields.io/badge/SQLite-07405E?style=for-the-badge&logo=sqlite&logoColor=white" alt="SQLite">
  <img src="https://img.shields.io/badge/Raspberry%20Pi-A22846?style=for-the-badge&logo=raspberrypi&logoColor=white" alt="Raspberry Pi">
</p>

---

## 📌 Cos'è Quiz Arena?

**Quiz Arena** è una piattaforma di quiz interattivi multiplayer in tempo reale che funziona **completamente offline**. Viene eseguita a bordo di un **Raspberry Pi** configurato come Access Point Wi-Fi isolato. 

I giocatori possono connettersi istantaneamente dal proprio smartphone inquadrando un **QR Code** o navigando all'indirizzo IP locale, senza installare alcuna applicazione o aver bisogno di una connessione internet attiva. L'host controlla la partita, le domande e la classifica in tempo reale tramite una dashboard dedicata.

---

## 🚀 Caratteristiche Principali

- **🔌 100% Offline-First:** Non necessita di connessione ad internet né di router esterni. Il Raspberry Pi crea la propria rete Wi-Fi dedicata.
- **⚡ Real-Time WebSockets:** Interazioni istantanee a bassissima latenza tra host e giocatori grazie a **Socket.io**.
- **📱 Zero Barriere d'Ingresso:** I giocatori scansionano il QR Code generato a schermo e partecipano direttamente dal proprio browser mobile preferito.
- **🎯 Gamification in Stile Kahoot:** Assegnazione dei punteggi dinamica basata sulla velocità e correttezza della risposta.
- **🛠️ Gestione Quiz Flessibile:** Creazione dei quiz da interfaccia amministrativa o importazione massiva da file **CSV/Excel**.
- **🔒 Hardware Lock & Licenza SaaS:** Sistema di protezione e attivazione SaaS a tempo, vincolato al seriale hardware della CPU del Raspberry Pi (`/proc/cpuinfo`) verificato periodicamente tramite le API private di GitHub.
- **📦 Distribuzione Singolo Binario:** Backend offuscato e compilato in un unico file eseguibile per architettura ARM tramite `pkg`, per proteggere il codice sorgente e facilitare il deploy.

---

## 🛠️ Stack Tecnologico

- **Runtime:** Node.js (v18+)
- **Server Web:** Express.js + Socket.io (WebSocket in tempo reale)
- **Database:** SQLite (file-based, integrato e a configurazione zero)
- **Frontend:** HTML5, CSS3 Custom (Vanilla CSS con design responsive moderno), JavaScript Vanilla (SPA leggera)
- **Compilazione & Packaging:** `pkg` (compilazione del backend Node.js in un binario nativo ARM)
- **Sistema Host:** Linux (Debian/Raspbian OS) configurato con `hostapd` e `dnsmasq` per l'Access Point locale

---

## 🌐 Architettura di Rete

Quiz Arena non richiede infrastruttura esterna. Il Raspberry Pi funge da server e da router Wi-Fi contemporaneamente:

```text
               +----------------------------------------+
               |  Raspberry Pi (Hotspot "QuizArena")    |
               |             IP: 192.168.4.1            |
               +----------------------------------------+
                                   |
         +-------------------------+-------------------------+
         |                                                   |
         v                                                   v
🌐 Dispositivo Host                               📱 Smartphone Giocatori
   Apre: http://192.168.4.1/host                     Aprono: http://192.168.4.1 (o QR Code)
   - Avvia e controlla i quiz                        - Inseriscono il nickname
   - Mostra domande, grafici e classifiche           - Inviano le risposte in tempo reale
```

---

## 🔄 Flusso di Gioco (State Machine)

Il flusso di una partita segue una macchina a stati solida e sincronizzata in tempo reale:

```text
   [ LOBBY ] ➔ I giocatori entrano e scelgono un nickname.
       │
       ▼
 [ QUESTION ] ➔ Domanda a schermo con countdown; risposte abilitate su mobile.
       │
       ▼
  [ RESULTS ] ➔ Mostra la risposta corretta, grafici di risposta e classifica parziale.
       │
       ├─➔ (Se ci sono altre domande, torna a QUESTION)
       ▼
[ FINAL_LEADERBOARD ] ➔ Podio finale dei vincitori e opzione di riavvio.
```

---

## 🗂️ Struttura del Progetto

Il codice è organizzato in modo modulare per separare nettamente le responsabilità di backend, frontend e strumenti interni:

```text
quiz-game/
├── server/
│   ├── index.js              # Entrypoint: verifica licenza ed avvia il server Express/Socket.io
│   ├── license.js            # Modulo licenza: controlli GitHub API + gestione cache locale
│   ├── game.js               # Macchina a stati di gioco e logica WebSocket
│   ├── quiz-manager.js       # Gestore database SQLite per CRUD quiz e importazioni
│   ├── routes/
│   │   ├── host.js           # Endpoint API per pannello amministratore
│   │   └── player.js         # Endpoint API per interfacce giocatori
│   └── db/
│       ├── schema.sql        # Struttura delle tabelle database SQLite
│       └── database.sqlite   # Database SQLite locale autogenerato
├── client/
│   ├── host/                 # Frontend Dashboard di controllo per l'Host
│   │   ├── index.html
│   │   └── app.js
│   └── player/               # Frontend Mobile-Responsive per i Giocatori
│       ├── index.html
│       └── app.js
├── tools/
│   └── generate-license.js   # Script di utility interno per generare e caricare licenze
├── .env                      # Token di accesso GitHub PAT e configurazioni d'ambiente
└── package.json              # Dipendenze e script NPM del progetto
```

---

## 🔑 Flusso del Sistema di Licenza SaaS

Il software integra un meccanismo di licenza a tempo vincolato all'hardware per l'utilizzo commerciale su base abbonamento:

```
[ Raspberry Pi Boot ] ──▶ [ Controlla Connessione Internet ]
                                  │
         ┌────────────────────────┴────────────────────────┐
         ▼ SI                                              ▼ NO
[ Fetch da GitHub Repo Privato ]                  [ Leggi Cache Licenza Locale ]
         │                                                 │
         ├────────────────────────┬────────────────────────┘
         ▼                        ▼
[ Verifica Serial CPU ]  ▶  [ Verifica Scadenza ] ──▶ [ Valida (Grace period: 7gg) ]
                                                            │
                                         ┌──────────────────┴──────────────────┐
                                         ▼ OK                                  ▼ FALLITO / SCADUTO
                                  [ Avvia Quiz Arena ]                  [ Blocca Esecuzione ]
```

1. **GitHub come Backend:** Le licenze sono memorizzate come file JSON nella cartella `licenze/<serial_pi>.json` di un repository privato GitHub (utilizzando il piano gratuito di GitHub).
2. **Validazione Hardware:** Il server legge l'identificativo seriale unico del chip Raspberry Pi leggendo `/proc/cpuinfo`.
3. **Grace Period Offline:** Il sistema richiede una connessione ad internet per il rinnovo della licenza almeno una volta ogni 7 giorni. In caso di mancanza di rete, si appoggia ad una cache locale crittografata.
4. **Costi di Infrastruttura e Manutenzione Zero:** Non sono necessari server cloud dedicati né database SaaS a pagamento. Tutto il database risiede in locale su SQLite, e GitHub gestisce la distribuzione delle licenze a costo zero. L'unico costo fisico del progetto è l'hardware del Raspberry Pi.

---

## 📋 Ordine di Sviluppo Concordato

Il progetto viene sviluppato seguendo un approccio incrementale in 9 step:

1. **Step 1:** `server/license.js` — Integrazione e test chiamate GitHub API + cache locale di fallback.
2. **Step 2:** `server/index.js` — Bootstrap iniziale, aggancio del controllo licenza all'avvio.
3. **Step 3:** `server/game.js` — Logica centrale e macchina a stati del gioco.
4. **Step 4:** `server/quiz-manager.js` + SQLite Schema — Gestione CRUD quiz e import CSV/Excel.
5. **Step 5:** Socket.io Integration — Configurazione ed eventi real-time tra Host e Player.
6. **Step 6:** Frontend Host — UI della dashboard amministrativa e di visualizzazione in sala.
7. **Step 7:** Frontend Player — Interfaccia mobile responsive, pulsanti di risposta e feedback.
8. **Step 8:** `tools/generate-license.js` — Script CLI interno per i developer per creare nuove licenze.
9. **Step 9:** Script di Deploy per Pi — Automazione installazione di `hostapd`, `dnsmasq` e avvio del binario compilato al boot.

---

## 👨‍💻 Autore e License

<div align="center">
  <a href="https://github.com/KekkoCoppola">
    <img src="https://github.com/KekkoCoppola.png" width="80" style="border-radius: 50%; border: 3px solid #38B2AC;" alt="Avatar" />
  </a>
  <br />
  <strong>Francesco Coppola</strong>
  <br />
  <p>Progetto Proprietario - Tutti i Diritti Riservati <br> 📧 fcoppola.dev@gmail.com</p>
</div>

