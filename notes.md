## ⚙️ Cosa deve fare il tuo progetto

Il tuo programma `./webserv [config_file]` deve:

1. **Leggere un file di configurazione** (come fa Nginx) che dice su quali porte ascoltare, quali directory servire, quali metodi (GET, POST, DELETE) accettare, ecc.
2. **Aprire un socket** (cioè una porta di comunicazione di rete) e mettersi in ascolto.
3. **Aspettare richieste dai client** (browser o telnet).
4. **Leggere le richieste** (che arrivano con il protocollo HTTP).
5. **Rispondere in modo corretto** con un messaggio HTTP (pagina HTML, immagine, file caricato, codice di errore, ecc.).
6. **Gestire più connessioni insieme** senza bloccarsi (qui entra in gioco `poll()`, `select()`, `epoll`, o `kqueue`).
7. **Supportare almeno 3 metodi HTTP**:

   * **GET** → scaricare/mostrare file (es. `index.html`).
   * **POST** → inviare dati al server (per esempio un form o upload di file).
   * **DELETE** → cancellare un file dal server.
8. **Supportare CGI** → ad esempio eseguire uno script PHP o Python e restituire l’output come risposta.
9. **Non crashare mai** → deve resistere a test intensivi e gestire disconnessioni, richieste sbagliate, ecc.

---

## 🔧 Funzioni che userai

Il progetto ti **obbliga** a usare funzioni di basso livello del sistema operativo (Linux/macOS).
Ecco una spiegazione semplificata delle più importanti:

### 🛠️ Per la rete (socket)

* **socket()** → crea un "punto di comunicazione" (la porta del server).
* **bind()** → lega il socket a un indirizzo IP e una porta (es. `127.0.0.1:8080`).
* **listen()** → mette il server in ascolto.
* **accept()** → accetta una connessione da un client (browser).
* **connect()** → (lato client) connessione a un server.
* **send()/write()** → manda dati al client.
* **recv()/read()** → riceve dati dal client.
* **close()** → chiude una connessione.

### 🛠️ Per gestire più connessioni insieme

* **poll() / select() / epoll / kqueue** → ti permettono di controllare contemporaneamente tanti socket e vedere quali sono pronti per leggere o scrivere, senza bloccare il programma.

### 🛠️ Per file e directory

* **open(), read(), write(), close()** → lavorano con i file.
* **opendir(), readdir(), closedir()** → per leggere cartelle (directory listing).
* **stat()** → controlla informazioni su file (esiste? è directory? dimensione?).

### 🛠️ Per CGI e processi

* **fork()** → crea un processo figlio (da usare solo per eseguire CGI).
* **execve()** → esegue un altro programma (es. PHP o Python).
* **pipe() / dup2()** → collegano input/output tra processi (utile per parlare con il CGI).
* **waitpid()** → aspetta che un processo figlio finisca.

### 🛠️ Altre utilità

* **fcntl()** → mette i socket in modalità non-bloccante.
* **setsockopt()** → opzioni sui socket (tipo `SO_REUSEADDR`).
* **signal(), kill()** → gestire segnali (tipo `CTRL+C`).

---

## 🔄 Flusso di lavoro del server (semplificato)

1. Leggi la configurazione (porta, host, root, ecc.).
2. Apri un socket e fai `bind() + listen()`.
3. Con `poll()` tieni sotto controllo tutte le connessioni.
4. Quando arriva una richiesta:

   * Leggi l’header HTTP (`GET /index.html HTTP/1.1`).
   * Decidi cosa fare (GET → apri file, POST → salva dati, DELETE → cancella file).
   * Prepara una risposta HTTP (`HTTP/1.1 200 OK` + contenuto).
   * Invia la risposta con `send()`.
5. Se è CGI, fai `fork()`, `execve()` e con `pipe()` gestisci input/output.
6. Torna ad ascoltare senza mai bloccarti.

---

## 📂 Esempio minimo

Se il browser fa:

```
GET /index.html HTTP/1.1
Host: localhost:8080
```

Il tuo server deve rispondere con qualcosa tipo:

```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 125

<html>
  <head><title>Hello</title></head>
  <body><h1>Benvenuto nel mio webserver!</h1></body>
</html>
```

---
