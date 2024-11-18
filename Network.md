# Padroneggiare le Reti Docker

## Introduzione
Alcuni mesi fa ho dimostrato attraverso un esempio pratico le ragioni per comprendere e sfruttare i volumi Docker. Oggi, faremo lo stesso con le reti Docker.

Se vuoi padroneggiare le reti Docker, questa lezione fa per te.

## I Container Vivono in una Rete
> 💡 **Concetto Chiave**  
> A causa della natura isolata dei container, essi non condividono la rete dell'host. Tuttavia, Docker fornisce delle reti dedicate per loro.

Quando il Runtime Docker si avvia, crea 3 reti predefinite:

```bash
$ docker network ls

NETWORK ID     NAME      DRIVER    SCOPE
5c65c2b3031b   bridge    bridge    local
cf446ef29441   host      host      local
50fd86384bb9   none      null      local
```

Analizziamo ciascuna di queste reti.

## La Rete Bridge
La rete bridge è la rete predefinita per i container Docker. Vediamo come funziona.

```bash
# Controlliamo la configurazione ispezionando la rete
$ docker network inspect bridge
```

📦 **Output di Esempio:**
```json
[
    {
        "Name": "bridge",
        "Id": "5c65c2b3031b6d10f357f74f6cb5bf04af13819fca28b5458e00bb6b1d1718ec",
        "Created": "2022-06-27T23:49:43.227773167Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        }
    }
]
```

> ℹ️ **Nota Importante**  
> Possiamo vedere che la rete risponde attraverso il Gateway 172.17.0.1 e al momento non ha container collegati.

### Esempio Pratico con NGINX

Creiamo un container NGINX per vedere come funziona la rete bridge:

```bash
# Creiamo un container NGINX
$ docker run --name nginx --network bridge -d nginx
```

🔍 **Cosa succede?**
- Viene creato un container NGINX
- Il container viene eseguito sulla porta 80
- Serve la pagina HTML "Welcome to NGINX"
- Viene automaticamente aggiunto alla rete bridge

### Verifichiamo la Connessione di Rete

```bash
$ docker network inspect bridge
```

📋 **Risultato:**
```json
"Containers": {
    "bb283ee626dbc631281fc0c27a1f02f075ab1908800965008a315cedd7f9d438": {
        "Name": "nginx",
        "IPv4Address": "172.17.0.2/16"
    }
}
```

## Comunicazione tra Container

> ⚠️ **Attenzione**  
> I container non condividono la rete dell'host. Questo significa che solo altri container nella stessa rete (bridge) possono comunicare tra loro.

### Test di Comunicazione

```bash
# Proviamo a comunicare da un nuovo container al nostro NGINX
$ docker run \
    --network bridge \
    alpine \
    sh -c "wget http://172.17.0.2 -O -"
```

✅ **Funziona!** Vedremo la pagina di benvenuto di NGINX.

## Reti Personalizzate
Docker permette di creare reti personalizzate con il driver bridge:

```bash
# Creiamo una nuova rete chiamata 'saturno'
$ docker network create saturno
```

🌟 **Vantaggi delle Reti Personalizzate:**
- Risoluzione dei nomi automatica
- Migliore isolamento
- Possibilità di connettere container on-the-fly

### Esempio con Risoluzione dei Nomi

```bash
# Creiamo un container nella rete personalizzata
$ docker run --name nginx --network saturno -d nginx

# Ora possiamo usare il nome del container invece dell'IP
$ docker run --network saturno alpine sh -c "wget http://nginx -O -"
```

## La Rete Host
> 🖥️ **Nota sulla Rete Host**  
> Questa rete è utile quando dobbiamo esporre il container alla rete dell'host.
> **Importante:** Funziona solo su Linux!

```bash
$ docker run --name nginx --network host -d nginx
```

## L'Opzione -p
Un modo alternativo per esporre i container all'host:

```bash
# Espone la porta 80 del container sulla porta 80 dell'host
$ docker run --name nginx -p 80:80 -d nginx
```

## La Rete None
Per container completamente isolati:

```bash
$ docker run --name nginx --network none -d nginx
```

⛔ **Caratteristiche della Rete None:**
- Nessun Gateway IP 
- Nessun indirizzo IP assegnato ai container
- Isolamento completo

## Bonus: Connettere Container al Volo

```bash
# Creiamo due container
$ docker run --name nginx -d nginx
$ docker run --name ruby -dit ruby irb

# Creiamo una nuova rete
$ docker network create saturno

# Colleghiamo i container esistenti alla nuova rete
$ docker network connect saturno nginx
$ docker network connect saturno ruby
```

## Conclusione
Questa lezione copre gli aspetti principali delle reti Docker. Per approfondimenti, consultare la documentazione ufficiale Docker.

> 📚 **Per Approfondire**  
> - Studia i diversi tipi di driver di rete
> - Esplora le opzioni di networking avanzate
> - Prova a creare architetture multi-container