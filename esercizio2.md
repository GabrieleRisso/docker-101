# Tutorial: Docker Multi-Container con Nginx e Client API

# Comandi Iniziali di Setup

```bash
# Creare la directory principale del progetto
mkdir docker-project
cd docker-project

# Creare la struttura per il server nginx
mkdir -p nginx-server/html
mkdir -p nginx-server/api

# Creare la directory per il client
mkdir api-client

# Creare tutti i file necessari (touch)
touch nginx-server/Dockerfile
touch nginx-server/nginx.conf
touch nginx-server/html/index.html
touch nginx-server/api/data.json
touch api-client/Dockerfile
touch api-client/script.sh
touch docker-compose.yml
```


## Struttura del Progetto

```plaintext
docker-project/
├── nginx-server/
│   ├── Dockerfile
│   ├── nginx.conf
│   ├── html/
│   │   └── index.html
│   └── api/
│       └── data.json
├── api-client/
│   └── Dockerfile
└── docker-compose.yml
```

## 1. Server Nginx con HTML e JSON

### 1.1 File JSON (nginx-server/api/data.json)
```json
{
  "messages": [
    {
      "id": 1,
      "text": "Benvenuto all'API di esempio!",
      "author": "Server Nginx"
    },
    {
      "id": 2,
      "text": "Questo è un JSON di test",
      "date": "2024-03-19"
    }
  ],
  "status": "success",
  "code": 200
}
```

### 1.2 HTML Page (nginx-server/html/index.html)
```html
<!DOCTYPE html>
<html lang="it">
<head>
    <meta charset="UTF-8">
    <title>Server Multi-Content</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 40px;
            line-height: 1.6;
        }
    </style>
</head>
<body>
    <h1>Server Multi-Content</h1>
    <p>Questa è la pagina HTML servita su /</p>
    <p>Puoi trovare il JSON su /json</p>
</body>
</html>
```

### 1.3 Nginx Configuration (nginx-server/nginx.conf)
```nginx
server {
    listen 80;
    server_name localhost;

    # Serve HTML
    location / {
        root /usr/share/nginx/html;
        index index.html;
    }

    # Serve JSON
    location /json {
        alias /usr/share/nginx/api/data.json;
        default_type application/json;
        add_header Access-Control-Allow-Origin *;
    }
}
```

### 1.4 Dockerfile per Nginx (nginx-server/Dockerfile)
```dockerfile
FROM nginx:alpine

# Rimuovere la configurazione default
RUN rm /etc/nginx/conf.d/default.conf

# Copiare la nuova configurazione
COPY nginx.conf /etc/nginx/conf.d/

# Copiare i file statici
COPY html /usr/share/nginx/html
COPY api /usr/share/nginx/api

EXPOSE 80
```

### 1.5 Dockerfile per Client (api-client/Dockerfile)
```dockerfile
FROM alpine:latest

# Installare curl e bash
RUN apk add --no-cache curl bash

# Script che farà la chiamata
COPY script.sh /script.sh
RUN chmod +x /script.sh

CMD ["/script.sh"]
```

### 1.6 Script Client (api-client/script.sh)
```bash
#!/bin/bash

# Aspetta 5 secondi per assicurarsi che il server nginx sia pronto
sleep 5

echo "Richiedo il contenuto JSON dal server..."
curl -s http://nginx-server/json | jq '.'

# Loop infinito per mantenere il container attivo
while true; do
    sleep 3600
done
```

### 1.7 Docker Compose (docker-compose.yml)
```yaml
version: '3'

services:
  nginx-server:
    build: 
      context: ./nginx-server
    ports:
      - "80:80"
    networks:
      - app-network

  api-client:
    build: 
      context: ./api-client
    depends_on:
      - nginx-server
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
```

## 2. Istruzioni per l'Avvio

1. Creare la struttura delle directory:
```bash
mkdir -p docker-project/{nginx-server/{html,api},api-client}
```

2. Copiare tutti i file nelle rispettive directory come mostrato nella struttura del progetto.

3. Avviare i container:
```bash
docker-compose up --build
```

## 3. Test e Verifica

1. L'HTML sarà accessibile su:
```
http://localhost
```

2. Il JSON sarà accessibile su:
```
http://localhost/json
```

3. Il container client farà automaticamente una richiesta al server e mostrerà il contenuto del JSON nei log.

Per vedere i log del client:
```bash
docker-compose logs api-client
```

## 4. Note Importanti

- Il container nginx-server espone due endpoint:
  - `/` per servire index.html
  - `/json` per servire data.json
- Il container api-client usa il nome del servizio `nginx-server` come hostname grazie alla rete Docker
- I container sono collegati tramite una rete bridge definita in docker-compose
- Il client attende 5 secondi prima di fare la richiesta per assicurarsi che nginx sia pronto

## 5. Troubleshooting

Se il client non riesce a raggiungere il server:
1. Verificare che entrambi i container siano nella stessa rete:
```bash
docker network ls
docker network inspect docker-project_app-network
```

2. Verificare i logs di entrambi i container:
```bash
docker-compose logs nginx-server
docker-compose logs api-client
```

## 6. Esercizi Suggeriti

1. Modificare il data.json con dati personalizzati
2. Aggiungere più endpoint JSON in nginx.conf
3. Modificare lo script del client per fare richieste periodiche
4. Aggiungere un terzo container che fa richieste sia all'HTML che al JSON
