# TP1

# Part I : Docker basics

## 1. Install

on désisntalle les packages possiblement déjà installés

```
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

script d’install

```
curl -fsSL https://get.docker.com -o get-docker.sh
```

on l’execute

```
sudo sh get-docker.sh
```

docker est automatiquement lancé avk le script mais au cas où

```
sudo systemctl start docker
```


## 3. Lancement des conteneurs

🌞 **Utiliser la commande `docker run`**

```
docker run --name web -d -p 9999:80 nginx
```

🌞**Rendre le service dispo sur internet**

```
C:\Users\samue>curl 20.160.34.183:9999
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

🌞 **Custom un peu le lancement du conteneur**

```
docker run --name meow -d -v $(pwd)/index.html:/usr/share/nginx/html/index.html -v $(pwd)/nginx.conf:/etc/nginx/conf.d/nginx.conf -p 9999:7777 -m="512m" nginx
```


## Part II : Images

🌞 Dans les deux cas, j'attends juste votre Dockerfile dans le compte-rendu
```
FROM ubuntu

RUN apt update -y

RUN apt install -y apache2

RUN mkdir /etc/apache2/logs

COPY apache2.conf /etc/apache2/

RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf

COPY index.html /var/www/html/

CMD [ "apache2", "-DFOREGROUND" ]
```


## Part III : docker-compose


docker-compose.yml
```
services:

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: wiki
      POSTGRES_PASSWORD: wikijsrocks
      POSTGRES_USER: wikijs
    logging:
      driver: none
    restart: unless-stopped
    volumes:
      - db-data:/var/lib/postgresql/data

  wiki:
    image: ghcr.io/requarks/wiki:2
    depends_on:
      - db
    environment:
      DB_TYPE: postgres
      DB_HOST: db
      DB_PORT: 5432
      DB_USER: wikijs
      DB_PASS: wikijsrocks
      DB_NAME: wiki
    restart: unless-stopped
    ports:
      - "80:3000"

volumes:
  db-data:
```

```
docker compse up -d
```

```
docker ps -a
CONTAINER ID   IMAGE                     COMMAND                  CREATED          STATUS          PORTS                                                   NAMES
0305e6790897   ghcr.io/requarks/wiki:2   "docker-entrypoint.s…"   19 minutes ago   Up 19 minutes   3443/tcp, 0.0.0.0:9999->3000/tcp, [::]:9999->3000/tcp   docker-compose-wiki-1
714c5d4d2670   postgres:15-alpine        "docker-entrypoint.s…"   19 minutes ago   Up 19 minutes   5432/tcp                                                docker-compose-db-1
```

