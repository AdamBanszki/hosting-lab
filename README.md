# Hosting lab

Mini hosting-környezet felépítése VMware alatt futó Kali Linuxon (Debian-alapú).
Cél: a webhosting support alapjainak gyakorlása (Nginx, DNS, HTTPS, SFTP, hibaelhárítás).

## 1. lépés: Nginx telepítése és ellenőrzése

    sudo apt update
    sudo apt install nginx -y
    systemctl status nginx
    sudo ss -tlnp | grep :80
    curl -I http://localhost

- `ss` kimenetében: az Nginx figyel a 80-as porton (LISTEN).
- `curl` válasza: `HTTP/1.1 200 OK`, vagyis a webszerver működik.

## 2. lépés: saját weboldal

A document root (`/var/www/html`) ellenőrzése, majd saját `index.html` létrehozása.

    grep root /etc/nginx/sites-enabled/default
    ls -l /var/www/html

- Ha az ügyfél oldala helyett az alap oldal látszik: rossz mappa vagy hiányzó `index.html`.

## 3. lépés: két weboldal egy szerveren (virtual host)

Két ügyfél, két mappa, két Nginx server block. A beállítás a `sites-available`
mappában van, a `sites-enabled` mappába symlinkkel kapcsolva be.

    sudo nginx -t
    sudo systemctl reload nginx
    curl -H "Host: ugyfel1.local" http://localhost

- Ha egy nevet nem ismer fel a szerver, a default oldalt adja vissza.
- Újratöltés előtt mindig `nginx -t`, éles szerveren `reload`, nem `restart`.

## 4. lépés: saját DNS-zóna (BIND9)

Zóna: `hosting.test` (A, NS, CNAME rekordok). Zónafájl: `/etc/bind/db.hosting.test`.

    sudo named-checkconf
    sudo named-checkzone hosting.test /etc/bind/db.hosting.test
    dig @127.0.0.1 ugyfel1.hosting.test +short

- Módosításnál a SOA serial számot mindig növelni kell.
- A rekordok neveinek végén a pont számít (teljes név).
- Az `aa` jelzés a dig kimenetében: hiteles (authoritative) válasz.

## 5. lépés: DNS és Nginx összekötése

Az Nginx `server_name` értékeit a DNS-zóna nevéhez igazítottuk (`*.hosting.test`),
a gép resolverét pedig a NetworkManagerben a saját BIND-ra állítottuk
(tartalék: a VMware DNS).

    sudo nmcli con mod "Wired connection 1" ipv4.dns "127.0.0.1 192.168.80.2" ipv4.ignore-auto-dns yes
    dig ugyfel1.hosting.test +short
    curl http://ugyfel1.hosting.test

- A DNS egyedül nem elég: a `www` névnek az Nginx `server_name` sorában is szerepelnie kell,
  különben a default blokk válaszol.
- Visszaállítás: `sudo nmcli con mod "Wired connection 1" ipv4.dns "" ipv4.ignore-auto-dns no`
