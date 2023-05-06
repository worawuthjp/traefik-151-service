# traefik-151-service
Configuration
 
We will be now taking two backend servers (Nginx container) and one Traefik container in the same Docker network zone. You have to add your domain instead “traefik.domain.com”.

 
Let’s begin by creating a directory in your home location.

 
$ mkdir traefik && cd traefik
 
Now create a docker network using the below command, this helps to reach the container from their name.

 
$ docker network create web_zone
 
Traefik.yaml configuration
 
Firstly, create a file named “traefik.yaml” and paste the following content:

 
# Static configuration
entryPoints:
    unsecure:
        address: :80
    secure:
        address: :443

certificatesResolvers:
    myresolver:
        acme:
            email: admin@domain.com
            storage: acme.json
            httpChallenge:
                entryPoint: unsecure      
providers:
      file:
      filename: tls.yaml
      watch: true
 
From the above code, the entry Points are the front-end listing with services and ports. certificatesResolvers is to use an on-demand Letsencrypt certificate and the Providers are the file to define routers or middlewares services.

 
File configuration
 
Now, in the same directory create one more file “tls.yaml” that we have defined in the provider section and paste the given 0yaml configuration as shown:

 
>http:
    routers:
        http_router:
            rule: "Host(`traefik.yourdomain.com`)"
            service: allbackend
        https_router:
            rule: "Host(`traefik.domain.com`)"
            service: allbackend
            tls:
                certResolver: myresolver
                options: tlsoptions
    services:
        allbackend:
            loadBalancer:
                servers:
                    - url: "http://server1/"
                    - url: "http://server2/"          
tls:
    options:
        tlsoptions:
            minVersion: VersionTLS12
 
Further, create the following file to store the Let’s Encrypt certificate:

$ touch acme.json
$ chmod 600 acme.json
 
Docker-compose for traefik
 
Additionally, create a container using docker-compose and map to 80, 443 ports, define your domain name, and create a file docker-compse.yml:

 
$ vim docker-compose.yml
later paste the following configuration:

 
version: '3'
services:

  traefik:
    image: traefik:latest
    command: --docker --docker.domain=yourdomain.com
    ports:
      - 80:80
      - 443:443
    networks:
      - web_zone
    volumes:
      - /run/docker.sock:/run/docker.sock
      - ./traefik.yaml:/traefik.yaml
      - ./tls.yaml:/tls.yaml
      - ./acme.json:/acme.json
    container_name: traefik
    restart: always
networks:
  web_zone:
      external: true
 
Backend server
 
Let’s run two backend servers using Nginx image. Make a directory for this:

$ mkdir ~/traefik/backend && cd ~/traefik/backend/

 
Create two index files as below.
 
echo "
 Hello server 1
" > index-server1.html
 
echo "
 Hello server 2/h1>" > index-server2.html
 
Docker compose file to run on two Nginx backend servers
 
The following is the simple compose file that makes two Nginx containers. Create a docker-compse.yml file and paste the following code:

version: '3'
services:
  myserver1:
    image: nginx
    container_name: nginx1
    restart: always
    volumes:
      - ./index-server1.html:/usr/share/nginx/html/index.html
    networks:
      - web_zone
  myserver2:
    image: nginx
    container_name: nginx2
    restart: always
    volumes:
      - ./index-server2.html:/usr/share/nginx/html/index.html
    networks:
      - web_zone
networks:
  web_zone:
        external: true
 
You need to start the Docker by running the container. First up the Nginx backend container by using the command:

 
:~/traefik/backend$ docker compose up -d
 
Two containers must be running, and this can be confirmed from the command:

 
:~/traefik/backend$ docker ps
 
Now, go back to the directory and run traefik load balancer.

 
:~/traefik$ docker compose up -d
 
Make sure the traefik container is up and running fine.

 
:~/traefik$ docker ps
 
Browse the site
 
Open a browser and type your domain name http://traefik.domain.com. Page will display as “Hello server1“.

echo "< h1 > Hello server 1< /h1 >" > index-server1.html
Likewise, if you refresh the page next will be routed to the second backend page. This is called as default routing algorithm in traffic.

echo "< h1 > Hello server 2< /h1 >" > index-server1.html
You can also check that the certificate is issued by Letsencrypt while the container is up. Just access it with HTTPS before the link” https://traefik.domain.com”