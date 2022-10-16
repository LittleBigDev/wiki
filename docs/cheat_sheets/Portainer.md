# Portainer
## First-time setup of a new Portainer Server
```bash
docker volume create portainer_data
docker run -d -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```

## Access Portainer web app
Visit [https://localhost:9443](https://localhost:9443).  
If it is your first time visiting the app, you will have to follow the [initial setup](https://docs.portainer.io/start/install/server/setup).

Optionnally, add an entry in your `/etc/hosts` file : 
``` title="/etc/hosts"
127.0.0.1	portainer.local
```

And then, access web app visiting [https://portainer.local:9443](https://portainer.local:9443).

## Upgrading on Docker
```bash
docker stop portainer
docker rm portainer
docker run -d -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```
