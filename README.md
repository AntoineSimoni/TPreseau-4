### TP4 - Spéléologie réseau : descente dans les couches

1. Mise en place générale
2. Création des VM

* Mise en place faites de mon coté, ainsi que rename des hosts
```
[user@client1 ~]$ ping routeur1
PING routeur1 (10.1.0.254) 56(84) bytes of data.
64 bytes from routeur1 (10.1.0.254): icmp_seq=1 ttl=64 time=0.720 ms
64 bytes from routeur1 (10.1.0.254): icmp_seq=2 ttl=64 time=0.769 ms
64 bytes from routeur1 (10.1.0.254): icmp_seq=3 ttl=64 time=0.858 ms
64 bytes from routeur1 (10.1.0.254): icmp_seq=4 ttl=64 time=0.734 ms
--- routeur1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3009ms
rtt min/avg/max/mdev = 0.720/0.770/0.858/0.057 ms
```
 serveur1 ping routeur1
 ```
[user@serveur1 ~]$ ping routeur1
PING routeur1 (10.2.0.254) 56(84) bytes of data.
64 bytes from routeur1 (10.2.0.254): icmp_seq=1 ttl=64 time=1.19 ms
64 bytes from routeur1 (10.2.0.254): icmp_seq=2 ttl=64 time=0.854 ms
64 bytes from routeur1 (10.2.0.254): icmp_seq=3 ttl=64 time=0.458 ms
64 bytes from routeur1 (10.2.0.254): icmp_seq=4 ttl=64 time=0.375 ms
^C
--- routeur1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 0.375/0.720/1.193/0.327 ms
```
3. Mise en place du routage statique
Routeur1
```
[user@routeur1 ~]$ ip route
10.1.0.0/24 dev enp0s8 proto kernel scope link src 10.1.0.254 metric 100
10.2.0.0/24 dev enp0s9 proto kernel scope link src 10.2.0.254 metric 101
```
```
client1
[user@client1 ~]$ ip route show
10.1.0.0/24 dev enp0s8 proto kernel scope link src 10.1.0.10 metric 100
10.2.0.0/24 via 10.2.0.254 dev enp0s8 proto static metric 100
10.2.0.254 dev enp0s8 proto static scope link metric 100
```
serveur1
```
[user@serveur1 ~]$ ip route show
10.1.0.0/24 via 10.1.0.254 dev enp0s8 proto static metric 100
10.1.0.254 dev enp0s8 proto static scope link metric 100
10.2.0.0/24 dev enp0s8 proto kernel scope link src 10.2.0.10 metric 100
```
client1 ping serveur1
```
    [user@client1 ~]$ ping serveur1
    PING serveur1 (10.2.0.10) 56(84) bytes of data.
    64 bytes from serveur1 (10.2.0.10): icmp_seq=1 ttl=63 time=0.778 ms
    64 bytes from servur1 (10.2.0.10): icmp_seq=2 ttl=63 time=1.41 ms
    64 bytes from serveur1 (10.2.0.10): icmp_seq=3 ttl=63 time=1.39 ms
    ^C
    --- serveur1 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2002ms
    rtt min/avg/max/mdev = 0.778/1.195/1.411/0.298 ms 
```
serveur1 ping client1
```
    [user@serveur1 ~]$ ping client1
    PING client1 (10.1.0.10) 56(84) bytes of data.
    64 bytes from client1 (10.1.0.10): icmp_seq=1 ttl=63 time=0.794 ms
    64 bytes from client1 (10.1.0.10): icmp_seq=2 ttl=63 time=1.43 ms
    64 bytes from client1 (10.1.0.10): icmp_seq=3 ttl=63 time=1.47 ms
    64 bytes from client1 (10.1.0.10): icmp_seq=4 ttl=63 time=1.31 ms
    ^C
    --- client1 ping statistics ---
    4 packets transmitted, 4 received, 0% packet loss, time 3005ms
    rtt min/avg/max/mdev = 0.794/1.252/1.471/0.271 ms
```
traceroute depuis client1 :
```
    [user@client1 ~]$ traceroute serveur1
    traceroute to serveur1 (10.2.0.10), 30 hops max, 60 byte packets
    1  routeur1 (10.1.0.254)  0.316 ms  0.183 ms  0.170 ms
    2  serveur1 (10.2.0.10)  0.434 ms !X  0.500 ms !X  0.522 ms !X
```
II Spéléologie réseau
1. ARP
A. Manip 1
client1 :
```
    [user@client1 ~]$ ip neigh show
    10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:1a DELAY
```
serveur1 : 
```
    [user@serveur1 ~]$ ip neigh show
    10.2.0.1 dev enp0s8 lladdr 0a:00:27:00:00:1b REACHABLE
```u
client1 ping serveur1 
``` [user@client1 ~]$ ip neigh show 10.1.0.254 dev enp0s8 lladdr 08:00:27:26:4b:c5 STALE 10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:1a REACHABLE 10.2.0.254 dev enp0s8 lladdr 08:00:27:26:4b:c5 REACHABLE ```

```
    [user@serveur1 ~]$ ip neigh show
    10.1.0.254 dev enp0s8 lladdr 08:00:27:1c:5c:be STALE
    10.2.0.1 dev enp0s8 lladdr 0a:00:27:00:00:1b DELAY
    10.2.0.254 dev enp0s8 lladdr 08:00:27:1c:5c:be STALE
```
Check
routeur1 :
```
    [user@routeur1 ~]$ ip neigh show
    10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:1a REACHABLE
```
C. Manip 3
On affiche la table ARP :
``` C:\Users\system32>arp -a
    Interface : 192.168.56.1 --- 0x7
    Adresse Internet      Adresse physique      Type
    192.168.56.255        ff-ff-ff-ff-ff-ff     statique
    224.0.0.22            01-00-5e-00-00-16     statique
    224.0.0.251           01-00-5e-00-00-fb     statique
    224.0.0.252           01-00-5e-00-00-fc     statique
    239.255.255.250       01-00-5e-7f-ff-fa     statique
    255.255.255.255       ff-ff-ff-ff-ff-ff     statique

    Interface : 10.33.2.178 --- 0x9
    Adresse Internet      Adresse physique      Type
    10.33.3.37            90-cd-b6-64-bc-e7     dynamique
    10.33.3.253           00-12-00-40-4c-bf     dynamique
    10.33.3.254           94-0c-6d-84-50-c8     dynamique
    10.33.3.255           ff-ff-ff-ff-ff-ff     statique
    224.0.0.22            01-00-5e-00-00-16     statique
    224.0.0.251           01-00-5e-00-00-fb     statique
    224.0.0.252           01-00-5e-00-00-fc     statique
    239.255.255.250       01-00-5e-7f-ff-fa     statique
    255.255.255.255       ff-ff-ff-ff-ff-ff     statique

    Interface : 192.168.102.1 --- 0x16
    Adresse Internet      Adresse physique      Type
    192.168.102.255       ff-ff-ff-ff-ff-ff     statique
    224.0.0.22            01-00-5e-00-00-16     statique
    224.0.0.251           01-00-5e-00-00-fb     statique
    224.0.0.252           01-00-5e-00-00-fc     statique
    239.255.255.250       01-00-5e-7f-ff-fa     statique

    Interface : 10.1.0.1 --- 0x1a
    Adresse Internet      Adresse physique      Type
    10.1.0.10             08-00-27-03-d0-93     dynamique
    10.1.0.254            08-00-27-26-4b-c5     dynamique
    10.1.0.255            ff-ff-ff-ff-ff-ff     statique
    224.0.0.22            01-00-5e-00-00-16     statique
    224.0.0.251           01-00-5e-00-00-fb     statique
    224.0.0.252           01-00-5e-00-00-fc     statique
    239.255.255.250       01-00-5e-7f-ff-fa     statique

    Interface : 10.2.0.1 --- 0x1b
    Adresse Internet      Adresse physique      Type
    10.2.0.10             08-00-27-89-17-80     dynamique
    10.2.0.255            ff-ff-ff-ff-ff-ff     statique
    224.0.0.22            01-00-5e-00-00-16     statique
    224.0.0.251           01-00-5e-00-00-fb     statique
    224.0.0.252           01-00-5e-00-00-fc     statique
    239.255.255.250       01-00-5e-7f-ff-fa     statique
```
On la suppprime : 
``` C:\WINDOWS\system32>arp -a
    Interface : 192.168.56.1 --- 0x7
    Adresse Internet      Adresse physique      Type
    224.0.0.22            01-00-5e-00-00-16     statique
    255.255.255.255       ff-ff-ff-ff-ff-ff     statique

    Interface : 10.33.2.178 --- 0x9
    Adresse Internet      Adresse physique      Type
    10.33.3.253           00-12-00-40-4c-bf     dynamique
    224.0.0.22            01-00-5e-00-00-16     statique
    224.0.0.252           01-00-5e-00-00-fc     statique

    Interface : 192.168.102.1 --- 0x16
    Adresse Internet      Adresse physique      Type
    224.0.0.22            01-00-5e-00-00-16     statique

    Interface : 10.1.0.1 --- 0x1a
    Adresse Internet      Adresse physique      Type
    224.0.0.22            01-00-5e-00-00-16     statique

    Interface : 10.2.0.1 --- 0x1b
    Adresse Internet      Adresse physique      Type
    224.0.0.22            01-00-5e-00-00-16     statique
```
On la re affiche
``` C:\WINDOWS\system32>arp -a
    Interface : 192.168.56.1 --- 0x7
    Adresse Internet      Adresse physique      Type
    192.168.56.255        ff-ff-ff-ff-ff-ff     statique
    224.0.0.22            01-00-5e-00-00-16     statique
    224.0.0.252           01-00-5e-00-00-fc     statique
    239.255.255.250       01-00-5e-7f-ff-fa     statique
    255.255.255.255       ff-ff-ff-ff-ff-ff     statique

    Interface : 10.33.2.178 --- 0x9
    Adresse Internet      Adresse physique      Type
    10.33.3.253           00-12-00-40-4c-bf     dynamique
    10.33.3.255           ff-ff-ff-ff-ff-ff     statique
    224.0.0.22            01-00-5e-00-00-16     statique
    224.0.0.252           01-00-5e-00-00-fc     statique
    239.255.255.250       01-00-5e-7f-ff-fa     statique

    Interface : 192.168.102.1 --- 0x16
    Adresse Internet      Adresse physique      Type
    192.168.102.255       ff-ff-ff-ff-ff-ff     statique
    224.0.0.22            01-00-5e-00-00-16     statique
    224.0.0.252           01-00-5e-00-00-fc     statique
    239.255.255.250       01-00-5e-7f-ff-fa     statique

    Interface : 10.1.0.1 --- 0x1a
    Adresse Internet      Adresse physique      Type
    10.1.0.255            ff-ff-ff-ff-ff-ff     statique
    224.0.0.22            01-00-5e-00-00-16     statique
    224.0.0.252           01-00-5e-00-00-fc     statique
    239.255.255.250       01-00-5e-7f-ff-fa     statique

    Interface : 10.2.0.1 --- 0x1b
    Adresse Internet      Adresse physique      Type
    10.2.0.255            ff-ff-ff-ff-ff-ff     statique
    224.0.0.22            01-00-5e-00-00-16     statique
    224.0.0.252           01-00-5e-00-00-fc     statique
    239.255.255.250       01-00-5e-7f-ff-fa     statique
```