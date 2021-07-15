# RBPi-OrAny-Multiwordpress
Multiple Wordpress Websites in Docker with Let's Encrypt and Nginx-Proxy in a single Raspberry Pi Server (or any server)

## Requirements:
- A linux-based server
- Hostnames (DNSs) to solve your server's ip address
- Not having anything else (program or container) listening to ports 80 and 443 as these will be used by nginx-proxy
- Docker installed
- Docker compose installed
- Curiosity :)


## Directories tree

```
/RBPi-OrAny-Multiwordpress
.
└── docker
    ├── nginx-proxy
    ├── portainer   
    └── wordpress
        ├── website1
        ├── website2
        └── websiteN
```

Download this repository with
```
git clone http://
```
and get easily going with docker-compose in the nginx-proxy directory and then with each wordpress website.

### Open the nginx-proxy directory and modify the docker-compose.yml file with
```
sudo nano nginx-proxy-multiwordpress/docker/nginx-proxy/docker-compose.yml
```
### to add your email in DEFAULT_EMAIL=your@email.com to get Let's Encrypt notifications in 

```
version: "3"
services:
  nginx-proxy:
    image: jwilder/nginx-proxy
    container_name: nginx-proxy
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/tmp/docker.sock:ro
      - letsencrypt-certs:/etc/nginx/certs
      - letsencrypt-vhost-d:/etc/nginx/vhost.d
      - letsencrypt-html:/usr/share/nginx/html
  letsencrypt-proxy:
    image: jrcs/letsencrypt-nginx-proxy-companion
    container_name: letsencrypt-proxy
    restart: unless-stopped
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - letsencrypt-certs:/etc/nginx/certs
      - letsencrypt-vhost-d:/etc/nginx/vhost.d
      - letsencrypt-html:/usr/share/nginx/html
    environment:
      - DEFAULT_EMAIL=your@email.com
      - NGINX_PROXY_CONTAINER=nginx-proxy


networks:
  default:
    external:
      name: nginx-proxy

volumes:
  letsencrypt-certs:
  letsencrypt-vhost-d:
  letsencrypt-html:
```

### go to the nginx-proxy directory
```
cd nginx-proxy-multiwordpress/docker/nginx-proxy
```
### and launch the docker container
```
sudo docker-compose up -d
```

### once done, proceed to modify the docker-compose file with
```
cd ../
```
```
sudo nano nginx-proxy-multiwordpress/docker/wordpress/website1/docker-compose.yml
```
### for each website with your domain name, website name and mysql/mariadb credentials

## Example website1 docker-compose.yml file
```
version: "3"

services:
  db_node_domain_website1:
    #image: mariadb:latest
    image: mariadb:10.5.11
    volumes:
      - db_data:/var/lib/mysql
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: yourPASSWORD
      MYSQL_DATABASE: website1-wp-db
      MYSQL_USER: website1-wp-db-user
      MYSQL_PASSWORD: yourPASSWORD
    container_name: website1-wordpress-db

  wordpress_website1:
    depends_on:
      - db_node_domain_website1
    #image: arm64v8/wordpress:latest
    #image: arm32v7/wordpress:latest
    #image: wordpress:latest
    image: arm64v8/wordpress:php7.4
    volumes:
      - wp_data:/var/www/html
    expose:
      - 80
    restart: unless-stopped
    environment:
      VIRTUAL_HOST: website1.com, www.website1.com
      LETSENCRYPT_HOST: website1.com, www.website1.com
      WORDPRESS_DB_HOST: db_node_domain_website1:3306
      WORDPRESS_DB_USER: website1-wp-db-user
      WORDPRESS_DB_PASSWORD: yourPASSWORD
      WORDPRESS_DB_NAME: website1-wp-db
    container_name: website1-wordpress
volumes:
  db_data:
  wp_data:

networks:
  default:
    external:
      name: nginx-proxy

```



## Make sure to have replaced all the "websiteN" with your website name!

### Tip 1: Use DietPi
If you will be doing this on a Raspberry Pi, I highly suggest you use [DietPi](https://dietpi.com/). It is a really solid and powerful yet minimal and efficient linux image which gets the most out of your RBPi. For context, I can currently run 2 - 3 simultaneous websites on a RBPi 3B+ (1GB RAM) which experience really low traffic before actually maxing out the RAM. So for small personal projects / blogs it is okay. A RBPi 4 with 4 - 8 GB RAM memory should perform much better.

### Tip 2: If you need to perform any changes to any of your docker-compose.yml files, modify it and then rebuild it and launch it with:
```
sudo docker-compose up -d --build --force-recreate --no-deps --remove-orphans
```

### Tip 3: To launch multiple wordpress websites in a non Raspberry Pi 64 bit (arm64v8) system, modify the docker-compose file for each website uncommenting the desired system in:
```
#image: arm64v8/wordpress:latest
#image: arm32v7/wordpress:latest
#image: wordpress:latest
```

### Tip 4: Last but not least, installing wordpress this way makes it very easy for you to migrate your wordpress websites to another host/machine. Simply copy the files (stop the containers before) from the current install usually located in  "/var/lib/docker" (you can verify with Portainer) to the same directory on the new host machine and then proceed to repeat the steps above. If everything went well you should have your wordpress sites with everything as you left them.

---

## Extra:
Install Portainer to manage your docker containers and volumes, easily debug and visualize cointainers running.

You simply have to go to portainer's directory and run
```
sudo docker-compose up -d
```

Once installed, it will become available to access and configure on port 9002 so you can access it like this: http://yourRBPIsIP:9002/

Example:
http://192.168.0.20:9002/

---

## Credits:
Big thanks to to the creators of all the libraries used in this project, they are the MVPs.
- [jwilder/nginx-proxy](https://github.com/nginx-proxy/nginx-proxy)
- [jrcs/letsencrypt-nginx-proxy-companion](JrCs/docker-letsencrypt-nginx-proxy-companion)

And also special thanks to Oleg Ishenko since this tutorial is mostly based on his tutorial [Use Nginx-Proxy and LetsEncrypt Companion to Host Multiple Websites](https://www.singularaspect.com/use-nginx-proxy-and-letsencrypt-companion-to-host-multiple-websites/)