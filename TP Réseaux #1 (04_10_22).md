# TP Réseaux #1     (*04/10/22*) 

## I. Exploration locale en solo
### 1. Affichage d'informations sur la pile TCP/IP locale

#### En ligne de commande

###### *Affichez les infos des cartes réseau de votre PC :*
COMMANDE : `ipconfig /all`

**Interface WiFi :**
IP : 10.33.16.192
MAC : 4C-03-4F-E9-73-F7
NOM : Killer(R) Wi-Fi 6 AX1650i 160MHz Wireless Network Adapter (201NGW)

**Interface Ethernet :**

IP : MEDIA DECO
MAC : 4C-03-4F-E9-73-F8
NOM : Killer E2600 Gigabit Ethernet Controller

###### *Affichez votre gateway :*

COMMANDE : `arp -a`

Passerelle : 10.33.19.254
Adresse MAC : 00-c0-e7-e0-04-4e

#### En graphique (GUI : Graphical User Interface)



###### *Trouvez comment afficher les informations sur une carte IP (change selon l'OS)*

![](https://i.imgur.com/JGMki2m.png)

### 2. Modifications des informations
##### 🌞 Utilisez l'interface graphique de votre OS pour changer d'adresse IP  :
![](https://i.imgur.com/LlMYQjg.png)
*Resultat :*
![](https://i.imgur.com/h0I9TQ8.png)

##### 🌞 Il est possible que vous perdiez l'accès internet.
On peut perdre l'accès si on utilise la même adresse IP que quelqu'un.
## II. Exploration locale en duo (Test avec Maxance Ferran)
### 3. Modification d'adresse IP
##### 🌞 Modifiez l'IP des deux machines pour qu'elles soient dans le même réseau
![](https://i.imgur.com/z3FqGi9.png)

##### 🌞 Vérifier à l'aide d'une commande que votre IP a bien été changée
```
PS C:\Users\alanw> ipconfig /all

Carte Ethernet Ethernet :

   Adresse IPv4. . . . . . . . . . . . . .: 10.10.10.69(préféré)

```
##### 🌞 Vérifier que les deux machines se joignent
```
PS C:\Users\alanw> ping 10.10.10.96

Envoi d’une requête 'Ping'  10.10.10.96 avec 32 octets de données :
Réponse de 10.10.10.96 : octets=32 temps=1 ms TTL=128
Réponse de 10.10.10.96 : octets=32 temps=1 ms TTL=128
Réponse de 10.10.10.96 : octets=32 temps=1 ms TTL=128
Réponse de 10.10.10.96 : octets=32 temps=1 ms TTL=128

Statistiques Ping pour 10.10.10.96:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 1ms, Maximum = 1ms, Moyenne = 1ms
```
##### 🌞 Déterminer l'adresse MAC de votre correspondant
```
PS C:\Users\alanw> arp -a


Interface : 10.10.10.69 --- 0x5
  Adresse Internet      Adresse physique      Type
  10.10.10.96           d8-bb-c1-aa-c0-e7     dynamique   <---
  10.10.11.255          ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
  255.255.255.255       ff-ff-ff-ff-ff-ff     statique
```

### 4. Utilisation d'un des deux comme gateway
##### 🌞Tester l'accès internet

```
PS C:\Users\maxfe> ping 1.1.1.1

Envoi d’une requête 'Ping'  1.1.1.1 avec 32 octets de données :
Réponse de 1.1.1.1 : octets=32 temps=28 ms TTL=54
Réponse de 1.1.1.1 : octets=32 temps=25 ms TTL=54
Réponse de 1.1.1.1 : octets=32 temps=25 ms TTL=54
Réponse de 1.1.1.1 : octets=32 temps=28 ms TTL=54

Statistiques Ping pour 1.1.1.1:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 25ms, Maximum = 28ms, Moyenne = 26ms
```


##### 🌞 Prouver que la connexion Internet passe bien par l'autre PC

```
PS C:\Users\maxfe> tracert 192.168.137.1

Détermination de l’itinéraire vers TelosGVNG [192.168.137.1]
avec un maximum de 30 sauts :

  1     1 ms     1 ms     1 ms  TelosGVNG [192.168.137.1]

Itinéraire déterminé.
```

### 5. Petit chat privé
##### 🌞 sur le PC serveur : 
```
PS C:\Users\alanw\netcat-1.11> .\nc.exe -l -p 8888
```
##### 🌞 sur le PC client: 
```
PS C:\Users\maxfe\Desktop\netcat-win32-1.11\netcat-1.11> .\nc.exe 192.168.137.1 8888
```
#### CONVERSATION : 
```
[fmaxance] : Salut !
[balan] : Salut
[balan] : ça dit quoi ?
[fmaxance] : rien de spécial mise à part que j'aime les pommes et toi ?
[balan] : bah écoute j'aime les pommes aussi
[fmaxance] : SUPER en revoir
```
##### 🌞 Visualiser la connexion en cours

```
PS C:\Users\alanw> netstat -a -n -b

  TCP    192.168.137.1:8888     192.168.137.2:56320    ESTABLISHED
 [nc.exe]
```
##### 🌞 Pour aller un peu plus loin

IP non définie : 
```
PS C:\Users\alanw\netcat-1.11> ./nc.exe -l -p 8888

PS C:\Users\alanw> netstat -a -n -b | Select-String 8888

  TCP    0.0.0.0:8888           0.0.0.0:0              LISTENING
```
N'importe qui peut se connecter sur le serveur car l'IP est non définie.


IP définie : 
```
PS C:\Users\alanw\netcat-1.11> ./nc.exe -l -p 8888 -s 192.168.137.1
PS C:\Users\alanw> netstat -a -n -b | Select-String 8888

  TCP    192.168.137.1:8888     0.0.0.0:0              LISTENING
```
### 6. Firewall
#### 🌞 Activez et configurez votre firewall

![](https://i.imgur.com/XNnmoOO.png)


## III. Manipulations d'autres outils/protocoles côté client
### 1. DHCP
#### 🌞Exploration du DHCP, depuis votre PC
```
PS C:\Users\alanw> ipconfig /all

Carte réseau sans fil Wi-Fi :

   Suffixe DNS propre à la connexion. . . :
   Description. . . . . . . . . . . . . . : Killer(R) Wi-Fi 6 AX1650i 160MHz Wireless Network Adapter (201NGW)
   Adresse physique . . . . . . . . . . . : 4C-03-4F-E9-73-F7
   DHCP activé. . . . . . . . . . . . . . : Oui <---- 
   Configuration automatique activée. . . : Oui
   Adresse IPv6 de liaison locale. . . . .: fe80::b980:3214:57b0:7a1d%6(préféré)
   Adresse IPv4. . . . . . . . . . . . . .: 10.33.16.192(préféré)
   Masque de sous-réseau. . . . . . . . . : 255.255.252.0
   Bail obtenu. . . . . . . . . . . . . . : mercredi 5 octobre 2022 09:05:38 <----------
   Bail expirant. . . . . . . . . . . . . : jeudi 6 octobre 2022 09:05:38 <----------
   Passerelle par défaut. . . . . . . . . : 10.33.19.254
   Serveur DHCP . . . . . . . . . . . . . : 10.33.19.254 <----------
   IAID DHCPv6 . . . . . . . . . . . : 88867663
   DUID de client DHCPv6. . . . . . . . : 00-01-00-01-29-C3-B8-56-08-8F-C3-51-69-8E
```
#### 🌞 Trouver l'adresse IP du serveur DNS que connaît votre ordinateur

```
PS C:\Users\alanw> ipconfig /all

   Serveurs DNS. . .  . . . . . . . . . . : 8.8.8.8
                                       8.8.4.4
                                       1.1.1.1
```
#### 🌞 Utiliser, en ligne de commande l'outil nslookup (Windows, MacOS) ou dig (GNU/Linux, MacOS) pour faire des requêtes DNS à la main
```
PS C:\Users\alanw> nslookup google.com
Serveur :   dns.google
Address:  8.8.8.8  <---- requête à cette adresse

Réponse ne faisant pas autorité :
Nom :    google.com
Addresses:  2a00:1450:4007:819::200e
          142.250.178.142
```
```
PS C:\Users\alanw> nslookup ynov.com
Serveur :   dns.google
Address:  8.8.8.8 <---- requête à cette adresse
Réponse ne faisant pas autorité :
Nom :    ynov.com
Addresses:  2606:4700:20::681a:ae9
          2606:4700:20::681a:be9
          2606:4700:20::ac43:4ae2
          172.67.74.226
          104.26.11.233
          104.26.10.233
```
Les deux utilises le même DNS, celui de google.

```
PS C:\Users\alanw> nslookup.exe 231.34.113.12
Serveur :   dns.google
Address:  8.8.8.8

*** dns.google ne parvient pas à trouver 231.34.113.12 : Non-existent domain

PS C:\Users\alanw> nslookup.exe 78.34.2.17
Serveur :   dns.google
Address:  8.8.8.8

Nom :    cable-78-34-2-17.nc.de <---- Domaine
Address:  78.34.2.17

```
Ici seulement une des deux adresses existes sur la première on voit que le premier n'a pas de domaine, `231.34.113.12 ` n'est donc pas présent dans le DNS de google.
## IV. Wireshark
### 1. Intro Wireshark
#### 🌞 Utilisez le pour observer les trames qui circulent entre vos deux carte Ethernet. Mettez en évidence : 
![](https://i.imgur.com/9Eef9Jf.png)
![](https://i.imgur.com/NCGT1TW.png)
![](https://i.imgur.com/9v8O7xz.png)
#### 🌞 Wireshark it
Notre PC se connecte à l'adresse 77.136.192.79 et au port 443.