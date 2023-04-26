
# Rapport Laboratoire 8 - Virtualisation avec Docker

Par : Jessy Grimard-Maheu

------

[TOC]

## Partie 0 - **Création/configuration d'un VPS**

J'ai créer une vm sur [Linode](linode.com) avec les configurations suivantes

- Image : Debian 11

- Région : Toronto, ON

- Linode Plan : Shared CPU --> Nanode 1 GB 

- SSH Keys : Ajouter votre clé publique (`ssh-keygen` pour générer une pair de clé au cas où vous n'en avez pas)

- Password : Mettre un très bon mot de passe

- Private IP : Yes (ça ne coûte rien et c'est plus cool)

  <img src="assets/image-20230316093040559.png" alt="image-20230316093040559" style="zoom:80%;" />

Après m'être créer un VPS sur [Linode](linode.com), j'ai bloqué tous les ports sauf 22 (pour SSH).

Ensuite, je me suis connecté en SSH en tant que l'utilisateur root avec ma clé ssh rsa. Il est ensuite possible de mettre le VPS à jour avec la commande suivante : 

```bash
apt update && apt upgrade -y
```

## Partie 1 - **Installation de Docker sur Debian 11**

J'ai utiliser la procédure suivante  (https://docs.docker.com/engine/install/debian/) pour l'installation de docker sur Debian 11

1. Suppression des anciennes version de docker (il n'en a probablement pas mais c'est une bonne idée de la faire pour être certain)

   ```bash
   # Supprime tout les packages en lien avec docker
   apt-get remove docker docker-engine docker.io containerd runc
   
   # Supprime les vielles images/containers/volumes (s'il y en a)
   rm -rf /var/lib/docker/
   ```

2. Configuration du dépôt

   1. Installation des packages pour permettre à `apt` d'utiliser un dépôt via HTTPS :

      ```bash
      apt install -y ca-certificates curl gnupg
      ```

   2. Ajouter la clé GPG officielle de Docker

      ```bash
      install -m 0755 -d /etc/apt/keyrings
      curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
      chmod a+r /etc/apt/keyrings/docker.gpg
      ```

   3. Configuration du dépôt
      Ajouter le dépôt de docker aux sources d'APT (/etc/apt/sources.list.d/docker.list)

      ```bash
      vim /etc/openvpn/server/server.conf
      
      # et mettre ce texte ⬇️
      
      deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian bullseye stable
      ```

3. Installation de Docker

    1. Mettre les sources d'APT à jour

        ```bash
        apt update
        ```

    2. Installer Docker Engine, containerd et Docker Compose

        ```bash
        apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
        ```

    3. Vérification de la version de Docker Engine
        Lorsque j'ai installer la version *release* de Docker était la version 23.0.3

        ```bash
        docker --version
        
        # La sortie de cette commande devrait ressembler à ceci :
        #
        #root@localhost:~# docker --version
        #Docker version 23.0.3, build 3e7cbfd
        ```

4. Tester Docker Engine avec l'image `hello-world`
   ```bash
   docker run hello-world
   ```

   Voici la sortie de cette commande :

   ```
   En format texte : 
   
   root@localhost:~# docker run hello-world
   Unable to find image 'hello-world:latest' locally
   latest: Pulling from library/hello-world
   2db29710123e: Pull complete 
   Digest: sha256:4e83453afed1b4fa1a3500525091dbfca6ce1e66903fd4c01ff015dbcb1ba33e
   Status: Downloaded newer image for hello-world:latest
   
   Hello from Docker!
   This message shows that your installation appears to be working correctly.
   
   To generate this message, Docker took the following steps:
    1. The Docker client contacted the Docker daemon.
    2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
       (amd64)
    3. The Docker daemon created a new container from that image which runs the
       executable that produces the output you are currently reading.
    4. The Docker daemon streamed that output to the Docker client, which sent it
       to your terminal.
   
   To try something more ambitious, you can run an Ubuntu container with:
    $ docker run -it ubuntu bash
   
   Share images, automate workflows, and more with a free Docker ID:
    https://hub.docker.com/
   
   For more examples and ideas, visit:
    https://docs.docker.com/get-started/
   
   root@localhost:~# 
   ```

   En image :

   <img src="./assets/images/image-20230417000755257.png" alt="image-20230417000755257"  />

## Partie 2 - **Image Debian 10 pour Docker**

1. Téléchargez et démarrez l’image Docker de Debian 10.
   ```bash
   docker run -it --name partie2 debian:buster /bin/bash
   ```

2. Par la suite, puisque le conteneur va déjà avoir été créer, il va falloir lancer le container :

   ```bash
   docker start -i partie2
   ```


**Questions :** 

- <u>*Après une commande exit, montrez à l’aide d’une capture d’écran que le conteneur est à l’arrêt.*</u>
  - ![image-20230417002445291](./assets/images/image-20230417002445291.png)
- <u>*Comment pouvez-vous quitter l’invite d’un conteneur sans arrêter ce dernier ?*</u>
  - Tapez `Ctrl+p` puis `Ctrl+q` cela permet de passer du mode interactif au mode démon (daemon).
    Ex :
    ![image-20230417002753724](./assets/images/image-20230417002753724.png)
- <u>*Quelle est la différence entre docker run et docker start?*</u>
  - `docker run` permet de créer un nouveau conteneur et de le démarrer en une seule commande. `docker start` sert uniquement au démarrage du conteneur. Il ne peut pas lancer d'application sur le conteneur (ex: pour lancer */bin/bash* il faudrait plutôt utiliser `docker exec`) et ne peut pas créer le conteneur si celui-ci n'a pas déjà été créer.


## Partie 3 - **Création d’un DockerFile**

1. Créer le docker file `vim Dockerfile`

   ```dockerfile
   FROM httpd:2.4
   
   # Set apt's frontend to noninteractive cuz we don't want any prompt
   ENV DEBIAN_FRONTEND noninteractive
   
   # Update the repo
   RUN apt-get update
   
   # Upgrade all the old packages (-y is optional cuz we are in noninteractive mode)
   RUN apt-get upgrade -y
   
   # Install the nano text editor (-y is optional cuz we are in noninteractive mode)
   RUN apt-get install -y nano
   
   # Create the user "r2" with the password "r2d2"
   RUN useradd -m -s /bin/bash -c "Robot R2D2" -p "$(openssl passwd -1 r2d2)" r2
   
   # Replace the "It works!" html with "Docker Apache 2" and add my name in the titlebar :]
   RUN echo -n "<html><head><title>Jessy GM</title></head><body><h1>Docker Apache 2</h1></body></html>" > /usr/local/apache2/htdocs/index.html
   ```

2. Build le Dockerfile

   ```bash
   docker build -t apache2-jessy .
   ```

3. Lancer le conteneur pour tester la page web

   ```bash
   docker run -dit --name jessy-apache-app -p 80:80 apache2-jessy
   ```

   ![](./assets/images/image-20230417022652803.png)

   (ça marche ! 🤠)

4. Sauvegarder le nouveau conteneur dans une nouvelle image

   ```bash
   docker commit Nom_conteneur_Debian_Apache_2 Nom_nouvelle_image
   ```
   
5. Voici la nouvelle image (avec screenshot): 
   ![image-20230419232055430](./assets/images/image-20230419232055430.png)
## Partie 4 - **Automatisation du déploiement de conteneurs à l’aide de compose.yaml**

1. Création du fichier docker-compose `vim docker-compose.yml` ou `vim docker-compose.yaml` (il n'y a pas vraiment de différence entre les deux)

   ```yaml
   version: "6.9"
   
   services:
     influxdb:
       container_name: influxdb  
       image: "influxdb:1.8"
       networks:
         - dockernet
       ports:
         - 8083:8083
         - 8086:8086
         - 8089:8089/udp      
       volumes:
         - ./influx:/var/lib/influxdb 
       environment:
         - INFLUXDB_DB=sor2023
         - INFLUXDB_ADMIN_USER=admin
         - INFLUXDB_ADMIN_PASSWORD=admin  
         - INFLUXDB_USERNAME=user
         - INFLUXDB_PASSWORD=password
       restart: unless-stopped  
   
     grafana:
       container_name: grafana
       image: "grafana/grafana:8.2.6"
       networks:
         - dockernet
       ports:
         - 3000:3000
       depends_on:
         - influxdb    
       restart: unless-stopped 
   
   
     mosquitto:
       container_name: mosquitto  
       image: "eclipse-mosquitto:1.6.15"
       networks:
         - dockernet
       ports:
         - 1883:1883
         - 9001:9001
       restart: unless-stopped  
   
     telegraf:
       container_name: telegraf  
       image: "telegraf:1.20"
       networks:
         - dockernet
       ports:
         - 8125:8125/udp
       depends_on:
         - influxdb
       restart: unless-stopped  
   
   networks:
     dockernet:
   ```

2. Démarrage du docker-compose :

   ```bash
   docker compose up -d
   ```

3. On peut maintenant aller sur le port 3000 du vps pour configurer grafana. On peut ajouter la base de donnée _internal ou dans mon cas sor2023

   ![image-20230420001900000](./assets/images/image-20230420001900000.png)

4. On peut ensuite cette BD comme source de donnés en créant un panel
   ![image-20230420002013080](./assets/images/image-20230420002013080.png)
