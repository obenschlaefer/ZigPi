# beepi

### Welcome to the Beepi-Project! - Willkommen beim Beepi-Projekt!
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


## 1. Hardware vorbereiten - Bootloader updaten für das Booten von USB Laufwerken (falls erfordelich)

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

## 2. Docker installieren

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

## 3. Portainer installieren (Docker Conatiner)

Docker-Volume für Portainer erstellen
```
docker volume create portainer_data
```

Zu Kontrolle: Liste alle aktiven Docker-Container anzeigen
```
docker ps
```

Portainer UI im Browser mit ```https://ip-adresse-vom-Rasberry-Pi:9443``` starten. Benutzer & Passwort anlegen!


## 4. Mosquitto installieren (Docker Container)

Verzeichnisse und config-file anlegen:

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

Portainer UI im Browser mit ```https://ip-adresse-vom-Rasberry-Pi:9443``` starten.

**Neuen Container erstellen - Add Container:**

![image](https://github.com/obenschlaefer/beepi/assets/79227566/132c130f-d9e1-4a49-b9ee-ba88e748e02d)

**Container-Einstellungen:**

Name für Container vergeben: _ _mosquitto_ _

Image:  _ _eclipse-mosquitto:latest_ _
Ports:  _ _1883 - 1883 TCP_ _
	      _ _9001 - 9001 TCP_ _

![image](https://github.com/obenschlaefer/beepi/assets/79227566/f935a7c3-72c1-4b65-868e-00facec4965b)

Volumes:
Container: _ _/mosquitto_ _		    Host: _ _/home/pi/mosquitto_ _		    Bind
Container: _ _/mosquitto/data_ _	Host: _ _/home/pi/mosquitto/data_ _	  Bind
Container: _ _/mosquitto/log_ _	  Host: _ _/home/pi/mosquitto/log_ _  	Bind

![image](https://github.com/obenschlaefer/beepi/assets/79227566/d8d98f8b-131c-46d9-90c8-a1e5bbbe706b)

Restart policy:
_ _Always_ _

![image](https://github.com/obenschlaefer/beepi/assets/79227566/f41ed412-f7aa-4ee2-b518-5abc51a24e0a)

**Deploy Container**

![image](https://github.com/obenschlaefer/beepi/assets/79227566/bb414b08-c9d6-4f39-ae37-ecdcbeb5acc1)











