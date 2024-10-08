# Foreman Training - Teil 2 - Content

## Login in Katello

Im Browser:

    https://foreman.betadots.training

## Content Credentials

Unter Content Credentials kann man GPG Keys und/oder SSL Zertifikate hinterlegen.

### GPG Key einbinden

    Foreman Login
      -> Content
        -> Content Credentials
          -> Create Content Credential

Name: CentOS 9 Stream

CentOS GPG Key kopieren vom mirror [http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-Official](http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-Official)

    Save

Wiederholen fuer PostgreSQL GPG Key mit [https://download.postgresql.org/pub/repos/yum/keys/PGDG-RPM-GPG-KEY-RHEL](https://download.postgresql.org/pub/repos/yum/keys/PGDG-RPM-GPG-KEY-RHEL)

Wiederholen für Debian 12: [https://ftp-master.debian.org/keys/archive-key-10.asc](https://ftp-master.debian.org/keys/archive-key-10.asc)

## Repository erzeugen

ACHTUNG: Das Anlegen von lokalen Repositories benoetigt viel Plattenplatz und viel Bandbreite.
Diese Schritte sollten nur gemacht werden, wenn man:

- eine schnelle Internetanbindung hat (min 50 MBit)
- genug Plattenplatz verfuegbar ist (min 20 GB)

Idealerweise legen die Teilnehmer nur das "kleine" PostgreSQL Repository an.

### Repository anlegen

#### PostgreSQL 14 Repositories - klein (ca 2 GB - Dauer: ca 5 Minuten)

    Foreman Login
      -> Content
        -> Products
          -> Create Product

Name: PSQL

GPG Key: Auswählen

Save

    -> New Repository

Name: 14-RHEL9

Type: yum

Upstream URL: https://download.postgresql.org/pub/repos/yum/14/redhat/rhel-9-x86_64/

#### CentOS Repository - gross (20 GB - Dauer: ca 1 Stunde)

    Foreman Login
      -> Content
        -> Products
          -> Repo Discovery

Repository Type: Yum Repositories

URL: `http://mirror.centos.org/centos/8` (oder lokaler Mirror)

Click Discover

Das kann einige Zeit dauern (5 min und mehr).
Katello holt sich dabei die Metainformationen der gesamten CentOS 8 Repositories.

`os/x86_64` und `updates/x86_64` auswaehlen.

Klick: Create selected

Name: CentOS8

GPG Key: aus Liste auswaehlen

Verify SSL: nur aktivieren, wenn man upstream SSL pruefen moechte.

Run Repository Creation

#### Debian Repository - gross (min 30GB - Dauer: ca 3 Stunden)

ACHTUNG: das Debian Repository ist zu gross fuer die Training VM!!!!
Der Sync bricht ab mit `no space left on device`!

Debian Repositories werden anders behandelt.
Hier muss zuerst ein Produkt und beim Repository eine URL angegeben werden.

Zusätzlich muss die Distribution und Komponente sowie Architektur angegeben werden.

    Foreman Login
      -> Content
        -> Product
          -> Create Product

Name angeben -> Save

In der Uebersicht: "New Repository" auswaehlen.

Name angeben: Debian 10

Bei "Type" `deb` auswaehlen und die Repo Informationen eintragen:

Upstream URL: `http://ftp.de.debian.org`

Im Linuxhotel: `http://debian/`

Release: stable/unstable/buster/bookworm/trixie/...

Components: main, free, non-free, ...

Architectures: amd64, arm, i386, ...

Verify SSL: aus

Mirror on Sync: an

Publish via HTTP: an

GPG Key: NICHT auswaehlen, der oben erzeugte GPG Key scheint verkehrt zu sein.

Save

#### SLES Repositories - gross

Fuer SLES gibt es 2 Wege:

1. mit einem SMT Server
2. SCCM Plugin

Weitere Details findet man auf der Webseite von Foreman: [https://theforeman.org/plugins/katello/nightly/user_guide/suse_content/index.html](https://theforeman.org/plugins/katello/nightly/user_guide/suse_content/index.html)

Wir beschreiben den Weg mit SCCM Plugin.
Wenn SMT Server genutzt wird, dann wird ein normales Repository angelegt.

Installation: `yum install -y tfm-rubygem-foreman_scc_manager`
Datenbank aktualisieren: `foreman-rake db:migrate`
Neustart Foreman: `forman-maintain service restart --only foreman`

Nun hat man unter Content einen neuen Eintrag: SuSE Subscription
Hier kann man einen SCCM Account angeben.

#### Docker Registry

## Docker (optional)

    Katello Login
      -> Content
        -> Products
          -> Create Product

Name: Fedora

Rest leer lassen

Save

Repository -> New Repository

Name: Fedora SSH

Type: Docker

URL: `https://registry.hub.docker.com`

Upstream Repository Name: fedora/ssh

Save

## Repositories Syncen

Foreman kann Repositories selbstaendig synchronisieren.

Dafuer benoetigt man einen Sync Plan

### Sync Plan

    Foreman Login
      -> Content
        -> Sync Plans

Create Sync Plan

Name: daily

Interval: daily

Start Date: Datum von heute

Start Time: 1,5 Stunden vorher (UTC time auf System)

Save

Products Tab auswählen.

Add auswählen

Ein Repository auswählen und Klick auf "Add selected"

    Foreman Login
      -> Content
        -> Sync Status

Repository auswaehlen und sync starten

### Initiales Syncen

    Katello Login
      -> Content
        -> Products
          -> CentOS8
            -> os x86_64

Download Policy: Immediate

Select Action -> Sync Now

## Content Views

Mit content views koennen verschiedene Konstellationen und Versionen von Repos verwaltet werden.

z.B. Basis OS Repo + Monitoring + eigenes Repo

    Foreman Login
      -> Content
        -> Content Views

Content Views koennen versioniert werden und man kann neue Versionen ueber das Webinterface erzeugen.

    Content Views
      -> Create Content View

Name: Katello Client

Repo hinzufuegen:

    Yum Content
      -> Repositories
        -> Add

Repository auswaehlen

Add Repositories

Jetzt kann eine neue Version der Content View erzeugt werden: "Publish New Version"

Save

Bevor wir diese Content View "promoten" koennen muessen wir die Lifecycle Environments erzeugen.

## Lifecycle Environments

Mit Lifecycle Environments verwaltet man Content Views (hinzufuegen, promoten, entfernen).
Das Default Lifecycle Environment nennt sich Library. Innerhalb dieser Library koennen weitere Environments angelegt werden.

z.B. Production, Testing, Pre-Prod

Ueber die Content Views kann man nun Content an ein Lifecycle Environment "promoten".

    Foreman Login
      -> Content
        -> Lifecycle
          -> Lifecycle Environments
            -> Create Environment Path

Name: Development

Save

## Alles zusammen bringen

Eine Content View Version kann man an ein Lifecycle Environment verknuepfen:

    Foreman Login
      -> Content
        -> Content Views
          -> Katello Client
            -> Promote

Lifecycle Environment(s) auswaehlen (hier: Production)

Bevor wir nun einen Host an diese Lifecycle Environment anbinden koennen brauchen wir einen Activation Key oder muessen uns mit User/Password am Katello Server authentifizieren.

Wir machen das mit einem Activation Key:

## Activation Key

  Foreman Login
    -> Content
      -> Lifecycle
        -> Activation Keys
          -> Create Activation Key

Name: Katello-Client
Environment: Development auswaehlen
Content View: Katello Client auswaehlen

oder: keine Content View angeben und nach dem Erzeugen des Activation Keys unter Subscriptions die gewuenschten Repositories asuwaehlen.

Save

## Content Hosts

Unter dem Eintrag Content Host kann man den Stand der Pakete auf einzelnen Systemen pruefen und modifizieren.

    Foreman Login
      -> Hosts
        -> Content Hosts

Host auswählen, Register auswählen.

OS und Content View und Activation Key angeben.

Shell command copy-paste.

Beispiel:

    curl -sS  'https://foreman.betadots.training/register?activation_keys=Library&location_id=2&operatingsystem_id=1&organization_id=1&update_packages=false' -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjo0LCJpYXQiOjE2NzkzMjEzMzcsImp0aSI6IjJiNTQ3YjBiNGU1ZDkyNmZlYTgyNTczMTIwYjYyMmRlODdiNDg5ZDFlNzg2ZTgwOGYxYzIzMDcxNWMzYTM5N2UiLCJleHAiOjE2NzkzMzU3MzcsInNjb3BlIjoicmVnaXN0cmF0aW9uI2dsb2JhbCByZWdpc3RyYXRpb24jaG9zdCJ9.zc_eRJ0zUF6K-49BpfD03sC9iiAU_kt1pryYVxrOx84' | bash

Weitere Informationen: [https://theforeman.org/plugins/katello/3.14/installation/clients.html#manual](https://theforeman.org/plugins/katello/3.14/installation/clients.html#manual)

ACHTUNG: in Katello 4 wurde von katello-agent auf Remote Execution gewechselt!!

## Host Collections

Innerhalb von Host Collections werden die folgenden Komponenten in Zusammenhang gesetzt:

- Products (Repositories)
- Nodes

    Foreman Login
      -> Hosts
        -> Host Collections
          -> Create Host Collection

Name angeben (Host Collection 1) und den neu angelegten Host hinzufügen.

Weiter geht es mit [Teil 3 Konfigurationsmanagement](../03_configmanagement)
