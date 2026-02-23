# Voraussetzungen:  
  
## VPS (z.B. IONOS):  
  
Du benötigst einen VPS, z.B. von IONOS (getestet).  
  
Minimalanforderungen:  
CPU:          4 vCore  
RAM:          4 GB  
Datenträger:  120 GB NVMe SSD  

## Betriebssystem:  

Als Basis dient entweder Proxmox direkt, falls das Image beim Hoster geladen werden kann.  
Falls nicht starten wir mit Debian Minimal:  

- Grafische Installation starten
- Sprache Deutsch auswählen und bestätigen
- WLAN einrichten, WPA-2 PSK nutzen
- Rechnername vergeben, zum Beispiel pve-home-01
- Domainname leer lassen
- Root-Passwort setzen
- Benutzerkonto anlegen mit Name und Passwort
- Geführte Partitionierung wählen und die gesamte Festplatte nutzen
- Laufwerk auswählen, alle Dateien auf eine Partition, Änderungen auf die Platte schreiben
- Paketmanager auf Deutschland setzen
- Softwareauswahl, nur SSH-Server und nur Standard-Systemwerkzeuge installieren
- Neustart durchführen und dann als root anmelden
- nano /etc/ssh/sshd_config öffnen, PermitRootLogin yes setzen und systemctl restart ssh ausführen
- nano /etc/network/interfaces öffnen und anpassen:

```
source /etc/network/interfaces.d/*

auto lo
iface lo inet loopback

auto ens6
iface ens6 inet manual
#IONOS

auto vmbr0
iface vmbr0 inet static
        address 217.154.74.243/32
        gateway 217.154.74.1
        post-up   echo 1 > /proc/sys/net/ipv4/ip_forward
        post-up iptables -t nat -A PREROUTING -i vmbr0 -p tcp -m multiport ! --dport 22,8006 -j DNAT --to 10.10.1.2
        post-up iptables -t nat -A PREROUTING -i vmbr0 -p udp -j DNAT --to 10.10.1.2
        bridge-ports ens6
        bridge-stp off
        bridge-fd 0
#IONOS

auto vmbr1
iface vmbr1 inet static
        address 10.10.1.1/30
        post-up   iptables -t nat -A POSTROUTING -s '10.10.1.0/30' -o vmbr0 -j MASQUERADE
        post-down iptables -t nat -D POSTROUTING -s '10.10.1.0/30' -o vmbr0 -j MASQUERADE
        bridge-ports none
        bridge-stp off
        bridge-fd 0
#OPNsense WAN

auto vmbr2
iface vmbr2 inet static
        address 10.11.1.100/24
        bridge-ports none
        bridge-stp off
        bridge-fd 0
#LAN 1
```

