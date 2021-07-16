# kronos-docker
Kronos Docker  

tester for Kronos  
Build  
```
docker build -t kronos .
```
Run  
```
docker run -d -v ~/kronosdata:/root/Kronos/DATA -p 3000:3000  --name kronos kronos
```  
docker-compose.yml  
```
version: "3.3"

services:

  traefik:
    image: "traefik:v2.4"
    container_name: "traefik"
    command:
      #- "--log.level=DEBUG"
      - "--api.insecure=true"
      #- "--api.dashboard=true"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.myresolver.acme.dnschallenge=true"
      - "--certificatesresolvers.myresolver.acme.dnschallenge.provider=cloudflare"
      #- "--certificatesresolvers.myresolver.acme.caserver=https://acme-staging-v02.api.letsencrypt.org/directory"
      - "--certificatesresolvers.myresolver.acme.email=email@example.com"
      - "--certificatesresolvers.myresolver.acme.storage=/letsencrypt/acme.json"
    ports:
      - "80:80"
      - "443:443"
      - "8080:8080"
    environment:
      - "CF_API_EMAIL=email@example.com"
      - "CF_API_KEY=SECRETAPIKEY"
    volumes:
      - "./letsencrypt:/letsencrypt"
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
  kronos:
    image: "kronos"
    container_name: "kronos-headless"
    restart: always 
    volumes:
        - '~/kronosdata:/root/Kronos/DATA'
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.kronos.rule=Host(`kronos.example.com`)"
      - "traefik.http.routers.kronos.entrypoints=web"
      - "traefik.http.routers.kronos.entrypoints=websecure"
      - "traefik.http.routers.kronos.tls.certresolver=myresolver"
      - traefik.webservice.frontend.entryPoints=http,https,ws,wss
      - "traefik.http.services.kronos.loadbalancer.server.port=3000"  
```
