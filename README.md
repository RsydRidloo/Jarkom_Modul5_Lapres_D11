# Jarkom_Modul5_Lapres_D11
Lapres praktikum jarkom modul 5 kelompok D11

Anggota Kelompok:
- M. Farras Pangestu - 05111840000134
- Rasyid Ridlo W. - 05111840000135

**Kelompok kami pembagian subnet menggunakan metode vlsm**

- Menyetting topologi

Pertama membuat file `topo.sh` di dalam putty dengan konfigurasi

    # Switch
    uml_switch -unix switch1 > /dev/null < /dev/null &
    uml_switch -unix switch2 > /dev/null < /dev/null &
    uml_switch -unix switch3 > /dev/null < /dev/null &
    uml_switch -unix switch4 > /dev/null < /dev/null &
    uml_switch -unix switch5 > /dev/null < /dev/null &
    uml_switch -unix switch6 > /dev/null < /dev/null &

    # Router
    xterm -T SURABAYA -e linux ubd0=SURABAYA,jarkom umid=SURABAYA eth0=tuntap,,,10.151.78.49 eth1=daemon,,,switch4 eth2=daemon,,,switch5 mem=64M &
    xterm -T BATU -e linux ubd0=BATU,jarkom umid=BATU eth0=daemon,,,switch4 eth1=daemon,,,switch2 eth2=daemon,,,switch3 mem=64M &
    xterm -T KEDIRI -e linux ubd0=KEDIRI,jarkom umid=KEDIRI eth0=daemon,,,switch5 eth1=daemon,,,switch1 eth2=daemon,,,switch6 mem=64M &

    # Server dan Webserver
    xterm -T MALANG -e linux ubd0=MALANG,jarkom umid=MALANG eth0=daemon,,,switch2 mem=128M &
    xterm -T MOJOKERTO -e linux ubd0=MOJOKERTO,jarkom umid=MOJOKERTO eth0=daemon,,,switch2 mem=128M &
    xterm -T PROBOLINGGO -e linux ubd0=PROBOLINGGO,jarkom umid=PROBOLINGGO eth0=daemon,,,switch1 mem=128M &
    xterm -T MADIUN -e linux ubd0=MADIUN,jarkom umid=MADIUN eth0=daemon,,,switch1 mem=128M &

    # Klien
    xterm -T GRESIK -e linux ubd0=GRESIK,jarkom umid=GRESIK eth0=daemon,,,switch6 mem=64M &
    xterm -T SIDOARJO -e linux ubd0=SIDOARJO,jarkom umid=SIDOARJO eth0=daemon,,,switch3 mem=64M &

Pada semua UML router, ketikkan `nano /etc/sysctl.conf` serta uncomment `net.ipv4.ip_forward=1` . Untuk mengaktifkan perubahan baru ketikkan `sysctl -p`.

Lalu setting pada `nano /etc/network/interfaces` pada masing-masing uml dengan konfigurasi

**SURABAYA (Sebagai Router)**

    auto eth0
    iface eth0 inet static
    address 10.151.78.50
    netmask 255.255.255.252
    gateway 10.151.78.49

    auto eth1
    iface eth1 inet static
    address 192.168.0.1
    netmask 255.255.255.252

    auto eth2
    iface eth2 inet static
    address 192.168.0.5
    netmask 255.255.255.252

**BATU (Sebagai Router)**

    auto eth0
    iface eth0 inet static
    address 192.168.0.2
    netmask 255.255.255.252
    gateway 192.168.0.1

    auto eth1
    iface eth1 inet static
    address 10.151.79.97
    netmask 255.255.255.248

    auto eth2
    iface eth2 inet static
    address 192.168.1.1
    netmask 255.255.255.0

**KEDIRI (Sebagai Router)**

    auto eth0
    iface eth0 inet static
    address 192.168.0.6
    netmask 255.255.255.252
    gateway 192.168.0.5

    auto eth1
    iface eth1 inet static
    address 192.168.0.9
    netmask 255.255.255.248

    auto eth2
    iface eth2 inet static
    address 192.168.2.1
    netmask 255.255.255.0

**MADIUN (Sebagai Web Server)**

    auto eth0
    iface eth0 inet static
    address 192.168.0.10
    netmask 255.255.255.248
    gateway 192.168.0.9

**PROBOLINGGO (Sebagai Web Server)**

    auto eth0
    iface eth0 inet static
    address 192.168.0.11
    netmask 255.255.255.248
    gateway 192.168.0.9

**MALANG (Sebagai DNS Server)**

    auto eth0
    iface eth0 inet static
    address 10.151.79.98
    netmask 255.255.255.248
    gateway 10.151.79.97

**MOJOKERTO (Sebagai DNS Server)**

    auto eth0
    iface eth0 inet static
    address 10.151.79.99
    netmask 255.255.255.248
    gateway 10.151.79.97

Kemudian menambahkan file route.sh di Surabaya saja, karena pada uml untuk 0.0.0.0 sudah otomatis terbuat

    route add -net 192.168.1.0 netmask 255.255.255.0 gw 192.168.0.2
    route add -net 192.168.2.0 netmask 255.255.255.0 gw 192.168.0.6
    route add -net 192.168.0.8 netmask 255.255.255.248 gw 192.168.0.6
    route add -net 10.151.79.96 netmask 255.255.255.248 gw 192.168.0.2

Kemudian untuk kofigurasi DHCP Server pada MOJOKERTO dan DHCP RELAY pada KEDIRI, SURABAYA, dan BATU adalah

Pertama sebelum install harus melakukan `apt-get update` terlebih dahulu

Pada DHCP Server Jalankan `apt-get install isc-dhcp-server` pada uml MOJOKERTO

Lalu setting pada `nano /etc/dhcp/dhcpd.conf`, dengan konfigurasi

    subnet 10.151.79.0 netmask 255.255.255.0 {
    }

    #SIDOARJO
        subnet 192.168.1.0 netmask 255.255.255.0 {
        range 192.168.1.1 192.168.1.254;
        option routers 192.168.1.1;
        option broadcast-address 192.168.1.255;
        option domain-name-servers 202.46.129.2;
        default-lease-time 600;
        max-lease-time 7200;
    }

    #GRESIK
        subnet 192.168.2.0 netmask 255.255.255.0 {
        range 192.168.2.1 192.168.2.254;
        option routers 192.168.2.1;
        option broadcast-address 192.168.2.255;
        option domain-name-servers 202.46.129.2;
        default-lease-time 600;
        max-lease-time 7200;
    }

## Nomor 1: Agar topologi yang kalian buat dapat mengakses keluar, kalian diminta untuk mengkonfigurasi SURABAYA menggunakan iptables, namun Bibah tidak ingin kalian menggunakan MASQUERADE.

    iptables -t nat -A POSTROUTING -s 192.168.0.0/22 -o eth0 -j SNAT --to-source 10.151.78.50

## Nomor 2: Mendrop semua akses SSH dari luar Topologi (UML) Kalian pada server yang memiliki ip DMZ (DHCP dan DNS SERVER) pada SURABAYA demi menjaga keamanan.

    iptables -A FORWARD -d 10.151.79.96/29 -i eth0 -p tcp --dport 22 -j DROP
    
## Nomor 3: Bibah meminta kalian untuk membatasi DHCP dan DNS server hanya boleh menerima maksimal 3 koneksi ICMP secara bersamaan yang berasal dari mana saja menggunakan iptables pada masing masing server, selebihnya akan di DROP.

    iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 -j DROP
    iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j DROP

## Nomor 4: Membatasi akses ke MALANG, Akses dari subnet SIDOARJO hanya diperbolehkan pada pukul 07.00 - 17.00 pada hari Senin sampai Jumat.

    iptables -A INPUT -s 192.168.1.0/24 -m time --timestart 07:00 --timestop 17:00 --weekdays Mon,Tue,Wed,Thu,Fri -j ACCEPT
    iptables -A INPUT -s 192.168.1.0/24 -j REJECT

## Nomor 5: Membatasi akses ke MALANG, Akses dari subnet GRESIK hanya diperbolehkan pada pukul 17.00 hingga pukul 07.00 setiap harinya.

    iptables -A INPUT -s 192.168.1.0/24 -m time --timestart 07:00 --timestop 17:00 -j REJECT

## Nomor 6: SURABAYA disetting sehingga setiap request dari client yang mengakses DNS Server akan didistribusikan secara bergantian pada PROBOLINGGO port 80 dan MADIUN port 80.

    iptables -t nat -A PREROUTING -p tcp -d 10.151.79.98 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 192.168.0.11:80
    iptables -t nat -A PREROUTING -p tcp -d 10.151.79.98 -j DNAT --to-destination 192.168.0.10:80

## Nomor 7: Bibah ingin agar semua paket didrop oleh firewall (dalam topologi) tercatat dalam log pada setiap UML yang memiliki aturan drop.

    iptables -N LOGGING
    iptables -A INPUT -j LOGGING
    iptables -A OUTPUT -j LOGGING
    iptables -A LOGGING -j LOG --log-prefix "IPTables-Dropped: " --log-level 4
    iptables -A LOGGING -j DROP
