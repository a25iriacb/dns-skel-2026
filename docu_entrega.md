**1. Instala o servidor BIND9 no equipo darthvader. Comproba que xa funciona coma servidor DNS caché pegando no documento de entrega a saída deste comando dig @localhost xunta.gal**

root@darthvader:/var/cache/bind# dig @localhost xunta.gal

; <<>> DiG 9.20.29-1~deb13u1-Debian <<>> @localhost xunta.gal
; (2 servers found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 21356
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 651f3e6e8237cf41010000006ac61dcbfee9a00cadfbb989 (good)
;; QUESTION SECTION:
;xunta.gal.                     IN      A

;; ANSWER SECTION:
xunta.gal.              28800   IN      A       85.91.64.109

;; Query time: 420 msec
;; SERVER: 127.0.0.1#53(localhost) (UDP)
;; WHEN: Wed Oct 07 10:24:11 UTC 2026
;; MSG SIZE  rcvd: 82


**2. Configura o servidor BIND9 no equipo mandalorian para que empregue como reenviador a darthvader pegando no documento de entrega contido do ficheiro /etc/bind/named.conf.options e a saída deste comando: dig @localhost santiagodecompostela.gal. Para un correcto funcionamento deberás borrar as root-hints do servidor mandalorian.**

options {
        directory "/var/cache/bind";

        forwarders {
                192.168.20.10;
        };

        forward only;
        dnssec-validation auto;
        listen-on { any; };
        listen-on-v6 { any; };
};

root@mandalorian:/var/cache/bind# dig @localhost santiagodecompostela.gal

; <<>> DiG 9.20.29-1~deb13u1-Debian <<>> @localhost santiagodecompostela.gal
; (2 servers found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 49037
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 724585726e2c0f69010000006ac6230d95fee46a08049516 (good)
;; QUESTION SECTION:
;santiagodecompostela.gal.      IN      A

;; ANSWER SECTION:
santiagodecompostela.gal. 300   IN      A       195.57.25.148

;; Query time: 3693 msec
;; SERVER: 127.0.0.1#53(localhost) (UDP)
;; WHEN: Wed Oct 07 10:46:37 UTC 2026
;; MSG SIZE  rcvd: 97

**3. Instala unha zona primaria de resolución directa chamada "starwars.lan" e engade os seguintes rexistros de recursos (a maiores dos rexistros NS e SOA imprescindibles):**

$TTL    604800
@       IN      SOA     darthsidious.starwars.lan. admin.starwars.lan. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      darthsidious.starwars.lan.
@       IN      MX  10  c3p0.starwars.lan.

darthvader     IN      A       192.168.20.10
skywalker      IN      A       192.168.20.101
skywalker      IN      A       192.168.20.111
luke           IN      A       192.168.20.22
darthsidious   IN      A       192.168.20.11
yoda           IN      A       192.168.20.24
yoda           IN      A       192.168.20.25
c3p0           IN      A       192.168.20.26

palpatine      IN      CNAME   darthsidious.starwars.lan.
lenda          IN      TXT     "Que a forza te acompanhe"

**4. Instala unha zona de resolución inversa que teña que ver co enderezo do equipo darthvader, e engade rexistros PTR para os rexistros tipo A do exercicio anterior. Pega no documento de entrega o contido do arquivo de zona, e do arquivo /etc/bind/named.conf.local**

$TTL    604800
@       IN      SOA     darthsidious.starwars.lan. admin.starwars.lan. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
; Servidor de nomes
@       IN      NS      darthsidious.starwars.lan.

; Rexistros PTR (Inversa)
10      IN      PTR     darthvader.starwars.lan.
11      IN      PTR     darthsidious.starwars.lan.
22      IN      PTR     luke.starwars.lan.
24      IN      PTR     yoda.starwars.lan.
25      IN      PTR     yoda.starwars.lan.
26      IN      PTR     c3p0.starwars.lan.
101     IN      PTR     skywalker.starwars.lan.
111     IN      PTR     skywalker.starwars.lan.


zone "starwars.lan" {
    type master;
    file "/etc/bind/zonas/db.starwars.lan";
};

zone "20.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zonas/db.192.168.20";
};

**5. Comproba que podes resolver os distintos rexistros de recursos. Pega no documento de entrega a saída dos comandos:**

Server:         localhost
Address:        127.0.0.1#53

Name:   darthvader.starwars.lan
Address: 192.168.20.10

Server:         localhost
Address:        127.0.0.1#53

Name:   skywalker.starwars.lan
Address: 192.168.20.111
Name:   skywalker.starwars.lan
Address: 192.168.20.101

Server:         localhost
Address:        127.0.0.1#53

*** Can't find starwars.lan: No answer

Server:         localhost
Address:        127.0.0.1#53

starwars.lan    mail exchanger = 10 c3p0.starwars.lan.

Server:         localhost
Address:        127.0.0.1#53

starwars.lan    nameserver = darthsidious.starwars.lan.

Server:         localhost
Address:        127.0.0.1#53

starwars.lan
        origin = darthsidious.starwars.lan
        mail addr = admin.starwars.lan
        serial = 2
        refresh = 604800
        retry = 86400
        expire = 2419200
        minimum = 604800

Server:         localhost
Address:        127.0.0.1#53

lenda.starwars.lan      text = "Que a forza te acompanhe"

11.20.168.192.in-addr.arpa      name = darthsidious.starwars.lan.