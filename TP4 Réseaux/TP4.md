# TP4 : TCP, UDP et services réseau



# Sommaire

- [TP4 : TCP, UDP et services réseau](#tp4--tcp-udp-et-services-réseau)
- [Sommaire](#sommaire)
- [I. First steps](#i-first-steps)
- [II. Mise en place](#ii-mise-en-place)
  - [1. SSH](#1-ssh)
  - [2. Routage](#2-routage)
- [III. DNS](#iii-dns)
  - [1. Présentation](#1-présentation)
  - [2. Setup](#2-setup)
  - [3. Test](#3-test)


# I. First steps

Faites-vous un petit top 5 des applications que vous utilisez sur votre PC souvent, des applications qui utilisent le réseau : un site que vous visitez souvent, un jeu en ligne, Spotify, j'sais po moi, n'importe.

🌞 **Déterminez, pour ces 5 applications, si c'est du TCP ou de l'UDP**

[StreamLabs_OBS](./STREAMLABS.pcapng) Ici l'application de StreamLabs utilise le TCP 

```
     [Streamlabs OBS.exe]
  TCP    172.20.10.2:14654      52.95.126.138:443    ESTABLISHED
```

[League_Of_Legend](./League%20Of%20Legend.pcapng) Ici le jeu utilise le UDP


***IP :** 162.249.73.149 **PORT :** 5189
**PORT SOURCE :** 52842*

```
PS C:\Users\alanw> netstat -n -b

Connexions actives

  Proto  Adresse locale         Adresse distante       État
  TCP    10.33.16.79:62306      104.199.65.124:4070    ESTABLISHED
 [Spotify.exe]
```
[Spotify](./tp4_spotify.pcapng)

```
PS C:\Users\alanw> netstat -n -b

Connexions actives

  Proto  Adresse locale         Adresse distante       État
  TCP    10.33.16.79:53231      199.232.58.249:443     ESTABLISHED
 [Discord.exe]
```
[Discord](./tp4_discord.pcapng)

[Netflix](./tp4_netflix.pcapng)





🌞 **Demandez l'avis à votre OS**

- votre OS est responsable de l'ouverture des ports, et de placer un programme en "écoute" sur un port
- il est aussi responsable de l'ouverture d'un port quand une application demande à se connecter à distance vers un serveur
- bref il voit tout quoi
- utilisez la commande adaptée à votre OS pour repérer, dans la liste de toutes les connexions réseau établies, la connexion que vous voyez dans Wireshark, pour chacune des 5 applications

**Il faudra ajouter des options adaptées aux commandes pour y voir clair. Pour rappel, vous cherchez des connexions TCP ou UDP.**

```
# MacOS
$ netstat

# GNU/Linux
$ ss

# Windows
$ netstat
```

🦈🦈🦈🦈🦈 **Bah ouais, captures Wireshark à l'appui évidemment.** Une capture pour chaque application, qui met bien en évidence le trafic en question.

# II. Mise en place

## 1. SSH

🖥️ **Machine `node1.tp4.b1`**

- n'oubliez pas de dérouler la checklist (voir [les prérequis du TP](#0-prérequis))
- donnez lui l'adresse IP `10.4.1.11/24`

Connectez-vous en SSH à votre VM.

🌞 **Examinez le trafic dans Wireshark**

[Trafic](./ANALYSE%20SSH%20NODE1.pcapng)

🌞 **Demandez aux OS**

```
  Proto  Adresse locale         Adresse distante       État
  TCP    10.4.1.10:31675        node1:ssh              ESTABLISHED
```
```
[telos@Telos ~]$ ss
  tcp   ESTAB 0      52                       10.4.1.11:ssh       10.4.1.10:31675 
```
🦈 **Je veux une capture clean avec le 3-way handshake, un peu de trafic au milieu et une fin de connexion**



- référez-vous au TP précédent

> Rien à remettre dans le compte-rendu pour cette partie.

# III. DNS

## 2. Setup

🌞 **Dans le rendu, je veux**

```
[Telos@dns1 ~]$ sudo cat /etc/named.conf
//
// named.conf
//
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching only nameserver (as a localhost DNS resolver only).
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//

options {
        listen-on port 53 { 127.0.0.1; any; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
        allow-query     { localhost; any; };
        allow-query-cache { localhost; any; };

        /*
         - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
         - If you are building a RECURSIVE (caching) DNS server, you need to enable
           recursion.
         - If your recursive DNS server has a public IP address, you MUST enable access
           control to limit queries to your legitimate users. Failing to do so will
           cause your server to become part of large scale DNS amplification
           attacks. Implementing BCP38 within your network would greatly
           reduce such attack surface
        */
        recursion yes;

        dnssec-validation yes;

        managed-keys-directory "/var/named/dynamic";
        geoip-directory "/usr/share/GeoIP";

        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";

        /* https://fedoraproject.org/wiki/Changes/CryptoPolicy */
        include "/etc/crypto-policies/back-ends/bind.config";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "tp4.b1" IN {
        type master;
        file "tp4.b1.db";
        allow-update { none; };
        allow-query { any; };
};

zone "1.4.10.in-addr.arpa" IN {
        type master;
        file "tp4.b1.rev";
        allow-update { none; };
        allow-query { any; };
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

```
[Telos@dns1 ~]$ sudo cat /var/named/tp4.b1.db
$TTL 86400
@ IN SOA dns-server.tp4.b1. admin.tp4.b1. (
    2019061800 ;Serial
    3600 ;Refresh
    1800 ;Retry
    604800 ;Expire
    86400 ;Minimum TTL
)

; Infos sur le serveur DNS lui même (NS = NameServer)
@ IN NS dns-server.tp4.b1.

; Enregistrements DNS pour faire correspondre des noms à des IPs
dns-server IN A 10.4.1.201
node1      IN A 10.4.1.11
```

```
[Telos@dns1 ~]$ sudo cat /var/named/tp4.b1.rev
$TTL 86400
@ IN SOA dns-server.tp4.b1. admin.tp4.b1. (
    2019061800 ;Serial
    3600 ;Refresh
    1800 ;Retry
    604800 ;Expire
    86400 ;Minimum TTL
)

; Infos sur le serveur DNS lui même (NS = NameServer)
@ IN NS dns-server.tp4.b1.

;Reverse lookup for Name Server
201 IN PTR dns-server.tp4.b1.
11 IN PTR node1.tp4.b1.
```

```
[Telos@dns1 ~]$ systemctl status named
● named.service - Berkeley Internet Name Domain (DNS)
     Loaded: loaded (/usr/lib/systemd/system/named.service; enabled; vendor preset: disabled)
     Active: active (running) since Mon 2022-10-31 12:15:56 CET; 9min ago
    Process: 688 ExecStartPre=/bin/bash -c if [ ! "$DISABLE_ZONE_CHECKING" == "yes" ]; then /usr/sbin/named-checkconf ->
    Process: 703 ExecStart=/usr/sbin/named -u named -c ${NAMEDCONF} $OPTIONS (code=exited, status=0/SUCCESS)
   Main PID: 708 (named)
      Tasks: 5 (limit: 11118)
     Memory: 26.0M
        CPU: 55ms
     CGroup: /system.slice/named.service
             └─708 /usr/sbin/named -u named -c /etc/named.conf

Oct 31 12:15:56 dns1.tp4.b1 named[708]: zone tp4.b1/IN: loaded serial 2019061800
Oct 31 12:15:56 dns1.tp4.b1 named[708]: zone 1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.ip6.arpa/I>
Oct 31 12:15:56 dns1.tp4.b1 named[708]: zone localhost.localdomain/IN: loaded serial 0
Oct 31 12:15:56 dns1.tp4.b1 named[708]: resolver priming query complete
Oct 31 12:15:56 dns1.tp4.b1 named[708]: managed-keys-zone: Unable to fetch DNSKEY set '.': failure
Oct 31 12:15:56 dns1.tp4.b1 named[708]: zone localhost/IN: loaded serial 0
Oct 31 12:15:56 dns1.tp4.b1 named[708]: all zones loaded
Oct 31 12:15:56 dns1.tp4.b1 named[708]: running
Oct 31 12:15:56 dns1.tp4.b1 systemd[1]: Started Berkeley Internet Name Domain (DNS).
Oct 31 12:15:56 dns1.tp4.b1 named[708]: listening on IPv4 interface enp0s8, 10.4.1.201#53
```

```
[Telos@dns1 ~]$ ss
Netid    State     Recv-Q     Send-Q                       Local Address:Port          Peer Address:Port     Process
tcp      ESTAB     0          52                              10.4.1.201:ssh              10.4.1.10:59196
```

🌞 **Ouvrez le bon port dans le firewall**

```
[Telos@dns1 ~]$ sudo firewall-cmd --add-port=59196/tcp --permanent
[sudo] password for Telos:
success
```

## 3. Test

🌞 **Sur la machine `node1.tp4.b1`**

```
[Telos@node1 ~]$ ping node1.tp4.b1
PING node1.tp4.b1(node1.tp4.b1 (fe80::a00:27ff:fe2d:28b0%enp0s8)) 56 data bytes
64 bytes from node1.tp4.b1 (fe80::a00:27ff:fe2d:28b0%enp0s8): icmp_seq=1 ttl=64 time=0.207 ms
64 bytes from node1.tp4.b1 (fe80::a00:27ff:fe2d:28b0%enp0s8): icmp_seq=2 ttl=64 time=0.063 ms
64 bytes from node1.tp4.b1 (fe80::a00:27ff:fe2d:28b0%enp0s8): icmp_seq=3 ttl=64 time=0.057 ms
```

```
[Telos@node1 ~]$ ping www.google.com
PING www.google.com (142.250.201.164) 56(84) bytes of data.
64 bytes from par21s23-in-f4.1e100.net (142.250.201.164): icmp_seq=1 ttl=113 time=14.8 ms
64 bytes from par21s23-in-f4.1e100.net (142.250.201.164): icmp_seq=2 ttl=113 time=15.8 ms
64 bytes from par21s23-in-f4.1e100.net (142.250.201.164): icmp_seq=3 ttl=113 time=14.7 ms
```