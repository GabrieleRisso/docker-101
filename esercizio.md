# Tutorial: Configurazione Web Server Nginx - Locale e Docker

## Parte 1: Installazione Nginx su Linux senza Docker

### Prerequisiti
- Sistema operativo Linux (Ubuntu/Debian)
- Accesso root o privilegi sudo

### Installazione e Configurazione Base

1. Aggiornare i repository:
```bash
sudo apt update
sudo apt upgrade -y
```

2. Installare Nginx:
```bash
sudo apt install nginx -y
```

3. Avviare Nginx e abilitarlo all'avvio:
```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

4. Verificare lo stato:
```bash
sudo systemctl status nginx
```

### Creazione Pagina Web

1. Creare una directory per il sito:
```bash
sudo mkdir -p /var/www/mio-sito
```

2. Creare la pagina index.html:
```bash
sudo nano /var/www/mio-sito/index.html
```

3. Inserire il contenuto base (personalizzabile):
```html
<!DOCTYPE html>
<html lang="it">
<head>
    <meta charset="UTF-8">
    <title>Il Mio Sito</title>
</head>
<body>
    <h1>Benvenuto nel mio sito web!</h1>
    <!-- Personalizza questa pagina come preferisci -->
</body>
</html>
```

4. Configurare Nginx:
```bash
sudo nano /etc/nginx/sites-available/mio-sito
```

5. Aggiungere la configurazione base:
```nginx
server {
    listen 80;
    server_name localhost;

    root /var/www/mio-sito;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

6. Abilitare il sito:
```bash
sudo ln -s /etc/nginx/sites-available/mio-sito /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/default  # rimuovere il default
sudo nginx -t  # test configurazione
sudo systemctl reload nginx
```

Ora puoi accedere al sito su http://localhost

## Parte 2: Containerizzazione con Docker

### Preparazione Ambiente Docker

1. Creare una nuova directory per il progetto Docker:
```bash
mkdir nginx-docker
cd nginx-docker
```

2. Creare il Dockerfile:
```bash
nano Dockerfile
```

Contenuto del Dockerfile:
```dockerfile
FROM ubuntu:latest

# Aggiornamento sistema e installazione nginx
RUN apt-get update && \
    apt-get install -y nginx && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Rimuovere il default.conf
RUN rm /etc/nginx/sites-enabled/default

# Configurazione nginx
COPY nginx.conf /etc/nginx/sites-available/default
RUN ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled/

# Esporre porta 80
EXPOSE 80

# Comando di avvio
CMD ["nginx", "-g", "daemon off;"]
```

3. Creare il file di configurazione nginx:
```bash
nano nginx.conf
```

```nginx
server {
    listen 80;
    server_name localhost;

    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

4. Creare docker-compose.yml:
```bash
nano docker-compose.yml
```

```yaml
version: '3'
services:
  webserver:
    build: .
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html
```

5. Creare la directory per i file HTML:
```bash
mkdir html
cp /var/www/mio-sito/index.html html/
```

### Avvio Container

1. Costruire e avviare il container:
```bash
docker-compose up --build
```

Il sito sarà accessibile su http://localhost

### Note Importanti:
- Personalizzare sempre index.html secondo le proprie necessità
- La versione Docker monta la directory `html` come volume, permettendo modifiche in tempo reale
- Per fermare il container: `docker-compose down`
- Per vedere i log: `docker-compose logs`

## Differenze Chiave tra le Due Approcci

1. **Installazione Diretta**:
   - Modifica diretta del sistema
   - Persistente dopo riavvio
   - Più difficile da replicare su altri sistemi

2. **Versione Docker**:
   - Ambiente isolato e portatile
   - Facile da distribuire e replicare
   - Gestione più semplice delle dipendenze
   - Possibilità di avere più versioni contemporaneamente

## Esercizi per gli Studenti

1. Modificare index.html aggiungendo stili CSS personalizzati
2. Aggiungere una seconda pagina HTML e collegarla alla principale
3. Modificare la configurazione nginx per gestire più siti virtuali
4. Sperimentare con diverse versioni di Ubuntu nel Dockerfile
