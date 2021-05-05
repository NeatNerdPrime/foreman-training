# Foreman Training - Teil 3 - Provisionieren Debian

TFTP images werden von folgender URL heruntergeladen:
http://ftp.debian.org/debian/dists/buster/main/installer-amd64/current/images/netboot/debian-installer/amd64/

Der host wird anhand des Mirrors gesetzt.
Bei lokalen Mirrorn unbedingt beachten, dass die Installer Images mit gemirrort werden.

## OS anlegen

Foreman Login -> Hosts -> Operating System -> Create Operating System

Tab Operating System -> Name: Debian

Tab Operating System -> Major: 10

Tab Operating System -> Minor: 3

Tab Operating System -> Release Name: buster

Tab Operating System -> Description: Debian 10.3

Tab Operating System -> Architecture: x86_64

Tab Partition Table -> Preseed Default

Tab Installation Media -> Debian Mirror

Achtung: Templates können erst nach dem nächsten Schritt ausgewählt werden!

Submit

### Provisionierungs Templates mit OS Assoziieren

Foreman Login -> Hosts -> Provisioning Templates

1. Stelle: Templates mit OS Assoziieren

1.a. kind = PXELinux

"Preseed default PXELinux" auswaehlen -> Association -> OS auswaehlen

1.b. kind = provision

"Preseed default" auswaehlen -> Association -> OS auswaehlen

1.c. kind = finish

"Preseed default finish" auswaehlen -> Association -> OS auswaehlen

1.d. PXE bauen

"Build PXE Default"

### Installation media

Wenn man einen eigenen anstelle der Debian Mirror verwenden moechte:

Foreman Login -> Hosts -> Installation Media -> Create medium

Name eintragen, bei "Path" den http Pfad eintragen.
Achtung: auf Namensaufloesung achten!

Operatingsystem Family angeben!

Wenn man ein Lifecycle Environment nutzen moecht,e kann man dies hier auswaehlen.

### OS mit Provisionierungs Templates assoziieren

Foreman Login -> Hosts -> Operating Systems

OS auswaehlen

2. Stelle: OS mit Templates Assoziieren

- Partition Table -> "Preseed default" auswaehlen
- Installation media -> "Debian mirror" auswaehlen (oder das vorher angelegten Installation media auswaehlen)
- Templates -> alle Templates auswaehlen, die man auswaehlen kann.

Submit

## Host erzeugen in VirtualBox

Hinweis: die Images werden initial in eine RAM Disk geladen. Deshalb benoetigt die VM mindesten 1516 MB RAM.

New -> Host -> 2048 MB RAM -> 8 GB HDD

Boot Einstellungen aendern: 1. Festplatte -> 2. Netzwerk
Netzwerk aendern: gleiches vboxnet, wie foreman.example42.training
MAC Adresse notieren

## Host in Foreman anlegen

Foreman Login -> Hosts -> Create Host

Host Tab:

- Name: hostname (ohne Domain)
- Hostgroup: Provision from foreman
- Environment: Production (sollte automatisch aus der Hostgroup rausfallen)

Interfaces Tab:

- Edit
  - Mac Adresse eintragen und OK zum speichern

Operating System Tab:

- Operating System: auswaehlen (Debian 10.3...)
- Media: Mirror waehlen
- Partition Table: Preeseed default
- PXE Loader: PXELinux BIOS

Submit

## Host in VirtualBox starten

Weiter mit Teil 6 [Deprovisionieren](../06_deprovisioning)
