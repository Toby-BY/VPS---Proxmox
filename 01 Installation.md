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

Grafische Installation starten.
Sprache Deutsch auswählen und bestätigen.
WLAN einrichten, WPA-2 PSK nutzen.
Rechnername vergeben, zum Beispiel pve-home-01.
Domainname leer lassen.
Root-Passwort setzen.
Benutzerkonto anlegen mit Name und Passwort.
Geführte Partitionierung wählen und die gesamte Festplatte nutzen.
Laufwerk auswählen, alle Dateien auf eine Partition, Änderungen auf die Platte schreiben.
Paketmanager auf Deutschland setzen
Softwareauswahl, nur SSH-Server und nur Standard-Systemwerkzeuge installieren.
Neustart durchführen und dann als root anmelden.
nano /etc/ssh/sshd_config öffnen, PermitRootLogin yes setzen und systemctl restart ssh ausführen.
