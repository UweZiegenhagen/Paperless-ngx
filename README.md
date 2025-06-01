# Paperless-ngx

## Installation im Docker-Container

* Meine Basis: Debian 12 System
* Anlage der notwendigen Verzeichnisse als root (via SSH)

### Paperless-ngx

```bash
mkdir /opt/docker/
cd /opt/docker/
mkdir paperless
cd paperless
mkdir paperlessdata
mkdir paperlessmedia
mkdir paperlessexport
mkdir paperlessconsume
mkdir redis
mkdir mariadb
```

docker-compose.yaml

```
version: "3.8"
services:
  broker:
    image: docker.io/library/redis:7
    restart: unless-stopped
    volumes:
      - /opt/docker/paperless/redis:/data

  db:
    image: docker.io/library/mariadb:10
    restart: unless-stopped
    volumes:
     - /opt/docker/paperless/mariadb:/var/lib/mysql
     - /opt/docker/paperless/paperlessexport:/backup
    environment:
      MARIADB_HOST: paperless
      MARIADB_DATABASE: paperless
      MARIADB_USER: paperless
      MARIADB_PASSWORD: paperless
      MARIADB_ROOT_PASSWORD: paperless

  webserver:
    image: ghcr.io/paperless-ngx/paperless-ngx:latest
    restart: unless-stopped
    depends_on:
      - db
      - broker
    ports:
      - "8888:8000"
    volumes:
      - /opt/docker/paperless/paperlessdata:/usr/src/paperless/data
      - /opt/docker/paperless/paperlessmedia:/usr/src/paperless/media
      - /opt/docker/paperless/paperlessexport:/usr/src/paperless/export
      - /opt/docker/paperless/paperlessconsume:/usr/src/paperless/consume
    environment:
      PAPERLESS_REDIS: redis://broker:6379
      PAPERLESS_DBENGINE: mariadb
      PAPERLESS_DBHOST: db
      PAPERLESS_DBUSER: paperless
      PAPERLESS_DBPASS: paperless
      PAPERLESS_DBPORT: 3306
```

docker-compose.env

```
# The UID and GID of the user used to run paperless in the container. Set this
# to your UID and GID on the host so that you have write access to the
# consumption directory.

USERMAP_UID=1001
USERMAP_GID=1001

# Additional languages to install for text recognition, separated by a
# whitespace. Note that this is
# different from PAPERLESS_OCR_LANGUAGE (default=eng), which defines the
# language used for OCR.
# The container installs English, German, Italian, Spanish and French by
# default.
# See https://packages.debian.org/search?keywords=tesseract-ocr-&searchon=names&suite=buster
# for available languages.

PAPERLESS_OCR_LANGUAGES=deu

###############################################################################
# Paperless-specific settings
###############################################################################

# All settings defined in the paperless.conf.example can be used here. The
# Docker setup does not use the configuration file.
# A few commonly adjusted settings are provided below.
# This is required if you will be exposing Paperless-ngx on a public domain
# (if doing so please consider security measures such as reverse proxy)
#PAPERLESS_URL=https://paperless.example.com
# Adjust this key if you plan to make paperless available publicly. It should
# be a very long sequence of random characters. You don't need to remember it.
PAPERLESS_SECRET_KEY=123456789087654321_bitte_aendern!
# Use this variable to set a timezone for the Paperless Docker containers. If not specified, defaults to UTC.

PAPERLESS_TIME_ZONE=Europe/Berlin

# The default language to use for OCR. Set this to the language most of your
# documents are written in.

PAPERLESS_OCR_LANGUAGE=deu

# Set if accessing paperless via a domain subpath e.g. https://domain.com/PATHPREFIX and using a reverse-proxy like traefik or nginx

#PAPERLESS_FORCE_SCRIPT_NAME=/PATHPREFIX
#PAPERLESS_STATIC_URL=/PATHPREFIX/static/ # trailing slash required
#PAPERLESS_CONSUMER_POLLING=30

PAPERLESS_CONSUMER_ASN_BARCODE_PREFIX=ASN
PAPERLESS_CONSUMER_ENABLE_ASN_BARCODE=true
PAPERLESS_CONSUMER_ENABLE_BARCODES=true
PAPERLESS_CONSUMER_BARCODE_SCANNER=ZXING
# Explizites Einschalten der Web API, eventuell nicht notwendig
PAPERLESS_ENABLE_API: 1
```

Ich nutze dann Portainer CE, erstelle einen "paperless" Stack und copy&paste dann die obere Konfiguration in den Stack Editor  bzw. lade die Env Datei.


### Samba, um auf das consume-Verzeichnis von Windows aus zuzugreifen


```
sudo apt install Samba
sudo chmod -R 777 /opt/docker/paperless/paperlessconsume
sudo nano /etc/samba/smb.conf
sudo systemctl restart smbd
smbpasswd -a uwe
```

Am Ende der smb.conf einfügen, dann samba neustarten:

```
[paperlessconsume]
   path = /opt/docker/paperless/paperlessconsume
   browseable = yes
   writable = yes
   valid users = uwe
   create mask = 0660
   directory mask = 0770
   public = no
```






## Grundlagen der Bedienung



## ASNs nutzen und erstellen

Hat man Dokumente, die man in Papierform aufheben muss oder möchte, dann bietet sich die Nutzung einer ASN (Archiv-Seriennummer) an. Mithilfe der ASN kann man alle Dokumente in einem Ordner der Reihe nach ablegen und sie trotzdem schnell wiederfinden. Ich nutze dies für alle Dokumente, die nicht steuerrelevant sind, diese kommen in einen separaten Ordner.

Ich erstelle mir ASN-Aufkleber (für die Aufkleber nutze ich Avery Zweckform 4736)  mit LaTeX, siehe https://www.uweziegenhagen.de/?p=4715.


```LaTeX
\documentclass[a4paper,12pt]{scrartcl}
\usepackage[total={210mm,297mm},top=0mm,left=0mm,bottom=0mm,includefoot]{geometry}
\usepackage[ASN]{ticket} %boxed,cutmark during development
\usepackage{qrcode} 
\usepackage{forloop}
\usepackage[T1]{fontenc}
 
% https://tex.stackexchange.com/questions/716116/generate-sequential-padded-barcodes-with-qrcode?noredirect=1#comment1780003_716116
\makeatletter
\newcommand{\padnum}[2]{%
  \ifnum#1>1 \ifnum#2<10 0\fi
  \ifnum#1>2 \ifnum#2<100 0\fi
  \ifnum#1>3 \ifnum#2<1000 0\fi
  \ifnum#1>4 \ifnum#2<10000 0\fi
  \ifnum#1>5 \ifnum#2<100000 0\fi
  \ifnum#1>6 \ifnum#2<1000000 0\fi
  \ifnum#1>7 \ifnum#2<10000000 0\fi
  \ifnum#1>8 \ifnum#2<100000000 0\fi
  \ifnum#1>9 \ifnum#2<1000000000 0\fi
  \fi\fi\fi\fi\fi\fi\fi\fi\fi
  \expandafter\@firstofone\expandafter{\number#2}%
}
\makeatother
 
\begin{filecontents*}[overwrite]{ASN.tdf}
\unitlength=1mm
\hoffset=-16mm
\voffset=-8mm
\ticketNumbers{4}{12}
\ticketSize{45.7}{21.2} % Breite und Höhe der Labels in mm
\ticketDistance{2.5}{0} % Abstand der Labels
\end{filecontents*}
 
%reset background 
\renewcommand{\ticketdefault}{}%
 
 
\newcounter{asn}
% start value of the labels
\setcounter{asn}{1}
 
\newcommand{\mylabel}{
\ticket{%
\hspace*{4mm}\raisebox{9mm}[4mm][2mm]{%
\qrcode[height=1.4cm]{ASN\padnum{5}{\value{asn}}}%
~\texttt{\large ASN\padnum{5}{\value{asn}}}
}
\stepcounter{asn}
}
}
 
\begin{document}
 
% just for the loop
\newcounter{x}
% create 48 labels
\forloop{x}{1}{\value{x} < 49}{
\mylabel%
}
 
%print on Avery  Zweckform 4736
% in original size, not scaled to fit page
\end{document}
```

## paperless sichern und wiederherstellen

Auf der Synology:

```
cd /volume1/docker/paperlessngx
docker exec paperlessngx_webserver_1 document_exporter -f -z ../export
```

Auf dem Debian 12 System als root

```
cd /volume1/docker/paperlessngx
docker exec paperless-webserver-1 document_exporter -f -z ../export
```

Da sichert nicht die Mariadb Datenbank, diese muss separat gesichert werden.

Login auf die Mariadb via Console auf den DB-Server von Portainer aus.

```bash
mariadb -upaperless -ppaperless
```

https://mariadb.com/kb/en/container-backup-and-restoration/


```SQL
mariadb-dump --all-databases -l -uroot -ppaperless > backup/db.sql'
```



## paperless-AI Installation

### Installation mit ChatGPT


### Installation mit LLama
