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
