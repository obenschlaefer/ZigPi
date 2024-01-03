# beepi

**Welcome to the Beepi-Project! - Willkommen beim Beepi-Projekt!**

![IMG_2459](https://github.com/obenschlaefer/beepi/assets/79227566/d1ced44a-cbcc-4d95-84e9-95ad60ac32a1)

#####################
English version below!
#####################

Hier fidest du eine Anleitung, um Zigbee2mqtt auf einem Raspberry Pi zu installieren.
1. Hardware vorbereiten - Bootloader updaten für das Booten von USB Laufwerken (falls erfordelich)
2. Docker installieren
3. Portainer installieren (Docker Container)
4. Mosquitto installieren (Docker Container)
5. Zigbee2mqtt installieren (Docker Container)
6. Testumgebung mit Openhab erstellen (Docker Container)
7. Heimdall installieren (Docker Container)

##############
English version:
##############

Here you will find instructions on how to install Zigbee2mqtt on a Raspberry Pi.
1. Prepare hardware - Update Bootloader fur booting from USB-Drives (if necessary)
2. Install Docker
3. Install Portainer (Docker Container)
4. Install Mosquitto (Docker Container)
5. Install Zigbee2mqtt (Docker Container)
6. Create test environment with Openhab (Docker Container)
7. Install Heimdall (Docker Container


# 1. - Hardware vorbereiten - Bootloader updaten für das Booten von USB Laufwerken (falls erfordelich)

Um den Versionstand des Bootloaders herauszufinden muss man zuerst ganz normal von Micro-SD Karte booten.
Dann per SSH verbinden und folgende Befehle ausführen

```
vcgencmd bootloader_version
sudo rpi-eeprom-update
```

Das angezeigte Datum muss  der 3. September 2020, oder neuer sein. 

Wenn das Datum aktueller als der 3. September 2020 ist, oder eine Meldung kommt, dass der Raspberry Pi "up to date" ist, brucht kein Update durchgeführt werden. Der Raspberry Pi ist so aktuell, dass er schon von Werrk aus von USB botten kann.
Wenn ein Update verfügbar ist, dann folgende Befehle ausführen

```
sudo rpi-eeprom-update -a
sudo reboot
```

Dadruch findet en Update des Bootloaders statt. Danach wird der Raspberry Pi neugetartet und bootet ab jetzt zuerst von USB. (Bootreihenfolge wurde gändert)

Micro-SD raus und (spätestens jetzt) das USB/SSD Laufwerk verbinden. Raspberry Pi einschalten und zur Erfolgskontrolle per SSH verbinden.

Danach ein Update kann nicht schaden

```
sudo apt-get update 
sudo apt-get full-upgrade
```

# 2. - Docker installieren


Verbindung per SSH 

Docker herunterladen:
```
curl -fsSL https://get.Docker.com -o get-Docker.sh
```

Script zur Installation ausführen
```
sudo sh get-Docker.sh
```

Docker für aktuellen Benutzer ausführbar machen
```
sudo usermod -aG docker $USER
```

Benutzerrichtlinien neun laden
```
newgrp docker 
```

Docker testen
```
docker run hello-world
```


# 3. - Portainer installieren (Docker Conatiner)

- Docker-Volume für Portainer erstellen
```
docker volume create portainer_data
```

- Portainer pullen, konfigurieren & starten
```
docker run -d -p 8000:8000 -p 9443:9443 --restart always --name portainer -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data cr.portainer.io/portainer/portainer-ce:latest
```

- Zu Kontrolle: Liste alle aktiven Docker-Container anzeigen
```
docker ps
```
<img width="1465" alt="image" src="https://github.com/obenschlaefer/beepi/assets/79227566/43dcd701-4700-4777-b5e2-308b965085f8">

Portainer UI im Browser mit ```https://ip-adresse-vom-Rasberry-Pi:9443``` starten. 
Beim ersten Start: Benutzer & Passwort anlegen!


# 4. - Mosquitto installieren (Docker Container)

## 4.1 - Verzeichnisse und config-file anlegen:

```
cd
sudo mkdir -p mosquitto/config
sudo nano mosquitto/config/mosquitto.conf
```
Nano Editor öffnet sich mit der zuvor erstellten ```mosquitto.conf```

Folgenden Inhalt in die ```mosquitto.conf``` hereinkopieren: 
```
persistence true
persistence_location /mosquitto/data/
log_dest file /mosquitto/log/mosquitto.log
listener 1883

## Authentication ##
allow_anonymous true
```
Wichtig: ``` allow_anonymous true ``` wird im späteren Verlauf noch geändert!

## 4.2 - Docker Container anlegen und konfigurieren (Portainer) 

UI im Browser mit ```https://ip-adresse-vom-Rasberry-Pi:9443``` starten.

**Neuen Container erstellen - Add Container:**

![image](https://github.com/obenschlaefer/beepi/assets/79227566/132c130f-d9e1-4a49-b9ee-ba88e748e02d)

Name: 	```Mosquitto```

Image: 	```eclipse-mosquitto:latest```

Ports: 
```1883 - 1883 TCP```


```9001 - 9001 TCP```


![image](https://github.com/obenschlaefer/beepi/assets/79227566/f935a7c3-72c1-4b65-868e-00facec4965b)

**Volumes:**

Container: ```/mosquitto``` 

Host: ```/home/pi/mosquitto``` 

```Bind```

Container: ```/mosquitto/data``` 

Host: ```/home/pi/mosquitto/data``` 

```Bind```

Container: ```/mosquitto/log``` 

Host: ```/home/pi/mosquitto/log``` 

```Bind```


![image](https://github.com/obenschlaefer/beepi/assets/79227566/d8d98f8b-131c-46d9-90c8-a1e5bbbe706b)



**Restart policy:** 
Always

![image](https://github.com/obenschlaefer/beepi/assets/79227566/f41ed412-f7aa-4ee2-b518-5abc51a24e0a)



**Deploy Container**

![image](https://github.com/obenschlaefer/beepi/assets/79227566/bb414b08-c9d6-4f39-ae37-ecdcbeb5acc1)


## **4.3 - Mosquitto testen mit MQTT-Explorer**

- Hier den MQTT-Explorer downloaden: https://mqtt-explorer.com/
- MQTT-Explorer starten

Eingabe von **Host** (IP-Adresse des Raspberry PI) und **Port** (wie hier in der Anleitung: 1883)

Zunächst kein Username und Passwort eingeben (wir haben ja noch kein ```User/Password``` in der ```mosquitto.conf``` angegeben)

<img width="756" alt="image" src="https://github.com/obenschlaefer/beepi/assets/79227566/dee0ca4b-f3a5-4e7c-bc8f-cce5980272f0">

Wenn sich der MQTT-Explorer erfolgreich mit dem Broker verbindet, kann das Programm wieder geschlossen werden. Das Ganze war nur ein erster Verbindungstest. Im nächsten Schritt legen wir **User** und **Password** fest.	


## **4.4 - Passwortschutz (authentication) einrichten**

- Im Mosquitto Container auf die Console gehen

<img width="817" alt="image" src="https://github.com/obenschlaefer/beepi/assets/79227566/5cd7803a-75ef-4b5f-bdfa-f3c894ca22d1">

- im nächten Schritt ```/bin/sh``` auswählen und auf ```Connect``` klicken.

<img width="348" alt="image" src="https://github.com/obenschlaefer/beepi/assets/79227566/da7578bb-c46d-401f-b2f5-f138b9f17a0c">

- Die ```Console``` des Containers öffnet sich. Folgende Zeilen eigeben:

```
mosquitto_passwd -c /mosquitto/config/password.txt admin
```

- Dieser Befehl legt den Benutzer (User) ```admin``` an. Wenn ein anderer Name gewünscht ist, den Befehl entsprechend ändern!

- Mit ```Enter``` bestätigen und dann das gewünschte ```Password``` engeben. Die Eingabe erfolgt verdeckt - d.h. sie wird nicht angezeigt.

<img width="457" alt="image" src="https://github.com/obenschlaefer/beepi/assets/79227566/0d0c9091-2dee-44e9-8b7e-3f40059d6f43">

- im nächsten Schritt auf ```Disconnect``` klicken, um die Container-Console zu schließen.
- Anschließend per ```SSH``` mit dem Raspberry Pi verbinden, und in der Console folgenden Befehle eingeben:

```
cd /mosquitto/config
sudo nano mosquitto.conf
```

Der Editor Nano öffnet die schon erstellte```mosquitto.conf```. Diese wie folgt ändern:

```
persistence true
persistence_location /mosquitto/data/
log_dest file /mosquitto/log/mosquitto.log
listener 1883

## Authentication ##
allow_anonymous false
password_file /mosquitto/config/password.txt
```
Die ```mosquitto.conf``` wird hier um die ```Authentication``` erweitert, und so schlussendlich sicher gemacht.

- Nano beenden: ```strg+x``` und dann den Speichern-Dialog mit ```y``` bestätigen. So wird die vorhandene ```mosquitto.conf``` mit den Änderunghen übwerschreiben.

Jetzt noch einmal die Verbindung mit dem MQTT-Explorer testen - diesmal mit dem angelegten ```User``` und dem ```Password```

<img width="688" alt="image" src="https://github.com/obenschlaefer/beepi/assets/79227566/f25bd488-fe8d-4f8e-b8b9-e4cfd48d4b39">


- Erfolgreiche Verbindung (Beispielansicht -sieht bei euch anders aus!)
  
<img width="1055" alt="image" src="https://github.com/obenschlaefer/beepi/assets/79227566/2829285d-6871-485f-b958-84d8dae10626">


# 5. Zigbee2MQTT in Docker installieren und konfigurieren (USB-Gateway)

## 5.1 USB-Gateway einrichten

Wenn du ein Netzwerk-Gateway (LAN-Gateway) verwendest, kannst du zu Punkt 6 springen.

- USB-Gateway (Dongle) einstecken.

- per SSH mit dem Pi verbinden. Dann folgende Befehle ausführen:

```
lsusb
```

Ausgabe (Beispiel):

```Bus 001 Device 003: ID 1a86:55d4 QinHeng Electronics SONOFF Zigbee 3.0 USB Dongle Plus V2```

Wichtig: Vendor-ID und Product-ID notieren. Hier in meinem Beispiel die ```1a86``` und die ```55d4```

<img width="597" alt="image" src="https://github.com/obenschlaefer/beepi/assets/79227566/40a240d3-12aa-4318-93c6-b8f8b673d5f7">

- Danach die ```99-usb-serial.rules``` datei erstellen:

```
sudo nano /etc/udev/rules.d/99-usb-serial.rules
```

- Folgendes muss in die ```99-usb-serial.rules``` hereinkopiert werden

```
SUBSYSTEM=="tty", ATTRS{idVendor}=="1a86", ATTRS{idProduct}=="55d4", SYMLINK+="zigbee-stick"
```

Vendor-ID und Product-ID an den hervorgehobenen Stellen eingeben (zwischen den “”) --> Hier muss natürlich jeweils eure ID eingetragen werden

<img width="597" alt="image" src="https://github.com/obenschlaefer/beepi/assets/79227566/d8205fa2-3833-4494-9741-a0eae780cf87">

- Nano beenden: ```strg+x``` und dann den Speichern-Dialog mit ```y``` bestätigen.

- Jetzt ein Neustart:
```
sudo reboot
```

- Abschließend: Nach dem Neustart Symlink testen:
- 
```
ls -l /dev/zigbee-stick
```

Ausgabe (Beispiel):
```lrwxrwxrwx 1 root root 7 Mar 30 11:28 /dev/zigbee-stick -> ttyACM0```

Hinweis: Der USB-Port, hier ```ttyACM0```, muss später in der ```configuration.yaml``` eingegeben werden.

<img width="448" alt="image" src="https://github.com/obenschlaefer/beepi/assets/79227566/00ce294d-a13f-4e95-a8a9-0f076a2e2396">

## 5.2 Konfiguration

- Im näschten Schritt die ```configuration.yaml``` erstellen. Dazu folgende Befehle ausführen (Verzeichnisse anlegen):
```
cd
mkdir -p zigbee2mqtt/data
cd zigbee2mqtt
```

- Standard ```configuration.yaml``` herunterladen

```
wget https://raw.githubusercontent.com/Koenkk/zigbee2mqtt/master/data/configuration.yaml -P data
```

- und für unser Projekt anpassen:

```
nano data/configuration.yaml
```

Es öffnent sich der Editor ```Nano``` mit der zuvor heruntergeladenen ```configuration.yaml```. Diese bitte wiefolgt anpassen:

```
# Inhalt configuration.yaml (USB-Gateway)
# Home Assistant integration (MQTT discovery)
homeassistant: false

# allow new devices to join
permit_join: true

# MQTT settings
mqtt:
  # MQTT base topic for zigbee2mqtt MQTT messages
  base_topic: zigbee2mqtt
  # MQTT server URL
  server: mqtt://172.17.31.113:1883
  # MQTT server authentication, uncomment if required:
  user: 
  password: 

# Serial settings
serial:
  # Location of CC2531 USB sniffer
  port: /dev/ttyACM0  
advanced:
  network_key: GENERATE
frontend:
  port: 8080
```
- Nano beenden: ```strg+x``` und dann den Speichern-Dialog mit ```y``` bestätigen.
  
**!! Wichtig:** 
- Bei den ```MQTT settings``` wird der MQTT ```user``` und das ```password``` eintragen. Außerdem wird hier bei ```server``` die IP-Adresse + Port des MQTT-Brokers eingtragen. Diesen haben wir ja vorab installiert (Mosquitto) --> Das bedeutet: Hier kommt die IP-Adresse eures Pi rein incl. dem Port, der vorher im Docker Container freigegeben wurde.

- Bei den ```Serial settings``` wird unter ```port``` den entsprechenden Pfad zum USB-Gateway eingetragen. 

Beispiel für die MQTT-Server URL:
```server: mqtt://172.17.31.113:1883```

Beispiel für den port:
```port: /dev/ttyACM0```

## 5.3 Konfiguraton des Zigbee2MQTT Docker-Containers mit Portainer (USB-Gateway)

- Portainer öffnen: 
UI im Browser mit ```https://ip-adresse-vom-Rasberry-Pi:9443``` starten.

- Neuen Container anlegen und entsprechend der folgenden Vorgaben Konfigurieren:

  <img width="197" alt="image" src="https://github.com/obenschlaefer/beepi/assets/79227566/7a0b2f77-dc44-43bf-944c-e7cfe651aa94">

**Allgeneine Konfiguration:**
  
Name: ```zigbee2mqtt```

Image: ```koenkk/zigbee2mqtt:latest```

Ports: ```8080 - 8080 TCP```

<img width="1433" alt="image" src="https://github.com/obenschlaefer/beepi/assets/79227566/237d2c00-a968-4e25-b20f-5a2efdeca3fa">

**Command & Logging:**

Driver: ```json-file```

- Klicke 2x auf ```add logging driver option```

option: ```max-file```
value: ```5```
option: ```max-size```
value: ```10m```

<img width="1600" alt="image" src="https://github.com/obenschlaefer/beepi/assets/79227566/91616488-ea10-42af-9191-cea9731d6bd8">

**Volumes:**

Container: ```/app/data```		

Host:  ```/home/pi/zigbee2mqtt/data```	```Bind```

Container: ```/run/udev```		

Host: ```/run/udev```			```Bind``` (Read-only) 

<img width="1629" alt="image" src="https://github.com/obenschlaefer/beepi/assets/79227566/9857f007-e89a-463b-9b3a-0af0f9773bd0">

**Restart policy:**

```always```

<img width="333" alt="image" src="https://github.com/obenschlaefer/beepi/assets/79227566/f912c55d-3f39-4b7d-9dcd-399125ee8cdc">

**Runtime & Ressources:**

- Klick 1x auf ```add device```

folgende Eingaben sind zu machen:

Host: ```/dev/zigbee-stick```	
container: ```/dev/ttyACM0``` (muss dem Eintrag in der "configuration.yaml"entsprechen)

<img width="1654" alt="image" src="https://github.com/obenschlaefer/beepi/assets/79227566/bd83a513-948a-4882-bdc5-cb53def9ea5a">
 
**Deploy container & kurz warten - fertig!**

<img width="378" alt="image" src="https://github.com/obenschlaefer/beepi/assets/79227566/04beca84-1933-44a5-a7ed-461090d37601">

Zum Schluss: Zigbee2mqtt Ui im Browswer aufrufen: 
```http://[RaspberryPi-IP]:8080```

<img width="1895" alt="image" src="https://github.com/obenschlaefer/beepi/assets/79227566/4190a7b9-ad39-4f43-a137-a33d3988397b">
Das Bild zeigt ein Beispiel mit bereits gepairten Devices. Die Erstansicht sieht bei euch anders aus (ohne Devices)!

# 6. Zigbee2MQTT in Docker installieren und konfigurieren (LAN-Gateway)

Unterschiede zum USB-Gateway:
1. Bei der Verwendung eines Netzwerkgateways entfällt logischerweise die gesamte USB-Konfiguration (Punkt 5.1 u. 5.2).
2. Außederm ist der Inhalt der```configuraton.yaml``` anders.
3. Die konfiguration des Docker Containers ist anders.
   
## 6.1 Konfiguraton LAN-Gateway (Beispiel)

Wenn ein Netzwerkgateway verwendet wird, muss dieses vorab über das Web-UI konfiguriert werden:

- In der Web-UI des Gateways die IP-Adresse des MQTT-Brokers eingetragen werden (Bild 1)
- Außerdem wird der ```Socket Port``` für die Eingabe in der ```configration.yaml``` benötigt (Bild 2)

Hier ein Beispiel für das LAN-Gateway, das ich verwende (Zigbee ESP32 GW)

<img width="1919" alt="image" src="https://github.com/obenschlaefer/beepi/assets/79227566/0621b641-3511-428d-8f40-aaff85fad8cf">
<img width="978" alt="image" src="https://github.com/obenschlaefer/beepi/assets/79227566/aad7518e-30d7-413f-b4ed-8f0ece508ed9">


## 6.2 Konfiguraton (SSH-console)


- per SSH mit dem Pi verbinden. Dann folgende Befehle ausführen:

- Im näschten Schritt die ```configuration.yaml``` erstellen. Dazu folgende Befehle ausführen (Verzeichnisse anlegen):
```
cd
mkdir -p zigbee2mqtt/data
cd zigbee2mqtt
```

- Standard ```configuration.yaml``` herunterladen

```
wget https://raw.githubusercontent.com/Koenkk/zigbee2mqtt/master/data/configuration.yaml -P data
```

- und für unser Projekt anpassen:

```
nano data/configuration.yaml
```

Es öffnent sich der Editor ```Nano``` mit der zuvor heruntergeladenen ```configuration.yaml```. Diese bitte wiefolgt anpassen:

```
 # Home Assistant integration (MQTT discovery)
homeassistant: false

# allow new devices to join
permit_join: true

# MQTT settings
mqtt:
  # MQTT base topic for zigbee2mqtt MQTT messages
  base_topic: zigbee2mqtt
  # MQTT server URL
  server: mqtt://172.17.31.113:1883
  # MQTT server authentication, uncomment if required:
  user: 
  password: 

# Serial settings
serial:
  # Location of CC2531 USB sniffer
  port: tcp://172.17.31.68:6638
advanced:
  network_key: GENERATE
frontend:
  port: 8080
```
- Nano beenden: ```strg+x``` und dann den Speichern-Dialog mit ```y``` bestätigen.

**!! Wichtig:** 
- Bei den ```MQTT settings``` wird der MQTT ```user``` und das ```password``` eintragen. Außerdem wird hier bei ```server``` die IP-Adresse + Port des MQTT-Brokers eingtragen. Diesen haben wir ja vorab installiert (Mosquitto) --> Das bedeutet: Hier kommt die IP-Adresse eures Pi rein incl. dem Port, der vorher im Docker Container freigegeben wurde.

- Bei den ```Serial settings``` wird unter ```port``` die IP-Adresse vom LAN-Gateway und der Socket Port eingetragen. 

Beispiel für die MQTT-Server URL:
```server: mqtt://172.17.31.113:1883```

Beispiel für den port:
port: tcp://172.17.31.68:6638

## 6.3 Konfiguraton des Zigbee2MQTT Docker-Containers mit Portainer (LAN-Gateway)

- Portainer öffnen: 
UI im Browser mit ```https://ip-adresse-vom-Rasberry-Pi:9443``` starten.

- Neuen Container anlegen und entsprechend der folgenden Vorgaben Konfigurieren:

  <img width="197" alt="image" src="https://github.com/obenschlaefer/beepi/assets/79227566/7a0b2f77-dc44-43bf-944c-e7cfe651aa94">

**Allgeneine Konfiguration:**
  
Name: ```zigbee2mqtt```

Image: ```koenkk/zigbee2mqtt:latest```

Ports: ```8080 - 8080 TCP```

<img width="1433" alt="image" src="https://github.com/obenschlaefer/beepi/assets/79227566/237d2c00-a968-4e25-b20f-5a2efdeca3fa">

**Command & Logging:**

Driver: ```json-file```

- Klicke 2x auf ```add logging driver option```

option: ```max-file```
value: ```5```
option: ```max-size```
value: ```10m```

<img width="1600" alt="image" src="https://github.com/obenschlaefer/beepi/assets/79227566/91616488-ea10-42af-9191-cea9731d6bd8">

**Volumes:**

Container: ```/app/data```		

Host:  ```/home/pi/zigbee2mqtt/data```	

```Bind```

<img width="1670" alt="image" src="https://github.com/obenschlaefer/beepi/assets/79227566/0b8a043b-f1b8-43af-8bfd-b82c311ae246">

**Restart policy:**

```always```

<img width="333" alt="image" src="https://github.com/obenschlaefer/beepi/assets/79227566/f912c55d-3f39-4b7d-9dcd-399125ee8cdc">

**Deploy container & kurz warten - fertig!**

<img width="378" alt="image" src="https://github.com/obenschlaefer/beepi/assets/79227566/04beca84-1933-44a5-a7ed-461090d37601">

Zum Schluss: Zigbee2mqtt Ui im Browswer aufrufen: 
```http://[RaspberryPi-IP]:8080```

<img width="1895" alt="image" src="https://github.com/obenschlaefer/beepi/assets/79227566/4190a7b9-ad39-4f43-a137-a33d3988397b">
Das Bild zeigt ein Beispiel mit bereits gepairten Devices. Die Erstansicht sieht bei euch anders aus (ohne Devices)!
