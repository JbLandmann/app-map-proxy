# Migration vers https #
1. Obtenir le certificat SSL

```
# Installer Certbot
sudo apt update
sudo apt install certbot

# Obtenir le certificat
sudo certbot certonly --webroot \
  -w /chemin/vers/proxy/certbot/www \
  -d dns_du_serveur \
  --email votre@email.com \
  --agree-tos \
  --no-eff-email
```

2. Copier les certificats

```
# Copier les certificats vers le dossier Nginx
sudo cp -r /etc/letsencrypt/* /chemin/vers/proxy/certbot/conf/
sudo chown -R $(whoami):$(whoami) /chemin/vers/proxy/certbot/conf
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
0 3 * * * certbot renew --webroot -w /chemin/vers/proxy/certbot/www && docker exec app-map-proxy nginx -s reload
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


