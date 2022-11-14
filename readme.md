# Jarkom-Modul-3-D11-2022

## Kelompok D11:

- Afira Rolobessy 5025201006

- Beryl 5025201029

- Warren Gerald Polandra 5025201233

## Pembagian Tugas:

- Afira: DHCP, Topologi (33.33%)
- Beryl: DHCP (33.33%)
- Warren: Proxy (33.33%)

## Soal 1
Loid bersama Franky berencana membuat peta tersebut dengan kriteria WISE sebagai DNS Server, Westalis sebagai DHCP Server, Berlint sebagai Proxy Server.

Topologi yang dibuat:

![Topologi](https://cdn.discordapp.com/attachments/856609726225973278/1041561246320828437/image.png)

List IP setiap node:

```
Ostania:	192.190.1.1	| DCHP Relay / Router

SSS:		DHCP		| Client / Client Proxy
Garden:		DHCP		| Client / Client Proxy

WISE:		192.190.2.2	| DNS Server
Berlint:	192.190.2.3	| Proxy Server
Westalis:	192.190.2.4	| DHCP Server

KemonoPark:	DHCP		| Client
NewstonCastle:	DHCP		| Client
Eden:		192.190.3.13	| Client / Fixed Address
```

Network configurstion pada Ostania:

```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
	address 192.190.1.1
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 192.190.2.1
	netmask 255.255.255.0

auto eth3
iface eth3 inet static
	address 192.190.3.1
	netmask 255.255.255.0
```

Network Configuration pada WISE:

```
auto eth0
iface eth0 inet static
	address 192.190.2.2
	netmask 255.255.255.0
	gateway 192.190.2.1
```

Network Configuration pada Berlint:

```
auto eth0
iface eth0 inet static
	address 192.190.2.3
	netmask 255.255.255.0
	gateway 192.190.2.1
```

Network Configuration pada Westalis:

```
auto eth0
iface eth0 inet static
	address 192.190.2.4
	netmask 255.255.255.0
	gateway 192.190.2.1
```

Setelah itu lakukan `iptables` pada Ostania

`iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.190.0.0/16`

Instalasi update dan service pada WISE sebagai DNS Server:

```
echo nameserver 192.168.122.1 > /etc/resolv.conf

apt-get update
apt-get install bind9 -y
service bind9 start
```

Instalasi update dan service pada Berlint sebagai Proxy Server:

```
echo nameserver 192.168.122.1 > /etc/resolv.conf

apt-get update
apt-get install apt-utils
apt-get install squid
service squid restart
```

Instalasi update dan service pada Westalis sebagai DHCP Server:

```
echo nameserver 192.168.122.1 > /etc/resolv.conf

apt-get update -y
apt-get install isc-dhcp-server -y
dhcpd --version

echo 'INTERFACES="eth0"' > /etc/default/isc-dhcp-server
```

### Ping Google pada WISE:
![Ping Google](https://cdn.discordapp.com/attachments/856609726225973278/1041565254150271006/image.png)

## Soal 2
dan Ostania sebagai DHCP Relay. Loid dan Franky menyusun peta tersebut dengan hati-hati dan teliti.

Instalasi update dan service pada Ostania:

```
apt-get update
apt-get install isc-dhcp-relay -y
dhcrelay --version

echo 'SERVERS="192.190.2.4"
INTERFACES="eth1 eth2 eth3"
OPTIONS=""' > /etc/default/isc-dhcp-relay
```

pada file `/etc/default/isc-dhcp-relay`, `SERVERS` diisi dengan IP dari Westalis sebagai DHCP server. `INTERFACES` diisi dengan `eth1 eth2 eth3` untuk menunjukkan jalur mana saja yang akan diberikan service DHCP

## Soal 3 dan 6
Ada beberapa kriteria yang ingin dibuat oleh Loid dan Franky, yaitu:

1. Semua client yang ada HARUS menggunakan konfigurasi IP dari DHCP Server.
2. Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.50 - [prefix IP].1.88 dan [prefix IP].1.120 - [prefix IP].1.155

Network Configuration dari SSS, Garden, Eden, NewstonCastle, dan KemonoPark:

```
auto eth0
iface eth0 inet dhcp
```

Pada Westalis sebagai DHCP server, konfigurasi pada file `/etc/dhcp/dhcpd.conf` untuk client yang ada pada Switch1:

```
subnet 192.190.1.0 netmask 255.255.255.0 {
    range 192.190.1.50 192.190.1.88;   # range 1
    range 192.190.1.120 192.190.1.155; # range 2
    option routers 192.190.1.1;
    option broadcast-address 192.190.1.255;
    option domain-name-servers 192.190.2.2; # IP WISE
    default-lease-time 300; # Waktu selama 5 menit
    max-lease-time 6900; # Waktu selama 115 menit
}
```

## Soal 4 dan 6
3. Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.10 - [prefix IP].3.30 dan [prefix IP].3.60 - [prefix IP].3.85

Pada Westalis sebagai DHCP server, konfigurasi pada file `/etc/dhcp/dhcpd.conf` untuk client yang ada pada Switch3:

```
subnet 192.190.3.0 netmask 255.255.255.0 {
    range 192.190.3.10 192.190.3.30; # range 1
    range 192.190.3.60 192.190.3.85; # range 2
    option routers 192.190.3.1;
    option broadcast-address 192.190.3.255;
    option domain-name-servers 192.190.2.2; # IP WISE
    default-lease-time 600; # Waktu selama 10 menit
    max-lease-time 6900; # Waktu selama 115 menit
}
```

## Soal 5
Client mendapatkan DNS dari WISE dan client dapat terhubung dengan internet melalui DNS tersebut.

Pada Westalis sebagai DHCP server, konfigurasi pada file `/etc/dhcp/dhcpd.conf` untuk Switch2:

```
subnet 192.190.2.0 netmask 255.255.255.0 {
    option routers 192.190.2.1;
}
```

Pada WISE sebagai DNS Server, konfigurasi pada file `/etc/bind/named.conf.options` untuk menambahkan DNS Forwarder:

```
options {
      directory "/var/cache/bind";

       forwarders {
              192.168.122.1; 
       };

      allow-query{any;};
      auth-nxdomain no;    # conform to RFC1035
      listen-on-v6 { any; };
};
```

Setelah itu restart service berikut:

- bind9: `service bind9 restart` pada WISE sebagai DNS Server
- DHCP Server: `service isc-dhcp-server restart` pada Westalis sebagai DHCP Server
- DHCP Relay: `service isc-dhcp-relay restart` pada Ostania sebagai DHCP Relay

### Hasil DHCP pada client:

Pada SSS:

![SSS](https://cdn.discordapp.com/attachments/856609726225973278/1041570846281564181/image.png)

Pada Garden:

![Garden](https://cdn.discordapp.com/attachments/856609726225973278/1041571127912308786/image.png)

Pada NewstonCastle:

![NewstonCastle](https://cdn.discordapp.com/attachments/856609726225973278/1041571517957406720/image.png)

Pada KemonoPark:

![KemonoPark](https://cdn.discordapp.com/attachments/856609726225973278/1041571725676138556/image.png)

## Soal 8
Pada Proxy Server di Berlint, Loid berencana untuk mengatur bagaimana Client dapat mengakses internet. Artinya setiap client harus menggunakan Berlint sebagai HTTP & HTTPS proxy. Adapun kriteria pengaturannya adalah sebagai berikut:

1. Client hanya dapat mengakses internet diluar (selain) hari & jam kerja (senin-jumat 08.00 - 17.00) dan hari libur (dapat mengakses 24 jam penuh)
2. Adapun pada hari dan jam kerja sesuai nomor (1), client hanya dapat mengakses domain loid-work.com dan franky-work.com (IP tujuan domain dibebaskan)
3. Saat akses internet dibuka, client dilarang untuk mengakses web tanpa HTTPS. (Contoh web HTTP: http://example.com)
4. Agar menghemat penggunaan, akses internet dibatasi dengan kecepatan maksimum 128 Kbps pada setiap host (Kbps = kilobit per second; lakukan pengecekan pada tiap host, ketika 2 host akses internet pada saat bersamaan, keduanya mendapatkan speed maksimal yaitu 128 Kbps)
5. Setelah diterapkan, ternyata peraturan nomor (4) mengganggu produktifitas saat hari kerja, dengan demikian pembatasan kecepatan hanya diberlakukan untuk pengaksesan internet pada hari libur

### Peraturan nomor 1

Konfigurasi Berlint sebagai Proxy Server pada file `/etc/squid/acl.conf`:

```
acl AVAILABLE_WORKING time MTWHF 08:00-17:00
```

#### Testing nomor 1

`lynx google.com` pada jam kerja:

![lynx jam kerja](https://cdn.discordapp.com/attachments/856609726225973278/1041672407024414841/image.png)

`lynx google.com` pada jam libur:

![lynx jam libur](https://cdn.discordapp.com/attachments/856609726225973278/1041672817931993118/image.png)

### Peraturan nomor 2

Konfigurasi WISE sebagai DNS Server pada file `/etc/bind/named.conf.local`:

```
zone "loid-work.com" {
        type master;
        file \"/etc/bind/jarkom/loid-work.com\";
};

zone "franky-work.com" {
        type master;
        file \"/etc/bind/jarkom/franky-work.com\";
};
```

Membuat direktori baru pada WISE sebagai DNS Server:

`mkdir /etc/bind/jarkom/`

Membuat konfigurasi untuk `loid-work.com` pada file `/etc/bind/jarkom/loid-work.com` dengan IP tujuan Eden:

```
;
; BIND data file for local loopback interface
;
\$TTL   604800
@       IN      SOA     loid-work.com. root.loid-work.com. (
                     0911202201         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@               IN      NS      loid-work.com.
@               IN      A       192.190.3.13
```

Membuat konfigurasi untuk `franky-work.com` pada file `/etc/bind/jarkom/franky-work.com` dengan IP tujuan Eden:

```
;
; BIND data file for local loopback interface
;
\$TTL   604800
@       IN      SOA     franky-work.com. root.franky-work.com. (
                     0911202202         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@               IN      NS      franky-work.com.
@               IN      A       192.190.3.13
```

Restart bind9 dengan `service bind9 restart`

Konfigurasi Berlint sebagai Proxy Server pada file `/etc/squid/sites.whitelist.working_hour.txt`:


```
.loid-work.com
.franky-work.com
```

Instalasi apache dan lynx pada Eden:

```
apt-get update
apt-get install lynx -y

apt-get install apache2 -y
service apache2 start
```

#### Testing nomor 2

`lynx loid-work.com`:

![loid-work.com](https://cdn.discordapp.com/attachments/856609726225973278/1041673126494355456/image.png)

`lynx franky-work.com`:

![franky-work.com](https://cdn.discordapp.com/attachments/856609726225973278/1041673512928153620/image.png)

### Peraturan nomor 3

Konfigurasi Berlint sebagai Proxy Server pada file `/etc/squid/https_regex.conf`:

```
acl HTTPS url_regex ^https://(\.*)$
```

#### Testing nomor 3

http pada hari kerja jam libur:

![http](https://cdn.discordapp.com/attachments/856609726225973278/1041673788565225512/image.png)

https pada hari kerja jam libur:

![https](https://cdn.discordapp.com/attachments/856609726225973278/1041674038692556800/image.png)

### Peraturan nomor 4

Konfigurasi Berlint sebagai Proxy Server pada file `/etc/squid/acl-bandwidth.conf`:

```
delay_pools 1
delay_class 1 2
delay_access 1 allow all
delay_parameters 4 16000/16000
```

### Peraturan nomor 5

```
acl day_off time AS 00:00-24:00
delay_pools 1
delay_class 1 2
delay_access 1 allow day_off
delay_parameters 4 16000/16000
```

#### Testing nomor 4 dan 5

Pada hari libur:

![Limit Bandwitdth hari libur](https://cdn.discordapp.com/attachments/856609726225973278/1041670643495735296/image.png)

Pada jam kerja:

![Limit Bandwidth jam kerja](https://cdn.discordapp.com/attachments/856609726225973278/1041670812933046282/image.png)

Pada hari kerja di bawah jam 08:00 dan di atas jam 17:00:

![Limit Bandwidth hari kerja jam libur](https://cdn.discordapp.com/attachments/856609726225973278/1041671895491297340/image.png)

### Konfigurasi pada file `/etc/squid/squid.conf`:

```
include /etc/squid/acl.conf
include /etc/squid/acl-bandwidth.conf

acl WORKING_HOUR_WHITELIST dstdomain "/etc/squid/sites.whitelist.working_hour.txt"

http_port 8080
visible_hostname Berlint

http_access allow WORKING_HOUR_WHITELIST AVAILABLE_WORKING
http_access deny all
```

## Kendala

- Kesulitan debugging akibat kesulitan mencari kesalahan karena tidak ada indikasi tempat error
- Beberapa materi tidak ada di modul dan harus dicari di internet
- Terlalu banyak trial dan error (terutama pada bagian bandwidth)
