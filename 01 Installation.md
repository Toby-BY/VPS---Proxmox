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
- nano /etc/network/interfaces öffnen und anpassen (evtl. können die Adressen auch per DHCP bezogen werden):  
  
```
auto lo
iface lo inet loopback
auto ens6
iface ens6 inet static
        address XXX.XXX.XX.XXX/32   # Die IPv4 des Servers
        gateway XXX.XXX.XX.1        # Die Gateway-Adresse des Servers
        dns-nameservers 8.8.8.8     # DNS-Server vorübergehend auf google
```
  
- Reboot durchführen  
  
  
  
## Installation von Proxmox VE 9 auf Debian 13 Trixie:  
  
- Per SSH dem Debian-System anmelden und Proxmox VE installieren.  
  Dazu fügen wir das Proxmox-VE-Repository hinzu, indem wir die Proxmox-VE-Repository-Quellen im bevorzugten deb822-Format eintragen:  
```
cat > /etc/apt/sources.list.d/pve-install-repo.sources << EOL
Types: deb
URIs: http://download.proxmox.com/debian/pve
Suites: trixie
Components: pve-no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
EOL
```
  
  
- Proxmox-VE-Repository-Schlüssel als root hinzufügen:
```
wget https://enterprise.proxmox.com/debian/proxmox-archive-keyring-trixie.gpg -O /usr/share/keyrings/proxmox-archive-keyring.gpg
```
  
- Aktualisieren der Repositories und des Systems:
```
apt update && apt full-upgrade
```
  
- Proxmox-VE-Kernel installieren und damit booten, da einige Pakete voraussetzen, dass bestimmte Kernel-Kompilier-Optionen gesetzt sind oder Kernel-Erweiterungen verfügbar sind:
```
apt install proxmox-default-kernel
```
  
- Neustarten:
```
systemctl reboot
```
  
- Installieren der Proxmox-VE-Pakete:
```
apt install proxmox-ve postfix open-iscsi chrony
```
  
- Entfernen des Standard-Kernel von Debian, um den Proxmox-Kernel zu benutzen:
```
apt remove linux-image-amd64 'linux-image-6.12*'
```
  
- Aktualisieren des GRUB2-Bootloader und anschließende Überprüfung der Konfiguration. Das ist wichtig, damit das System sauber mit dem neuen Kernel startet:
```
update-grub
```
  
- Das Paket os-prober entfernen.
  Dieses kleine Tool durchsucht beim Boot-Konfigurationsprozess alle vorhandenen Partitionen und legt automatisch Einträge für ein mögliches Dual-Boot-System an.
```
apt remove os-prober
```
  
  
```
auto lo
iface lo inet loopback

auto ens6
iface ens6 inet manual
#IONOS

auto vmbr0
iface vmbr0 inet static
        address XXX.XXX.XX.XXX/32   # Die IPv4 des Servers
        gateway XXX.XXX.XX.1        # Die Gateway-Adresse des Servers
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

