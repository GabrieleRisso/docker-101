# Utilizzo di Docker Compose

## Introduzione
Nei nostri post precedenti, abbiamo esplorato come i container Docker e i Dockerfile possono essere utilizzati per pacchettizzare le applicazioni in modo ordinato. Ma cosa succede quando le tue applicazioni diventano più complesse, con componenti multipli e configurazioni intricate? Immagina di costruire un negozio online, dove hai microservizi separati per il frontend, backend, elaborazione dei pagamenti, gestione degli ordini e analisi. Ognuno di questi microservizi potrebbe essere sviluppato utilizzando diversi linguaggi di programmazione e tecnologie, e tutti devono essere costruiti, pacchettizzati e configurati correttamente.

È qui che Docker Compose viene in soccorso. È uno strumento potente specificamente progettato per gestire applicazioni che vengono eseguite in più container Docker. Puoi definire l'intero stack applicativo in un singolo file YAML, incluso ogni microservizio, la sua configurazione e come interagiscono tra loro.

## Caratteristiche Essenziali

Le caratteristiche essenziali di Docker Compose possono essere raggruppate in tre categorie:

1. **Isolamento**: Immagina di eseguire più istanze della tua applicazione, ognuna completamente isolata dalle altre. Docker Compose rende questo possibile, permettendo di replicare l'intero stack applicativo su vari ambienti.

2. **Gestione dei Dati Stateful**: Se la tua applicazione necessita di memorizzare dati su disco, come un database, Docker Compose si occupa di gestire i volumi di dati associati ai tuoi container.

3. **Design Iterativo**: Docker Compose lavora con una configurazione chiara che definisce tutti i container nella tua applicazione. Puoi facilmente aggiungere nuovi container a questa configurazione senza interrompere quelli esistenti.

## CLI di Docker Compose

Docker Compose lavora in tandem con Docker Engine per orchestrare applicazioni multi-container. Utilizza uno strumento da riga di comando chiamato `docker-compose` per comunicare con l'Engine.

### Comandi CLI Principali

1. **docker-compose up**: Inizializza e avvia i container come definito nel file di configurazione.
   ```bash
   docker-compose up -d  # Avvia in modalità detached
   ```

2. **docker-compose ps**: Fornisce uno snapshot dei container e del loro stato attuale.
   ```bash
   docker-compose ps
   ```

3. **docker-compose down**: Ferma e rimuove tutte le risorse.
   ```bash
   docker-compose down
   ```

## File Docker Compose

Il file Docker Compose (docker-compose.yml) è strutturato in quattro sezioni principali:

### 1. Nome
```yaml
name: myapp
```

### 2. Servizi
```yaml
services:
    server:
        image: nginx:latest
        ports:
            - 8080:80
```

### 3. Reti
```yaml
networks:
    - front-tier
    - back-tier
```

### 4. Volumi
```yaml
volumes:
    db-data:
```

## Esempio Pratico: Creazione di un Web Server

### Struttura del Progetto
```
compose-server/
├── init/
│   ├── prepare.sh
│   └── Dockerfile
└── docker-compose.yaml
```

### Script di Preparazione (prepare.sh)
```bash
#!/usr/bin/env sh

if [ -d "data" ]; then
    rm -rf data
fi

mkdir data
touch data/index.html

echo "<h1>Benvenuto in Docker Compose!</h1>" >> data/index.html
echo "<img src='https://www.docker.com/wp-content/uploads/2021/10/Moby-logo-sm.png' />" >> data/index.html
```

### Dockerfile
```dockerfile
FROM busybox
ADD prepare.sh /usr/bin/prepare.sh
RUN chmod +x /usr/bin/prepare.sh
ENTRYPOINT ["sh", "/usr/bin/prepare.sh"]
```

### docker-compose.yaml
```yaml
name: "compose-server"

services:
  init:
    build:
      context: ./init
    volumes:
      - static:/data

  server:
    image: nginx
    volumes:
      - static:/usr/share/nginx/html
    ports:
      - "8080:80"

volumes:
  static:
```

## Comandi per l'Esecuzione

1. Avviare l'applicazione:
   ```bash
   docker-compose up --detach
   ```

2. Verificare lo stato:
   ```bash
   docker-compose ps
   ```

3. Fermare e rimuovere tutto:
   ```bash
   docker-compose down
   ```

## Conclusione

Docker Compose è uno strumento fondamentale per la gestione di applicazioni multi-container. La sua capacità di definire l'intera infrastruttura come codice, gestire lo stato e facilitare lo sviluppo iterativo lo rende indispensabile nel moderno sviluppo software. La combinazione di un file di configurazione chiaro con comandi CLI potenti permette di gestire facilmente anche le applicazioni più complesse.

Con Docker Compose, puoi:
- Definire l'intera infrastruttura in un singolo file
- Gestire facilmente le dipendenze tra servizi
- Mantenere la persistenza dei dati
- Scalare i servizi secondo necessità
- Semplificare il processo di sviluppo e deployment

La padronanza di Docker Compose è essenziale per qualsiasi sviluppatore o DevOps engineer che lavora con container Docker.