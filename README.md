# 4e20 — Countdown + Bacheca

Pagina countdown con bacheca commenti condivisa. Backend Node.js puro, zero dipendenze npm.

---

## Struttura

```
4e20-countdown/
├── server.js          ← backend Node.js (API + serve file statici)
├── package.json
├── .gitignore
├── data/
│   └── comments.json  ← database commenti (JSON file)
└── public/
    └── index.html     ← frontend
```

---

## Deploy su VPS / Antigravity (server Linux)

### 1. Copia i file sul server

```bash
# via scp dalla tua macchina
scp -r 4e20-countdown/ utente@tuo-server:/var/www/4e20-countdown

# oppure via git
git init
git add .
git commit -m "init"
git remote add origin git@github.com:tuoaccount/4e20-countdown.git
git push -u origin main
# poi sul server: git clone ...
```

### 2. Verifica Node.js installato

```bash
node -v   # serve >= 18
```

Se non è installato:
```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs
```

### 3. Dai i permessi al file dei commenti

```bash
cd /var/www/4e20-countdown
chmod 664 data/comments.json
```

### 4. Avvia con PM2 (processo che resta vivo)

```bash
npm install -g pm2

pm2 start server.js --name 4e20
pm2 save
pm2 startup   # segui le istruzioni per farlo partire al riavvio
```

Comandi utili PM2:
```bash
pm2 status        # stato processi
pm2 logs 4e20     # log in tempo reale
pm2 restart 4e20  # riavvia
pm2 stop 4e20     # ferma
```

### 5. Configura NGINX come reverse proxy

Crea `/etc/nginx/sites-available/4e20`:

```nginx
server {
    listen 80;
    server_name tuodominio.com www.tuodominio.com;

    location / {
        proxy_pass         http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header   Upgrade $http_upgrade;
        proxy_set_header   Connection 'upgrade';
        proxy_set_header   Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/4e20 /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### 6. SSL con Certbot (HTTPS)

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d tuodominio.com -d www.tuodominio.com
```

---

## Test in locale

```bash
cd 4e20-countdown
node server.js
# apri http://localhost:3000
```

---

## API

| Metodo | Endpoint        | Descrizione              |
|--------|-----------------|--------------------------|
| GET    | /api/comments   | Ritorna tutti i commenti |
| POST   | /api/comments   | Aggiunge un commento     |

**POST body (JSON):**
```json
{
  "name": "Mario",
  "text": "Ci saremo!"
}
```

---

## Cambiare la porta

```bash
PORT=8080 node server.js
# oppure con PM2:
pm2 start server.js --name 4e20 --env PORT=8080
```

---

## Note

- I commenti sono salvati in `data/comments.json`. Fai un backup periodico se vuoi conservarli.
- La pagina aggiorna automaticamente i commenti ogni 30 secondi.
- Zero dipendenze npm: funziona con solo Node.js installato.
