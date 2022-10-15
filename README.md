# Commands

## Container
- `run <Image Name>:<Version>` <br />
Container mit Image erstellen und wenn das Image noch nicht auf dem System ist, von Docker-Hub runterladen
    - `--name` Setzt den Names des Containers
    - `-d` Setzt detached mode -> im Hintergrund laufen lassen
    - `-i` Interactive Mode aktiviert stdin
    - `-t` Aktiviert ein Terminal -> am besten immer `-it` wenn man mit dem Container interagieren möchte
    - `-p <Host-Port>:<Container-Port>` Port mapping bsp. `-p 80:5000` -> der port 80 vom Host wird auf Port 5000 vom Docker Container gemapped.
    - `-v <Host-Dir>:<Container-Dir>` Volume Mapping bsp. `-v /opt/datadir:/var/lib/mysql` -> die Daten an dem Pfad im Docker Container werden and dem Pfad von Host System gespeichert, damit sie nicht verloren gehen
    - `-e <Variablen-Name>=<Wert>` Environment Variablen
    - `--link <Container Name>:<Hostname>` Damit ein Container einen anderen Container anhand eines Hostnames finden kann und mit ihm kommunizieren kann, muss der Name und der Hostname gelinkt werden.
- `ps` <br />
Zeigt alle laufenden Container
    - `-a` Zeigt auch die zuletzt gestoppten Container
- `stop <Name/ID>` <br />
Stoppt einen Container anhand seiner ID oder des Namens
- `rm <Name/ID>` <br />
Löschen von einem Container (z.B. gestoppte Container)
- `exec <Name/ID> <command>` <br />
Führt einen Command in dem angegebenen Container aus
- `attach <Name/ID>` <br />
Einen Container aus dem Hintergrund in den Vordergrund holen
- `inspect <Name/ID>` <br />
Gibt Details eines Containers in JSON wieder
- `logs <Name/ID>` <br />
Zeigt die Logs (stdout) eines Containers


## Images
- `images` <br />
Zeigt alle runtergeladenen Images
- `rmi <Name>` <br />
Löscht ein runtergeladenes Image (Kein Container darf mit diesem Image laufen, wenn es gelöscht wird)
- `pull <Name>` <br />
Runterladen von Images von Docker-Hub
- `build <Dockerfile>` <br />
Erstellt ein Image anhand einer Dockerfile
    - `-t <Tag-Name>` Bsp. "chwiese/my-custom-app" als name des Images

# Images

## Erstellen
Datei mit dem Namen "Dockerfile" erstellen und Baseimage und Grundlegende Commands laufen lasse, sowie Dateien kopieren und das gewollte Programm ausführen.

Bsp:
```docker
FROM Ubuntu

RUN apt-get update
RUN apt-get install python

RUN pip install flask
RUN pip install flask-mysql

COPY . /opt/source-code

ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run
```

Build:
```
docker build Dockerfile -t chwiese/my-custom-app
```

Auf Docker-Hub hochladen:
```
docker push chwiese/my-custom-app
```
## Weitere Commands
Den Benutzer, der für die weiteren RUN, CMD oder ENTRYPOINT Befehle benutzt werden soll setzen:
```docker
USER <Benutzer>[:<Gruppe>]
```

Den Ordner, von wo alle folgenden RUN, CMD, ENTRYPOINT, COPY und ADD Befehle ausgeführt werden sollen setzen:
```docker
WORKDIR <Pfad>
```

Eine Environment Variable setzen:
```docker
ENV key=value
```

## CMD vs ENTRYPOINT

### CMD
Den Start-Command spezifiziert:
```docker
CMD ["<Programm>", "<Parameter1>", ...]
```

Command als Parameter übergeben, welcher anstatt des eigentlichen Start-Commands aus der Dockerfile ausgeführt wird (Alles wird ersetzt):
```
docker run <Image Name> <Command>
```

### ENTRYPOINT
Den Start-Command spezifizieren:
```docker
ENTRYPOINT ["<Programm>"]
```

Einen Parameter zu dem in der Dockerfile unter ENTRYPOINT spezifizierten Start-Command hinzufügen (Alles wird dran gehangen):
```
docker run <Image Name> <Parameter>
```

Wenn nach einem Entrypoint noch ein CMD definiert wird, dient dieser quasi als default Parameter, wenn keiner übergeben wird.
```docker
ENTRYPOINT ["<Programm>"]

CMD ["<Parameter>"]
```

# Networking
Container können untereinander anhand deren Namen anstatt einer IP-Adresse kommunizieren.

## Default networks
### Bridge (default)
```
docker run <Image Name>
```
Privates, internes Netzwerk von Docker auf dem Host. Die Container sitzen hinter einem NAT, also gibt es interne IP-Adressen (172.17.0.X).
Um von aussen auf die Container zuzugreifen, muss man Ports vom Host auf die Container mappen.

### None
```
docker run <Image Name> --network=none
```
Die Container sind mit keinem Netzwerk verbunden.

### Host
```
docker run <Image Name> --network=host
```
Benutzt das Netzwerk des Hosts, also kein NAT und kein Port-Mapping. Wenn ein Container auf einm Port hört, ist dies der gleiche Port vom Host System. (Mehrer Container können also nicht den selben Port haben wie bei Bridged)

## User networks
Ein weiteres internes Netzwerk, zu dem standard Docker Netzwerk kann mit
```
docker newtwork create \
    --driver bridge \
    --subnet <Subnetz>/<Größe> (bsp. 182.18.0.0/16)
    custom-isolated-network (name)
```
hinzugefügt werden. Alle Netzwerke können mit 
```
docker network ls
``` 
angezeigt werden.

# Storage

## Volume mount
Ein Volume ist ein Ordner in ```/var/lib/docker/volumes```, in dem Daten von Containern persistent gespeichert werden können.
### Erstellen eines Volumes:
```
docker volume create <Volume Name>
```
Ein volume kann dann von einem Container mit
```
docker run -v <Volume Name>:<Pfad im Container> <Image Name>
```
gemounted werden und alle Daten an dem Pfad im Container werden in dem Volume gespeichert.

Im Image kann auch ein Pfad im Container angegeben werden, welcher gemounted werden soll. Wenn beim Start des Containers dieser Pfad __NICHT__ mit -v auf einen Pfad vom Host gesetzt wird, erstellt Docker automatisch einen Ordner in 
```/var/lib/docker/volumes``` dafür.
```docker
VOLUME ["<Pfad im Container>"]
```

## Bind mount
Ein Bind mount kann ohne das erstellen eines Volumes genutzt werden. Mit
```
docker run docker run -v <Pfad vom Host>:<Pfad im Container> <Image Name>
```
wird einfach der Pfad vom Host in den Pfad im Container gemounted.

# Compose
## Erstellen
Datei mit dem Namen "docker-compose.yml" erstellen.
```yaml
services:
    <Service Name>:
        # Setzt den Namen des Containers
        container_name: <Name>
        # Setzt das Image für den Container
        image: <Image Name>
        ports:
            - <Host Port>:<Container Port>
            - ...
        volumes:
            - <Host Pfad oder Volume Name>:<Container Pfad>
            - ...
        # Erstellt einen Link mit dem Container Namen auf einen Hostname mit
        # dem gleichen Namen
        links:
            - <Service Name> # oder <Service Name>:<Hostname>
            - ...
        # Überschreibt den standard Command
        command: <Command>
        # Überschreibt den standard Entrypoint
        entrypoint: <Command>
        # Der Service wartet auf seine Dependencies, bevor er startet
        depends_on:
            - <Service Name>
            - ...
        environment:
            # Setzt Environment Variablen für den Container
            <Key>: <Value>
            ...
        # Setzt, wann der Container neu gestartet werden soll
        restart: "no", "always", "on-failure", "unless-stopped"
        # Setzt den Terminal modus (-t)
        tty: true
        # Setzt die STDIN (-i)
        stdin_open: true

volumes:
    # hinzufügen eines bereits existierenden Volumes
    <Volume-Name>:
        external:
            name: <Volume-Name>

networks:
    # setzt das standard Netzwerk auf ein externes Netzwerk
    default:
        external: true
        name: <Netzwerk-Name>
```
## Starten
```
docker-compose up
docker-compose up -d <-- Detached mode
```
