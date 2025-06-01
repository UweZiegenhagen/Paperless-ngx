# Paperless-ngx

## Installation im Docker-Container

* Meine Basis: Debian 12 System
* Anlage der notwendigen Verzeichnisse als root

### Paperless-ngx

```
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

https://www.uweziegenhagen.de/?p=4715

## paperless-AI Installation

### Installation mit ChatGPT


### Installation mit LLama
