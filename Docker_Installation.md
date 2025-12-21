# Docker Installation

As described in https://www.thomas-krenn.com/en/wiki/Docker_installation_on_Debian_12

## Install packages required for the installation

sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

## Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

## Download GPG key and store repository in the system

curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker.gpg] https://download.docker.com/linux/debian bookworm stable" |tee /etc/apt/sources.list.d/docker.list > /dev/null 
apt update 

## Install Docker packages

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin


# Portainer Installation


docker run -d \
  -p 8000:8000 -p 9443:9443 \
  --name portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /mnt/datenpool/docker/portainer:/data \
  portainer/portainer-ce:lts


In case Portainer cannot connect to the local Docker:

systemctl edit docker.service


Insert 

[Service] 
Environment=DOCKER_MIN_API_VERSION=1.24


Restart with 

systemctl restart docker