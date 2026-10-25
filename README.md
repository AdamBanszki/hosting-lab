# Hosting lab

Mini hosting-környezet VMware alatt futó Kali Linuxon (Debian-alapú): DNS, webszerver, HTTPS, SFTP, és hibaelhárítási gyakorlat L1 support szemlélettel.

A környezetet útmutató segítségével, lépésről lépésre építettem fel. A cél az volt, hogy megértsem, hogyan működik egy weboldal elérése a névfeloldástól az oldalig, és hogy gyakoroljam a hibakeresést.

## Mit tartalmaz

| Terület | Eszköz | Mit gyakoroltam |
|---|---|---|
| DNS | BIND9 | saját zóna (`hosting.test`), A, NS, CNAME és SOA rekordok |
| Webszerver | Nginx | két ügyfél egy szerveren (virtual host), `www` alias |
| HTTPS | OpenSSL | önaláírt tanúsítvány, 443-as port |
| Fájlfeltöltés | OpenSSH | SFTP-hozzáférés az ügyfélnek, a saját mappájába zárva (chroot) |
| Hibaelhárítás | dig, curl, naplók | öt szándékosan előidézett hiba megkeresése és javítása |

## Egy kérés útja

    böngésző
      → a gép DNS-beállítása (127.0.0.1)
      → BIND: named.conf.local → db.hosting.test → IP-cím
      → Nginx: sites-enabled → sites-available/ugyfel1.local
      → /var/www/ugyfel1.local/html/index.html

A DNS és az Nginx két külön program, külön beállítással. A DNS csak az IP-címet adja, az Nginx a kérésben szereplő név alapján választ oldalt. Ezért egy névnek két helyen kell szerepelnie: a zónafájlban és az Nginx `server_name` sorában.

## Fájltérkép

| Fájl | Szolgáltatás | Szerepe |
|---|---|---|
| `/etc/bind/db.hosting.test` | BIND | a zóna rekordjai (név és IP-cím párosítása) |
| `/etc/bind/named.conf.local` | BIND | megadja, mely zónáért felel és hol a fájlja |
| `/etc/nginx/sites-available/` | Nginx | az ügyfelek server blockjai (név, port, mappa) |
| `/etc/nginx/sites-enabled/` | Nginx | symlinkek, amelyek bekapcsolják a beállításokat |
| `/var/www/ugyfel*.local/html/` | Nginx | az ügyfelek oldalai (document root) |
| `/etc/ssl/hosting-lab/` | Nginx | a titkos kulcs és a tanúsítvány |
| `/etc/ssh/sshd_config.d/sftp-ugyfel.conf` | SSH | az SFTP-korlátozás (csak fájlátvitel, chroot) |

Minden programnál két dolog kell: a **tartalom** (a beállítás) és a **kapcsoló** (ami miatt a program használja), módosítás után pedig **újratöltés**.

## Hibaelhárítás

Részletes leírás: [troubleshooting.md](troubleshooting.md)

Az alap hibakeresés három kérdés:

1. **Feloldódik a név?** (`dig`)
2. **Fut a szerver és válaszol?** (`curl -I`, `systemctl status`)
3. **Jó oldalt ad?** (a státuszkód és a hibanapló)

| # | Tünet | Ok | Felismerés |
|---|---|---|---|
| 1 | az alap oldal jön | hiányzó symlink a `sites-enabled` mappában | a név feloldódik, de rossz oldal jön |
| 2 | nem tölt be | rossz IP a zónafájlban | `dig` más IP-t ad, mint a szerver címe |
| 3 | 404 | rossz `root` útvonal | a naplóban `No such file or directory` |
| 4 | 403 | rossz fájljogosultság | a naplóban `Permission denied` |
| 5 | minden oldal elérhetetlen | leállt az Nginx | `inactive (dead)`, a 80-as portot senki nem figyeli |

## Tanulságok

- Az `nginx -t` a nyelvtani hibát fogja meg, a logikai hibát (létező, de rossz útvonal) nem. Javítás után `curl`-lel is tesztelni kell.
- A hibanapló az Nginx szemszögét mutatja: ha egy útvonal ott másként szerepel, mint a lemezen, a beállításban van az eltérés.
- Ha a `curl` rossz oldalt kap, a kérés elérte a szervert. Ha nem tud csatlakozni, a kérés meg sem érkezett (rossz IP, leállt szolgáltatás vagy zárt port).
- Zónafájl módosításánál a SOA serial számot növelni kell, különben a másodlagos névszerverek nem veszik észre a változást.

## Korlátok

- A `hosting.test` kitalált név, nincs delegálás, ezért a rekordokat csak az a gép látja, amelyik a lokális BIND-ot kérdezi.
- A tanúsítvány önaláírt, ezért a kliensek figyelmeztetnek. Éles környezetben a Let's Encrypt (certbot) adna megbízható tanúsítványt.
- Éles környezetben a DNS-rekordokat általában a regisztrátor vagy egy DNS-szolgáltató paneljén kezelik, a háttérben azonban ugyanezek a zónafájlok állnak.
- A labor jelszavai csak tesztelésre szolgáltak, a repó nem tartalmaz jelszót vagy kulcsot.

## Konfigurációs fájlok

A `configs/` mappában a labor beállításainak másolatai találhatók (BIND zóna, Nginx server blockok, SFTP-szabály, az oldalak). Az eredeti fájlok a `/etc` és a `/var/www` mappákban vannak, a másolat csak szemléltetés. A titkos kulcs és a tanúsítvány szándékosan nincs a repóban.
