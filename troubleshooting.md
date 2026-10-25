# Hibaelhárítási esetek

## 1. Az ügyfél oldala helyett az alap oldal jön

- **Tünet:** az `ugyfel2.hosting.test` az alap "Hosting lab" oldalt mutatja.
- **Ok:** a `sites-enabled` mappából hiányzik a symlink, az Nginx nem tölti be az ügyfél beállítását, ezért a default blokk válaszol.
- **Ellenőrzés:** `dig` (a név feloldódik, tehát nem DNS-hiba), `curl` (a szerver válaszol, de rossz oldalt ad), `ls -l /etc/nginx/sites-enabled/` (a hivatkozás hiányzik).
- **Javítás:** `sudo ln -s /etc/nginx/sites-available/ugyfel2.local /etc/nginx/sites-enabled/`, majd `sudo nginx -t && sudo systemctl reload nginx`.
