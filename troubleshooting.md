# Hibaelhárítási esetek

## 1. Az ügyfél oldala helyett az alap oldal jön

- **Tünet:** az `ugyfel2.hosting.test` az alap "Hosting lab" oldalt mutatja.
- **Ok:** a `sites-enabled` mappából hiányzik a symlink, az Nginx nem tölti be az ügyfél beállítását, ezért a default blokk válaszol.
- **Ellenőrzés:** `dig` (a név feloldódik, tehát nem DNS-hiba), `curl` (a szerver válaszol, de rossz oldalt ad), `ls -l /etc/nginx/sites-enabled/` (a hivatkozás hiányzik).
- **Javítás:** `sudo ln -s /etc/nginx/sites-available/ugyfel2.local /etc/nginx/sites-enabled/`, majd `sudo nginx -t && sudo systemctl reload nginx`.

## 2. Az oldal nem tölt be ("Could not connect")

- **Tünet:** az `ugyfel2.hosting.test` nem tölt be, a `curl` "Could not connect to server" hibát ad a 80-as porton.
- **Ok:** a zónafájlban az `ugyfel2` A rekordja rossz IP-címre mutatott, ahol nem fut szerver.
- **Ellenőrzés:** `dig` (a név feloldódik, de az IP nem egyezik a működő `ugyfel1` és a szerver `eth0` címével). A kérés nem érte el az Nginxet, ezért az Nginx fájljai nem érintettek.
- **Javítás:** a rekord javítása a `/etc/bind/db.hosting.test` fájlban, `named-checkzone`, majd `sudo rndc reload`.
- **Megjegyzés:** több névszerver esetén a SOA serial számot is növelni kell.

## 3. 404 Not Found

- **Tünet:** az `ugyfel2.hosting.test` 404-et ad.
- **Ok:** az Nginx beállításában a `root` sor elírt útvonalra mutatott (`htm` helyett `html`), ezért a kért fájl nem található.
- **Ellenőrzés:** `dig` (a név jó), `curl -I` (404, vagyis a kérés elérte az Nginxet), `sudo tail /var/log/nginx/error.log` (az Nginx által keresett útvonal), `grep root` és `ls` (a beállítás és a valódi mappa összevetése).
- **Javítás:** a `root` útvonal javítása, majd `sudo nginx -t && sudo systemctl reload nginx`.
- **Megjegyzés:** az `nginx -t` nem fogja meg a nem létező mappát, mert az logikai hiba. Javítás után `curl`-lel is tesztelni kell.
