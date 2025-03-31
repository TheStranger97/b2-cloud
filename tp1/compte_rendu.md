# TP1

# 1. Install

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

# 2. vérif install

# 3. Lancement des conteneurs

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