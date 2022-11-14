# Jarkom-Modul-3-D11-2022

## Kelompok D11:

- Afira Rolobessy 5025201006

- Beryl 5025201029

- Warren Gerald Polandra 5025201233

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
