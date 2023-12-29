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

Docker-Volume für Portainer erstellen
```
docker volume create portainer_data
```

Zu Kontrolle: Liste alle aktiven Docker-Container anzeigen
```
docker ps
```

Portainer UI im Browser mit ```https://ip-adresse-vom-Rasberry-Pi:9443``` starten. Benutzer & Passwort anlegen!



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
___

## 4.2 - Portainer Container anlegen und konfigurieren 

UI im Browser mit ```https://ip-adresse-vom-Rasberry-Pi:9443``` starten.

### **4.2.1 - Neuen Container erstellen - Add Container:**

![image](https://github.com/obenschlaefer/beepi/assets/79227566/132c130f-d9e1-4a49-b9ee-ba88e748e02d)
___
### **4.2.2 - Container-Einstellungen:**

**Name:** 	Mosquitto

**Image:**  	eclipse-mosquitto:latest

**Ports:**  
1883 - 1883 TCP


9001 - 9001 TCP


![image](https://github.com/obenschlaefer/beepi/assets/79227566/f935a7c3-72c1-4b65-868e-00facec4965b)
___
### **4.2.3 - Volumes:**

**Container**: /mosquitto --> **Host:** /home/pi/mosquitto -- >**Bind**

**Container:** /mosquitto/data --> **Host:** /home/pi/mosquitto/data --> **Bind**

**Container:** /mosquitto/log --> **Host:** /home/pi/mosquitto/log -->**Bind**


![image](https://github.com/obenschlaefer/beepi/assets/79227566/d8d98f8b-131c-46d9-90c8-a1e5bbbe706b)
___


### **4.2.4 - Restart policy:** 
Always

![image](https://github.com/obenschlaefer/beepi/assets/79227566/f41ed412-f7aa-4ee2-b518-5abc51a24e0a)
___


### **4.2.5 - Deploy Container**

![image](https://github.com/obenschlaefer/beepi/assets/79227566/bb414b08-c9d6-4f39-ae37-ecdcbeb5acc1)
___

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

```mosquitto_passwd -c /mosquitto/config/password.txt admin```

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


# 5. Zigbee2MQTT in Docker installieren (USB-Gateway-Konfig)

Wenn du ein Netzwerkgateway verwendest, kannst du zum Punkt 6 springen.

- USB-Gateway (Dongle) einstecken.

- per SSH mit dem Pi verbinden. Dann folgende Befehle ausführen:

```lsusb```

Ausgabe (Beispiel):

```Bus 001 Device 003: ID 1a86:55d4 QinHeng Electronics SONOFF Zigbee 3.0 USB Dongle Plus V2```

Wichtig: Vendor-ID und Product-ID notieren. Hier in meinem Beispiel die ```1a86``` und die ```55d4```

- Danach die ```99-usb-serial.rules``` datei erstellen:

```sudo nano /etc/udev/rules.d/99-usb-serial.rules```

- Folgendes muss in die ```99-usb-serial.rules``` hereinkopiert werden

```SUBSYSTEM=="tty", ATTRS{idVendor}=="1a86", ATTRS{idProduct}=="55d4", SYMLINK+="zigbee-stick"```

Vendor-ID und Product-ID an den hervorgehobenen Stellen eingeben (zwischen den “”) --> Hier muss natürlich jweils eure ID eingetragen werden

- Datei speichern und Nanao beenden

```Strg - x``` und mit ```y``` bestätigen

- Jetzt ein Neustart:
```sudo reboot```

- Nacg dem  Neustart Symlink testen:
```ls -l /dev/zigbee-stick```

Ausgabe (Beispiel)
```lrwxrwxrwx 1 root root 7 Mar 30 11:28 /dev/zigbee-stick -> ttyACM0```

Hinweis: Der USB-Port, hier ```ttyACM0```, muss später in der ```configuration.yaml``` eingegeben werden.

- Im nächetn Schritt die ```configuration.yaml``` erstellen. Dazu folgende Befehle ausführen (Verzeichnisse anlegen):
```
cd
mkdir -p zigbee2mqtt/data
cd zigbee2mqtt
```

wget https://raw.githubusercontent.com/Koenkk/zigbee2mqtt/master/data/configuration.yaml -P data

nano data/configuration.yaml





