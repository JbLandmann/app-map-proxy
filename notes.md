# Migration vers https #
1. Obtenir le certificat SSL

```
# Installer Certbot
sudo apt update
sudo apt install certbot
sudo mkdir -p /var/www/certbot

# Obtenir le certificat
sudo certbot certonly --webroot \
  -w /var/www/certbot \
  -d vps-954b32b8.vps.ovh.net \
  --email votre@email.com \
  --agree-tos \
  --no-eff-email
```

2. Le proxy monte directement les certificats du VPS

```
# Les certificats restent dans /etc/letsencrypt sur le VPS
# Le proxy monte /etc/letsencrypt et /var/www/certbot en lecture seule
sudo test -f /etc/letsencrypt/live/vps-954b32b8.vps.ovh.net/fullchain.pem
sudo test -f /etc/letsencrypt/live/vps-954b32b8.vps.ovh.net/privkey.pem
```

3. Recharger le container puis tester 

une requete http doit etre redirigée vers https

4. Renouvellement automatique (minimum 90 jours)

creer un cronjob
```
sudo crontab -e
```
puis le configurer ( le mieux 1x par jour)
```
0 3 * * * certbot renew --webroot -w /var/www/certbot && docker exec app-map-proxy nginx -s reload
```
doc cronjob:
```
# tous les jours a 3h et 14h
0 3,14 * * * commande
│   │  │ │ │
│   │  │ │ └── Jour de la semaine (0-7, 0 et 7 = dimanche)
│   │  │ └──── Mois (1-12)
│   │  └────── Jour du mois (1-31)
│   └──────── Heure (0-23)
└────────── Minute (0-59)
```

5. Secret API backend

Le proxy relit `REQUEST_API_KEY` depuis `/etc/app-map/back.env` au demarrage et l'injecte vers le backend sur les requetes `/api`.
Le front doit viser `https://vps-954b32b8.vps.ovh.net/api/v1` ; le proxy retire `/api/` avant de transmettre au back qui expose `/v1/*`.


