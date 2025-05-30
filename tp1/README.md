# TP1

# Part I : Docker basics

## 1. Install

🌞 Installer Docker votre machine Azure

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


# Part II : Images

🌞 Construire votre propre image

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


# Part III : docker-compose

🌞 Installez un WikiJS en utilisant Docker


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
docker compose up -d
```

```
docker ps -a
CONTAINER ID   IMAGE                     COMMAND                  CREATED          STATUS          PORTS                                                   NAMES
0305e6790897   ghcr.io/requarks/wiki:2   "docker-entrypoint.s…"   19 minutes ago   Up 19 minutes   3443/tcp, 0.0.0.0:9999->3000/tcp, [::]:9999->3000/tcp   docker-compose-wiki-1
714c5d4d2670   postgres:15-alpine        "docker-entrypoint.s…"   19 minutes ago   Up 19 minutes   5432/tcp                                                docker-compose-db-1
```

## 3. Make your own meow


🌞 Vous devez :

- construire une image qui
  - contient python3
  - contient l'application et ses dépendances
  - lance l'application au démarrage du conteneur

```
FROM ubuntu

RUN apt update -y && apt install -y python3-pip

COPY app python-app

RUN pip install --break-system-packages -r python-app/requirements.txt

CMD [ "python3", "python-app/app.py" ]
```

```
docker build . -t python-app
```

```
nano docker-compose.yml
```

```
services:
  db:
    image: redis:latest
    container_name: redis_db
    restart: always
    ports:
      - '6379:6379'

  pyapp:
    image: python-app:latest
    container_name: pyapp
    restart: always
    ports:
      - '9999:8888'
    depends_on:
      - db
```

```
docker compose up -d
```

```
docker ps -a
CONTAINER ID   IMAGE               COMMAND                  CREATED         STATUS         PORTS                                         NAMES
d39b758d589e   python-app:latest   "python3 python-app/…"   7 seconds ago   Up 6 seconds   0.0.0.0:9999->8888/tcp, [::]:9999->8888/tcp   pyapp
ca10445636df   redis:latest        "docker-entrypoint.s…"   7 seconds ago   Up 7 seconds   0.0.0.0:6379->6379/tcp, [::]:6379->6379/tcp   redis_db
```

# Part IV : Docker security


## 1. Le groupe docker

🌞 Prouvez que vous pouvez devenir root

```
azureuser@wawaa:~/travail/tp1/part3$ cat /etc/shadow
cat: /etc/shadow: Permission denied
azureuser@wawaa:~/travail/tp1/part3$ docker run -it ubuntu
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
5a7813e071bf: Already exists
Digest: sha256:72297848456d5d37d1262630108ab308d3e9ec7ed1c3286a32fe09856619a782
Status: Downloaded newer image for ubuntu:latest
root@95478a656f12:/# cat /etc/shadow
root:*:20115:0:99999:7:::
daemon:*:20115:0:99999:7:::
bin:*:20115:0:99999:7:::
sys:*:20115:0:99999:7:::
sync:*:20115:0:99999:7:::
games:*:20115:0:99999:7:::
man:*:20115:0:99999:7:::
lp:*:20115:0:99999:7:::
mail:*:20115:0:99999:7:::
news:*:20115:0:99999:7:::
uucp:*:20115:0:99999:7:::
proxy:*:20115:0:99999:7:::
www-data:*:20115:0:99999:7:::
backup:*:20115:0:99999:7:::
list:*:20115:0:99999:7:::
irc:*:20115:0:99999:7:::
_apt:*:20115:0:99999:7:::
nobody:*:20115:0:99999:7:::
ubuntu:!:20115:0:99999:7:::
```

🌞 Utilisez Trivy 


## 2. Scan de vuln


- [Base de donnée de wiki](#scan-bd-wikijs)
- [Wiki js](#scan-wiki-js)
- [Apache](#scan-apache)
- [Nginx](#scan-nginx)



### Scan bd wikijs
```
azureuser@wawaa:~/travail/docker-compose$ trivy image postgres:15-alpine
2025-03-31T20:28:23Z    INFO    [vulndb] Need to update DB
2025-03-31T20:28:23Z    INFO    [vulndb] Downloading vulnerability DB...
2025-03-31T20:28:23Z    INFO    [vulndb] Downloading artifact...        repo="mirror.gcr.io/aquasec/trivy-db:2"
61.68 MiB / 61.68 MiB [----------------------------------------------------------------------------------------------------------------------------------------------------------------] 100.00% 4.05 MiB p/s 15s
2025-03-31T20:28:39Z    INFO    [vulndb] Artifact successfully downloaded       repo="mirror.gcr.io/aquasec/trivy-db:2"
2025-03-31T20:28:39Z    INFO    [vuln] Vulnerability scanning is enabled
2025-03-31T20:28:39Z    INFO    [secret] Secret scanning is enabled
2025-03-31T20:28:39Z    INFO    [secret] If your scanning is slow, please try '--scanners vuln' to disable secret scanning
2025-03-31T20:28:39Z    INFO    [secret] Please see also https://trivy.dev/v0.61/docs/scanner/secret#recommendation for faster secret detection

Broadcast message from root@wawaa (Mon 2025-03-31 20:28:57 UTC):

Azure Patch Management initiated a reboot after a patch installation run.
The system is going down for reboot at Mon 2025-03-31 20:35:56 UTC!

2025-03-31T20:29:11Z    INFO    Detected OS     family="alpine" version="3.21.3"
2025-03-31T20:29:11Z    INFO    [alpine] Detecting vulnerabilities...   os_version="3.21" repository="3.21" pkg_num=46
2025-03-31T20:29:11Z    INFO    Number of language-specific files       num=1
2025-03-31T20:29:11Z    INFO    [gobinary] Detecting vulnerabilities...
2025-03-31T20:29:11Z    WARN    Using severities from other vendors for some vulnerabilities. Read https://trivy.dev/v0.61/docs/scanner/vulnerability#severity-selection for details.

Report Summary

┌────────────────────────────────────┬──────────┬─────────────────┬─────────┐
│               Target               │   Type   │ Vulnerabilities │ Secrets │
├────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ postgres:15-alpine (alpine 3.21.3) │  alpine  │        5        │    -    │
├────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/bin/gosu                 │ gobinary │       58        │    -    │
└────────────────────────────────────┴──────────┴─────────────────┴─────────┘
Legend:
- '-': Not scanned
- '0': Clean (no security findings detected)


postgres:15-alpine (alpine 3.21.3)

Total: 5 (UNKNOWN: 0, LOW: 0, MEDIUM: 0, HIGH: 5, CRITICAL: 0)

┌─────────┬────────────────┬──────────┬────────┬───────────────────┬───────────────┬─────────────────────────────────────────────────────────────┐
│ Library │ Vulnerability  │ Severity │ Status │ Installed Version │ Fixed Version │                            Title                            │
├─────────┼────────────────┼──────────┼────────┼───────────────────┼───────────────┼─────────────────────────────────────────────────────────────┤
│ libxml2 │ CVE-2024-56171 │ HIGH     │ fixed  │ 2.13.4-r3         │ 2.13.4-r4     │ libxml2: Use-After-Free in libxml2                          │
│         │                │          │        │                   │               │ https://avd.aquasec.com/nvd/cve-2024-56171                  │
│         ├────────────────┤          │        │                   │               ├─────────────────────────────────────────────────────────────┤
│         │ CVE-2025-24928 │          │        │                   │               │ libxml2: Stack-based buffer overflow in xmlSnprintfElements │
│         │                │          │        │                   │               │ of libxml2                                                  │
│         │                │          │        │                   │               │ https://avd.aquasec.com/nvd/cve-2025-24928                  │
│         ├────────────────┤          │        │                   ├───────────────┼─────────────────────────────────────────────────────────────┤
│         │ CVE-2025-27113 │          │        │                   │ 2.13.4-r5     │ libxml2: NULL Pointer Dereference in libxml2 xmlPatMatch    │
│         │                │          │        │                   │               │ https://avd.aquasec.com/nvd/cve-2025-27113                  │
├─────────┼────────────────┤          │        ├───────────────────┼───────────────┼─────────────────────────────────────────────────────────────┤
│ libxslt │ CVE-2024-55549 │          │        │ 1.1.42-r1         │ 1.1.42-r2     │ libxslt: Use-After-Free in libxslt (xsltGetInheritedNsList) │
│         │                │          │        │                   │               │ https://avd.aquasec.com/nvd/cve-2024-55549                  │
│         ├────────────────┤          │        │                   │               ├─────────────────────────────────────────────────────────────┤
│         │ CVE-2025-24855 │          │        │                   │               │ libxslt: Use-After-Free in libxslt numbers.c                │
│         │                │          │        │                   │               │ https://avd.aquasec.com/nvd/cve-2025-24855                  │
└─────────┴────────────────┴──────────┴────────┴───────────────────┴───────────────┴─────────────────────────────────────────────────────────────┘

usr/local/bin/gosu (gobinary)

Total: 58 (UNKNOWN: 0, LOW: 1, MEDIUM: 23, HIGH: 31, CRITICAL: 3)

┌─────────┬────────────────┬──────────┬────────┬───────────────────┬──────────────────────────────────┬──────────────────────────────────────────────────────────────┐
│ Library │ Vulnerability  │ Severity │ Status │ Installed Version │          Fixed Version           │                            Title                             │
├─────────┼────────────────┼──────────┼────────┼───────────────────┼──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ stdlib  │ CVE-2023-24538 │ CRITICAL │ fixed  │ v1.18.2           │ 1.19.8, 1.20.3                   │ golang: html/template: backticks not treated as string       │
│         │                │          │        │                   │                                  │ delimiters                                                   │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2023-24538                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2023-24540 │          │        │                   │ 1.19.9, 1.20.4                   │ golang: html/template: improper handling of JavaScript       │
│         │                │          │        │                   │                                  │ whitespace                                                   │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2023-24540                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2024-24790 │          │        │                   │ 1.21.11, 1.22.4                  │ golang: net/netip: Unexpected behavior from Is methods for   │
│         │                │          │        │                   │                                  │ IPv4-mapped IPv6 addresses                                   │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2024-24790                   │
│         ├────────────────┼──────────┤        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2022-27664 │ HIGH     │        │                   │ 1.18.6, 1.19.1                   │ golang: net/http: handle server errors after sending GOAWAY  │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2022-27664                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2022-28131 │          │        │                   │ 1.17.12, 1.18.4                  │ golang: encoding/xml: stack exhaustion in Decoder.Skip       │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2022-28131                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2022-2879  │          │        │                   │ 1.18.7, 1.19.2                   │ golang: archive/tar: github.com/vbatts/tar-split: unbounded  │
│         │                │          │        │                   │                                  │ memory consumption when reading headers                      │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2022-2879                    │
│         ├────────────────┤          │        │                   │                                  ├──────────────────────────────────────────────────────────────┤
│         │ CVE-2022-2880  │          │        │                   │                                  │ golang: net/http/httputil: ReverseProxy should not forward   │
│         │                │          │        │                   │                                  │ unparseable query parameters                                 │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2022-2880                    │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2022-29804 │          │        │                   │ 1.17.11, 1.18.3                  │ ELSA-2022-17957: ol8addon security update (IMPORTANT)        │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2022-29804                   │
│         ├────────────────┤          │        │                   │                                  ├──────────────────────────────────────────────────────────────┤
│         │ CVE-2022-30580 │          │        │                   │                                  │ golang: os/exec: Code injection in Cmd.Start                 │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2022-30580                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2022-30630 │          │        │                   │ 1.17.12, 1.18.4                  │ golang: io/fs: stack exhaustion in Glob                      │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2022-30630                   │
│         ├────────────────┤          │        │                   │                                  ├──────────────────────────────────────────────────────────────┤
│         │ CVE-2022-30631 │          │        │                   │                                  │ golang: compress/gzip: stack exhaustion in Reader.Read       │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2022-30631                   │
│         ├────────────────┤          │        │                   │                                  ├──────────────────────────────────────────────────────────────┤
│         │ CVE-2022-30632 │          │        │                   │                                  │ golang: path/filepath: stack exhaustion in Glob              │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2022-30632                   │
│         ├────────────────┤          │        │                   │                                  ├──────────────────────────────────────────────────────────────┤
│         │ CVE-2022-30633 │          │        │                   │                                  │ golang: encoding/xml: stack exhaustion in Unmarshal          │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2022-30633                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2022-30634 │          │        │                   │ 1.17.11, 1.18.3                  │ ELSA-2022-17957: ol8addon security update (IMPORTANT)        │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2022-30634                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2022-30635 │          │        │                   │ 1.17.12, 1.18.4                  │ golang: encoding/gob: stack exhaustion in Decoder.Decode     │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2022-30635                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2022-32189 │          │        │                   │ 1.17.13, 1.18.5                  │ golang: math/big: decoding big.Float and big.Rat types can   │
│         │                │          │        │                   │                                  │ panic if the encoded...                                      │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2022-32189                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2022-41715 │          │        │                   │ 1.18.7, 1.19.2                   │ golang: regexp/syntax: limit memory used by parsing regexps  │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2022-41715                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2022-41716 │          │        │                   │ 1.18.8, 1.19.3                   │ Due to unsanitized NUL values, attackers may be able to      │
│         │                │          │        │                   │                                  │ maliciously se...                                            │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2022-41716                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2022-41720 │          │        │                   │ 1.18.9, 1.19.4                   │ golang: os, net/http: avoid escapes from os.DirFS and        │
│         │                │          │        │                   │                                  │ http.Dir on Windows                                          │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2022-41720                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2022-41722 │          │        │                   │ 1.19.6, 1.20.1                   │ golang: path/filepath: path-filepath filepath.Clean path     │
│         │                │          │        │                   │                                  │ traversal                                                    │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2022-41722                   │
│         ├────────────────┤          │        │                   │                                  ├──────────────────────────────────────────────────────────────┤
│         │ CVE-2022-41723 │          │        │                   │                                  │ golang.org/x/net/http2: avoid quadratic complexity in HPACK  │
│         │                │          │        │                   │                                  │ decoding                                                     │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2022-41723                   │
│         ├────────────────┤          │        │                   │                                  ├──────────────────────────────────────────────────────────────┤
│         │ CVE-2022-41724 │          │        │                   │                                  │ golang: crypto/tls: large handshake records may cause panics │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2022-41724                   │
│         ├────────────────┤          │        │                   │                                  ├──────────────────────────────────────────────────────────────┤
│         │ CVE-2022-41725 │          │        │                   │                                  │ golang: net/http, mime/multipart: denial of service from     │
│         │                │          │        │                   │                                  │ excessive resource consumption                               │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2022-41725                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2023-24534 │          │        │                   │ 1.19.8, 1.20.3                   │ golang: net/http, net/textproto: denial of service from      │
│         │                │          │        │                   │                                  │ excessive memory allocation                                  │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2023-24534                   │
│         ├────────────────┤          │        │                   │                                  ├──────────────────────────────────────────────────────────────┤
│         │ CVE-2023-24536 │          │        │                   │                                  │ golang: net/http, net/textproto, mime/multipart: denial of   │
│         │                │          │        │                   │                                  │ service from excessive resource consumption                  │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2023-24536                   │
│         ├────────────────┤          │        │                   │                                  ├──────────────────────────────────────────────────────────────┤
│         │ CVE-2023-24537 │          │        │                   │                                  │ golang: go/parser: Infinite loop in parsing                  │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2023-24537                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2023-24539 │          │        │                   │ 1.19.9, 1.20.4                   │ golang: html/template: improper sanitization of CSS values   │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2023-24539                   │
│         ├────────────────┤          │        │                   │                                  ├──────────────────────────────────────────────────────────────┤
│         │ CVE-2023-29400 │          │        │                   │                                  │ golang: html/template: improper handling of empty HTML       │
│         │                │          │        │                   │                                  │ attributes                                                   │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2023-29400                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2023-29403 │          │        │                   │ 1.19.10, 1.20.5                  │ golang: runtime: unexpected behavior of setuid/setgid        │
│         │                │          │        │                   │                                  │ binaries                                                     │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2023-29403                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2023-39325 │          │        │                   │ 1.20.10, 1.21.3                  │ golang: net/http, x/net/http2: rapid stream resets can cause │
│         │                │          │        │                   │                                  │ excessive work (CVE-2023-44487)                              │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2023-39325                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2023-45283 │          │        │                   │ 1.20.11, 1.21.4, 1.20.12, 1.21.5 │ The filepath package does not recognize paths with a \??\    │
│         │                │          │        │                   │                                  │ prefix as...                                                 │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2023-45283                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2023-45287 │          │        │                   │ 1.20.0                           │ golang: crypto/tls: Timing Side Channel attack in RSA based  │
│         │                │          │        │                   │                                  │ TLS key exchanges....                                        │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2023-45287                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2023-45288 │          │        │                   │ 1.21.9, 1.22.2                   │ golang: net/http, x/net/http2: unlimited number of           │
│         │                │          │        │                   │                                  │ CONTINUATION frames causes DoS                               │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2023-45288                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2024-34156 │          │        │                   │ 1.22.7, 1.23.1                   │ encoding/gob: golang: Calling Decoder.Decode on a message    │
│         │                │          │        │                   │                                  │ which contains deeply nested structures...                   │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2024-34156                   │
│         ├────────────────┼──────────┤        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2022-1705  │ MEDIUM   │        │                   │ 1.17.12, 1.18.4                  │ golang: net/http: improper sanitization of Transfer-Encoding │
│         │                │          │        │                   │                                  │ header                                                       │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2022-1705                    │
│         ├────────────────┤          │        │                   │                                  ├──────────────────────────────────────────────────────────────┤
│         │ CVE-2022-1962  │          │        │                   │                                  │ golang: go/parser: stack exhaustion in all Parse* functions  │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2022-1962                    │
│         ├────────────────┤          │        │                   │                                  ├──────────────────────────────────────────────────────────────┤
│         │ CVE-2022-32148 │          │        │                   │                                  │ golang: net/http/httputil: NewSingleHostReverseProxy - omit  │
│         │                │          │        │                   │                                  │ X-Forwarded-For not working                                  │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2022-32148                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2022-41717 │          │        │                   │ 1.18.9, 1.19.4                   │ golang: net/http: excessive memory growth in a Go server     │
│         │                │          │        │                   │                                  │ accepting HTTP/2 requests...                                 │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2022-41717                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2023-24532 │          │        │                   │ 1.19.7, 1.20.2                   │ golang: crypto/internal/nistec: specific unreduced P-256     │
│         │                │          │        │                   │                                  │ scalars produce incorrect results                            │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2023-24532                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2023-29406 │          │        │                   │ 1.19.11, 1.20.6                  │ golang: net/http: insufficient sanitization of Host header   │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2023-29406                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2023-29409 │          │        │                   │ 1.19.12, 1.20.7, 1.21.0-rc.4     │ golang: crypto/tls: slow verification of certificate chains  │
│         │                │          │        │                   │                                  │ containing large RSA keys                                    │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2023-29409                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2023-39318 │          │        │                   │ 1.20.8, 1.21.1                   │ golang: html/template: improper handling of HTML-like        │
│         │                │          │        │                   │                                  │ comments within script contexts                              │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2023-39318                   │
│         ├────────────────┤          │        │                   │                                  ├──────────────────────────────────────────────────────────────┤
│         │ CVE-2023-39319 │          │        │                   │                                  │ golang: html/template: improper handling of special tags     │
│         │                │          │        │                   │                                  │ within script contexts                                       │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2023-39319                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2023-39326 │          │        │                   │ 1.20.12, 1.21.5                  │ golang: net/http/internal: Denial of Service (DoS) via       │
│         │                │          │        │                   │                                  │ Resource Consumption via HTTP requests...                    │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2023-39326                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2023-45284 │          │        │                   │ 1.20.11, 1.21.4                  │ On Windows, The IsLocal function does not correctly detect   │
│         │                │          │        │                   │                                  │ reserved de ......                                           │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2023-45284                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2023-45289 │          │        │                   │ 1.21.8, 1.22.1                   │ golang: net/http/cookiejar: incorrect forwarding of          │
│         │                │          │        │                   │                                  │ sensitive headers and cookies on HTTP redirect...            │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2023-45289                   │
│         ├────────────────┤          │        │                   │                                  ├──────────────────────────────────────────────────────────────┤
│         │ CVE-2023-45290 │          │        │                   │                                  │ golang: net/http: golang: mime/multipart: golang:            │
│         │                │          │        │                   │                                  │ net/textproto: memory exhaustion in                          │
│         │                │          │        │                   │                                  │ Request.ParseMultipartForm                                   │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2023-45290                   │
│         ├────────────────┤          │        │                   │                                  ├──────────────────────────────────────────────────────────────┤
│         │ CVE-2024-24783 │          │        │                   │                                  │ golang: crypto/x509: Verify panics on certificates with an   │
│         │                │          │        │                   │                                  │ unknown public key algorithm...                              │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2024-24783                   │
│         ├────────────────┤          │        │                   │                                  ├──────────────────────────────────────────────────────────────┤
│         │ CVE-2024-24784 │          │        │                   │                                  │ golang: net/mail: comments in display names are incorrectly  │
│         │                │          │        │                   │                                  │ handled                                                      │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2024-24784                   │
│         ├────────────────┤          │        │                   │                                  ├──────────────────────────────────────────────────────────────┤
│         │ CVE-2024-24785 │          │        │                   │                                  │ golang: html/template: errors returned from MarshalJSON      │
│         │                │          │        │                   │                                  │ methods may break template escaping                          │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2024-24785                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2024-24789 │          │        │                   │ 1.21.11, 1.22.4                  │ golang: archive/zip: Incorrect handling of certain ZIP files │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2024-24789                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2024-24791 │          │        │                   │ 1.21.12, 1.22.5                  │ net/http: Denial of service due to improper 100-continue     │
│         │                │          │        │                   │                                  │ handling in net/http                                         │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2024-24791                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2024-34155 │          │        │                   │ 1.22.7, 1.23.1                   │ go/parser: golang: Calling any of the Parse functions        │
│         │                │          │        │                   │                                  │ containing deeply nested literals...                         │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2024-34155                   │
│         ├────────────────┤          │        │                   │                                  ├──────────────────────────────────────────────────────────────┤
│         │ CVE-2024-34158 │          │        │                   │                                  │ go/build/constraint: golang: Calling Parse on a "// +build"  │
│         │                │          │        │                   │                                  │ build tag line with...                                       │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2024-34158                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2024-45336 │          │        │                   │ 1.22.11, 1.23.5, 1.24.0-rc.2     │ golang: net/http: net/http: sensitive headers incorrectly    │
│         │                │          │        │                   │                                  │ sent after cross-domain redirect                             │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2024-45336                   │
│         ├────────────────┤          │        │                   │                                  ├──────────────────────────────────────────────────────────────┤
│         │ CVE-2024-45341 │          │        │                   │                                  │ golang: crypto/x509: crypto/x509: usage of IPv6 zone IDs can │
│         │                │          │        │                   │                                  │ bypass URI name...                                           │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2024-45341                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2025-22866 │          │        │                   │ 1.22.12, 1.23.6, 1.24.0-rc.3     │ crypto/internal/nistec: golang: Timing sidechannel for P-256 │
│         │                │          │        │                   │                                  │ on ppc64le in crypto/internal/nistec                         │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2025-22866                   │
│         ├────────────────┼──────────┤        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2022-30629 │ LOW      │        │                   │ 1.17.11, 1.18.3                  │ golang: crypto/tls: session tickets lack random              │
│         │                │          │        │                   │                                  │ ticket_age_add                                               │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2022-30629                   │
└─────────┴────────────────┴──────────┴────────┴───────────────────┴──────────────────────────────────┴──────────────────────────────────────────────────────────────┘
```

[Sommaire des scans](#2-scan-de-vuln)

# Scan wiki js

```
azureuser@wawaa:~/travail/docker-compose$ trivy image e5549c107513
2025-03-31T20:31:47Z    INFO    [vuln] Vulnerability scanning is enabled
2025-03-31T20:31:47Z    INFO    [secret] Secret scanning is enabled
2025-03-31T20:31:47Z    INFO    [secret] If your scanning is slow, please try '--scanners vuln' to disable secret scanning
2025-03-31T20:31:47Z    INFO    [secret] Please see also https://trivy.dev/v0.61/docs/scanner/secret#recommendation for faster secret detection

Broadcast message from root@wawaa (Mon 2025-03-31 20:31:57 UTC):

Azure Patch Management initiated a reboot after a patch installation run.
The system is going down for reboot at Mon 2025-03-31 20:35:56 UTC!


Broadcast message from root@wawaa (Mon 2025-03-31 20:32:57 UTC):

Azure Patch Management initiated a reboot after a patch installation run.
The system is going down for reboot at Mon 2025-03-31 20:35:56 UTC!

2025-03-31T20:33:21Z    INFO    Detected OS     family="alpine" version="3.21.3"
2025-03-31T20:33:21Z    INFO    [alpine] Detecting vulnerabilities...   os_version="3.21" repository="3.21" pkg_num=71
2025-03-31T20:33:21Z    INFO    Number of language-specific files       num=1
2025-03-31T20:33:21Z    INFO    [node-pkg] Detecting vulnerabilities...
2025-03-31T20:33:22Z    INFO    Table result includes only package filenames. Use '--format json' option to get the full path to the package file.

Report Summary

┌──────────────────────────────────────────────────────────────────────────────────┬──────────┬─────────────────┬─────────┐
│                                      Target                                      │   Type   │ Vulnerabilities │ Secrets │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ e5549c107513 (alpine 3.21.3)                                                     │  alpine  │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ opt/yarn-v1.22.22/package.json                                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/corepack/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@isaacs/cliui/node_modules/ansi-reg- │ node-pkg │        0        │    -    │
│ ex/package.json                                                                  │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@isaacs/cliui/node_modules/emoji-re- │ node-pkg │        0        │    -    │
│ gex/package.json                                                                 │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@isaacs/cliui/node_modules/string-w- │ node-pkg │        0        │    -    │
│ idth/package.json                                                                │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@isaacs/cliui/node_modules/strip-an- │ node-pkg │        0        │    -    │
│ si/package.json                                                                  │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@isaacs/cliui/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@isaacs/string-locale-compare/packa- │ node-pkg │        0        │    -    │
│ ge.json                                                                          │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/agent/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/arborist/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/config/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/fs/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/git/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/installed-package-contents/- │ node-pkg │        0        │    -    │
│ package.json                                                                     │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/map-workspaces/package.json  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/metavuln-calculator/package- │ node-pkg │        0        │    -    │
│ .json                                                                            │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/name-from-folder/package.js- │ node-pkg │        0        │    -    │
│ on                                                                               │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/node-gyp/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/package-json/package.json    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/promise-spawn/package.json   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/query/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/redact/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/run-script/package.json      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@pkgjs/parseargs/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@sigstore/bundle/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@sigstore/core/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@sigstore/protobuf-specs/package.js- │ node-pkg │        0        │    -    │
│ on                                                                               │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@sigstore/sign/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@sigstore/tuf/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@sigstore/verify/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@tufjs/canonical-json/package.json   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@tufjs/models/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/abbrev/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/agent-base/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/aggregate-error/package.json         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/ansi-regex/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/ansi-styles/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/aproba/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/archy/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/balanced-match/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/bin-links/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/binary-extensions/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/brace-expansion/package.json         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/cacache/package.json                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/chalk/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/chownr/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/ci-info/package.json                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/cidr-regex/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/clean-stack/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/cli-columns/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/cmd-shim/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/color-convert/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/color-name/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/common-ancestor-path/package.json    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/cross-spawn/node_modules/which/pack- │ node-pkg │        0        │    -    │
│ age.json                                                                         │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/cross-spawn/package.json             │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/cssesc/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/debug/node_modules/ms/package.json   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/debug/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/diff/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/eastasianwidth/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/emoji-regex/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/encoding/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/env-paths/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/err-code/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/exponential-backoff/package.json     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/fastest-levenshtein/package.json     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/foreground-child/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/fs-minipass/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/glob/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/graceful-fs/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/hosted-git-info/package.json         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/http-cache-semantics/package.json    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/http-proxy-agent/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/https-proxy-agent/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/iconv-lite/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/ignore-walk/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/imurmurhash/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/indent-string/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/ini/package.json                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/init-package-json/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/ip-address/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/ip-regex/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/is-cidr/package.json                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/is-fullwidth-code-point/package.json │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/is-lambda/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/isexe/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/jackspeak/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/jsbn/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/json-parse-even-better-errors/packa- │ node-pkg │        0        │    -    │
│ ge.json                                                                          │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/json-stringify-nice/package.json     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/jsonparse/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/just-diff-apply/package.json         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/just-diff/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/libnpmaccess/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/libnpmdiff/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/libnpmexec/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/libnpmfund/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/libnpmhook/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/libnpmorg/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/libnpmpack/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/libnpmpublish/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/libnpmsearch/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/libnpmteam/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/libnpmversion/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/lru-cache/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/make-fetch-happen/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/minimatch/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/minipass-collect/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/minipass-fetch/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/minipass-flush/node_modules/minipas- │ node-pkg │        0        │    -    │
│ s/package.json                                                                   │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/minipass-flush/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/minipass-pipeline/node_modules/mini- │ node-pkg │        0        │    -    │
│ pass/package.json                                                                │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/minipass-pipeline/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/minipass-sized/node_modules/minipas- │ node-pkg │        0        │    -    │
│ s/package.json                                                                   │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/minipass-sized/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/minipass/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/minizlib/node_modules/minipass/pack- │ node-pkg │        0        │    -    │
│ age.json                                                                         │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/minizlib/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/mkdirp/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/ms/package.json                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/mute-stream/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/negotiator/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/node-gyp/node_modules/proc-log/pack- │ node-pkg │        0        │    -    │
│ age.json                                                                         │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/node-gyp/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/nopt/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/normalize-package-data/package.json  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/npm-audit-report/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/npm-bundled/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/npm-install-checks/package.json      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/npm-normalize-package-bin/package.j- │ node-pkg │        0        │    -    │
│ son                                                                              │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/npm-package-arg/package.json         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/npm-packlist/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/npm-pick-manifest/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/npm-profile/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/npm-registry-fetch/package.json      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/npm-user-validate/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/p-map/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/package-json-from-dist/package.json  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/pacote/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/parse-conflict-json/package.json     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/path-key/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/path-scurry/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/postcss-selector-parser/package.json │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/proc-log/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/proggy/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/promise-all-reject-late/package.json │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/promise-call-limit/package.json      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/promise-inflight/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/promise-retry/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/promzard/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/qrcode-terminal/package.json         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/read-cmd-shim/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/read-package-json-fast/package.json  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/read/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/retry/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/safer-buffer/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/semver/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/shebang-command/package.json         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/shebang-regex/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/signal-exit/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/sigstore/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/smart-buffer/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/socks-proxy-agent/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/socks/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/spdx-correct/node_modules/spdx-expr- │ node-pkg │        0        │    -    │
│ ession-parse/package.json                                                        │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/spdx-correct/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/spdx-exceptions/package.json         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/spdx-expression-parse/package.json   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/spdx-license-ids/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/sprintf-js/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/ssri/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/string-width-cjs/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/string-width/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/strip-ansi-cjs/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/strip-ansi/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/supports-color/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/tar/node_modules/fs-minipass/node_m- │ node-pkg │        0        │    -    │
│ odules/minipass/package.json                                                     │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/tar/node_modules/fs-minipass/packag- │ node-pkg │        0        │    -    │
│ e.json                                                                           │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/tar/node_modules/minipass/package.j- │ node-pkg │        0        │    -    │
│ son                                                                              │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/tar/package.json                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/text-table/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/tiny-relative-date/package.json      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/treeverse/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/tuf-js/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/unique-filename/package.json         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/unique-slug/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/util-deprecate/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/validate-npm-package-license/node_m- │ node-pkg │        0        │    -    │
│ odules/spdx-expression-parse/package.json                                        │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/validate-npm-package-license/packag- │ node-pkg │        0        │    -    │
│ e.json                                                                           │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/validate-npm-package-name/package.j- │ node-pkg │        0        │    -    │
│ son                                                                              │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/walk-up-path/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/which/node_modules/isexe/package.js- │ node-pkg │        0        │    -    │
│ on                                                                               │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/which/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/wrap-ansi-cjs/node_modules/ansi-sty- │ node-pkg │        0        │    -    │
│ les/package.json                                                                 │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/wrap-ansi-cjs/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/wrap-ansi/node_modules/ansi-regex/p- │ node-pkg │        0        │    -    │
│ ackage.json                                                                      │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/wrap-ansi/node_modules/emoji-regex/- │ node-pkg │        0        │    -    │
│ package.json                                                                     │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/wrap-ansi/node_modules/string-width- │ node-pkg │        0        │    -    │
│ /package.json                                                                    │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/wrap-ansi/node_modules/strip-ansi/p- │ node-pkg │        0        │    -    │
│ ackage.json                                                                      │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/wrap-ansi/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/write-file-atomic/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/yallist/package.json                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@algolia/cache-browser-local-storage/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@algolia/cache-common/package.json                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@algolia/cache-in-memory/package.json                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@algolia/client-account/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@algolia/client-analytics/package.json                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@algolia/client-common/package.json                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@algolia/client-recommendation/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@algolia/client-search/package.json                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@algolia/logger-common/package.json                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@algolia/logger-console/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@algolia/requester-browser-xhr/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@algolia/requester-common/package.json                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@algolia/requester-node-http/package.json                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@algolia/transporter/package.json                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@ampproject/remapping/package.json                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@apollo/client/node_modules/@wry/context/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@apollo/client/node_modules/@wry/equality/package.json         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@apollo/client/node_modules/graphql-tag/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@apollo/client/node_modules/optimism/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@apollo/client/node_modules/symbol-observable/package.json     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@apollo/client/node_modules/ts-invariant/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@apollo/client/node_modules/zen-observable-ts/package.json     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@apollo/client/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@apollo/protobufjs/node_modules/@types/node/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@apollo/protobufjs/node_modules/long/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@apollo/protobufjs/package.json                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@apollographql/apollo-tools/package.json                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@apollographql/graphql-playground-html/node_modules/xss/packa- │ node-pkg │        0        │    -    │
│ ge.json                                                                          │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@apollographql/graphql-playground-html/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@apollographql/graphql-upload-8-fork/node_modules/@types/body- │ node-pkg │        0        │    -    │
│ -parser/package.json                                                             │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@apollographql/graphql-upload-8-fork/node_modules/@types/expr- │ node-pkg │        0        │    -    │
│ ess-serve-static-core/package.json                                               │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@apollographql/graphql-upload-8-fork/node_modules/@types/expr- │ node-pkg │        0        │    -    │
│ ess/package.json                                                                 │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@apollographql/graphql-upload-8-fork/node_modules/busboy/pack- │ node-pkg │        0        │    -    │
│ age.json                                                                         │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@apollographql/graphql-upload-8-fork/node_modules/depd/packag- │ node-pkg │        0        │    -    │
│ e.json                                                                           │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@apollographql/graphql-upload-8-fork/node_modules/dicer/packa- │ node-pkg │        1        │    -    │
│ ge.json                                                                          │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@apollographql/graphql-upload-8-fork/node_modules/http-errors- │ node-pkg │        0        │    -    │
│ /package.json                                                                    │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@apollographql/graphql-upload-8-fork/node_modules/statuses/pa- │ node-pkg │        0        │    -    │
│ ckage.json                                                                       │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@apollographql/graphql-upload-8-fork/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@ardatan/aggregate-error/node_modules/tslib/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@ardatan/aggregate-error/package.json                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@ardatan/relay-compiler/node_modules/chalk/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@ardatan/relay-compiler/node_modules/cliui/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@ardatan/relay-compiler/node_modules/yargs-parser/package.json │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@ardatan/relay-compiler/node_modules/yargs/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@ardatan/relay-compiler/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@asciidoctor/cli/node_modules/cliui/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@asciidoctor/cli/node_modules/wrap-ansi/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@asciidoctor/cli/node_modules/y18n/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@asciidoctor/cli/node_modules/yargs-parser/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@asciidoctor/cli/node_modules/yargs/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@asciidoctor/cli/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@asciidoctor/core/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@azure/abort-controller/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@azure/core-auth/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@azure/core-http/node_modules/@azure/abort-controller/package- │ node-pkg │        0        │    -    │
│ .json                                                                            │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@azure/core-http/node_modules/punycode/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@azure/core-http/node_modules/tough-cookie/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@azure/core-http/node_modules/universalify/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@azure/core-http/node_modules/uuid/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@azure/core-http/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@azure/core-lro/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@azure/core-paging/package.json                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@azure/core-tracing/package.json                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@azure/core-util/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@azure/logger/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@azure/ms-rest-azure-env/package.json                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@azure/ms-rest-js/node_modules/form-data/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@azure/ms-rest-js/node_modules/tslib/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@azure/ms-rest-js/node_modules/uuid/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@azure/ms-rest-js/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@azure/ms-rest-nodeauth/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@azure/storage-blob/node_modules/@azure/abort-controller/pack- │ node-pkg │        0        │    -    │
│ age.json                                                                         │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@azure/storage-blob/package.json                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/code-frame/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/compat-data/package.json                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/core/node_modules/convert-source-map/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/core/node_modules/json5/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/core/node_modules/semver/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/core/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/generator/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/helper-annotate-as-pure/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/helper-compilation-targets/node_modules/lru-cache/pack- │ node-pkg │        0        │    -    │
│ age.json                                                                         │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/helper-compilation-targets/node_modules/semver/package- │ node-pkg │        0        │    -    │
│ .json                                                                            │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/helper-compilation-targets/node_modules/yallist/packag- │ node-pkg │        0        │    -    │
│ e.json                                                                           │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/helper-compilation-targets/package.json                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/helper-create-class-features-plugin/node_modules/semve- │ node-pkg │        0        │    -    │
│ r/package.json                                                                   │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/helper-create-class-features-plugin/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/helper-function-name/package.json                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/helper-member-expression-to-functions/package.json      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/helper-module-imports/package.json                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/helper-module-transforms/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/helper-optimise-call-expression/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/helper-plugin-utils/package.json                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/helper-replace-supers/package.json                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/helper-skip-transparent-expression-wrappers/package.js- │ node-pkg │        0        │    -    │
│ on                                                                               │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/helper-split-export-declaration/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/helper-string-parser/package.json                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/helper-validator-identifier/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/helper-validator-option/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/helpers/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/parser/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/plugin-proposal-class-properties/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/plugin-proposal-object-rest-spread/package.json         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/plugin-syntax-class-properties/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/plugin-syntax-flow/package.json                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/plugin-syntax-jsx/package.json                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/plugin-syntax-object-rest-spread/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/plugin-transform-arrow-functions/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/plugin-transform-block-scoped-functions/package.json    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/plugin-transform-block-scoping/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/plugin-transform-classes/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/plugin-transform-computed-properties/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/plugin-transform-destructuring/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/plugin-transform-flow-strip-types/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/plugin-transform-for-of/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/plugin-transform-function-name/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/plugin-transform-literals/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/plugin-transform-member-expression-literals/package.js- │ node-pkg │        0        │    -    │
│ on                                                                               │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/plugin-transform-modules-commonjs/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/plugin-transform-object-super/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/plugin-transform-parameters/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/plugin-transform-property-literals/package.json         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/plugin-transform-react-display-name/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/plugin-transform-react-jsx/package.json                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/plugin-transform-shorthand-properties/package.json      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/plugin-transform-spread/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/plugin-transform-template-literals/package.json         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/runtime/node_modules/regenerator-runtime/package.json   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/runtime/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/template/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/traverse/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@babel/types/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@colors/colors/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@dabh/diagnostics/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@elastic/transport/node_modules/hpagent/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@elastic/transport/node_modules/secure-json-parse/benchmarks/- │ node-pkg │        0        │    -    │
│ package.json                                                                     │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@elastic/transport/node_modules/secure-json-parse/package.json │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@elastic/transport/package.json                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@exlinc/keycloak-passport/node_modules/oauth/package.json      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@exlinc/keycloak-passport/node_modules/passport-oauth2/packag- │ node-pkg │        0        │    -    │
│ e.json                                                                           │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@exlinc/keycloak-passport/package.json                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@gar/promisify/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/batch-delegate/node_modules/tslib/package.json  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/batch-delegate/package.json                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/batch-execute/node_modules/tslib/package.json   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/batch-execute/package.json                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/code-file-loader/node_modules/tslib/package.js- │ node-pkg │        0        │    -    │
│ on                                                                               │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/code-file-loader/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/delegate/node_modules/tslib/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/delegate/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/git-loader/node_modules/tslib/package.json      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/git-loader/package.json                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/github-loader/node_modules/tslib/package.json   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/github-loader/package.json                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/graphql-file-loader/node_modules/tslib/package- │ node-pkg │        0        │    -    │
│ .json                                                                            │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/graphql-file-loader/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/graphql-tag-pluck/node_modules/@babel/parser/p- │ node-pkg │        0        │    -    │
│ ackage.json                                                                      │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/graphql-tag-pluck/node_modules/@babel/traverse- │ node-pkg │        0        │    -    │
│ /node_modules/@babel/parser/package.json                                         │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/graphql-tag-pluck/node_modules/@babel/traverse- │ node-pkg │        0        │    -    │
│ /node_modules/@babel/types/package.json                                          │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/graphql-tag-pluck/node_modules/@babel/traverse- │ node-pkg │        1        │    -    │
│ /package.json                                                                    │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/graphql-tag-pluck/node_modules/@babel/types/pa- │ node-pkg │        0        │    -    │
│ ckage.json                                                                       │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/graphql-tag-pluck/node_modules/tslib/package.j- │ node-pkg │        0        │    -    │
│ son                                                                              │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/graphql-tag-pluck/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/import/node_modules/@graphql-tools/utils/packa- │ node-pkg │        0        │    -    │
│ ge.json                                                                          │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/import/package.json                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/json-file-loader/node_modules/tslib/package.js- │ node-pkg │        0        │    -    │
│ on                                                                               │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/json-file-loader/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/links/node_modules/form-data/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/links/node_modules/is-promise/package.json      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/links/node_modules/tslib/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/links/package.json                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/load-files/package.json                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/load/node_modules/globby/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/load/node_modules/is-glob/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/load/node_modules/p-limit/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/load/node_modules/tslib/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/load/package.json                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/merge/node_modules/@graphql-tools/merge/node_m- │ node-pkg │        0        │    -    │
│ odules/@graphql-tools/utils/package.json                                         │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/merge/node_modules/@graphql-tools/merge/node_m- │ node-pkg │        0        │    -    │
│ odules/tslib/package.json                                                        │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/merge/node_modules/@graphql-tools/merge/packag- │ node-pkg │        0        │    -    │
│ e.json                                                                           │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/merge/node_modules/@graphql-tools/schema/node_- │ node-pkg │        0        │    -    │
│ modules/@graphql-tools/utils/package.json                                        │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/merge/node_modules/@graphql-tools/schema/node_- │ node-pkg │        0        │    -    │
│ modules/tslib/package.json                                                       │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/merge/node_modules/@graphql-tools/schema/packa- │ node-pkg │        0        │    -    │
│ ge.json                                                                          │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/merge/node_modules/@graphql-tools/utils/packag- │ node-pkg │        0        │    -    │
│ e.json                                                                           │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/merge/node_modules/tslib/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/merge/node_modules/value-or-promise/package.js- │ node-pkg │        0        │    -    │
│ on                                                                               │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/merge/package.json                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/mock/node_modules/tslib/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/mock/package.json                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/module-loader/node_modules/tslib/package.json   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/module-loader/package.json                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/relay-operation-optimizer/node_modules/@graphq- │ node-pkg │        0        │    -    │
│ l-tools/utils/package.json                                                       │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/relay-operation-optimizer/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/resolvers-composition/node_modules/@graphql-to- │ node-pkg │        0        │    -    │
│ ols/utils/package.json                                                           │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/resolvers-composition/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/schema/node_modules/tslib/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/schema/package.json                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/stitch/node_modules/tslib/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/stitch/package.json                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/url-loader/node_modules/cross-fetch/package.js- │ node-pkg │        1        │    -    │
│ on                                                                               │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/url-loader/node_modules/cross-fetch/polyfill/p- │ node-pkg │        0        │    -    │
│ ackage.json                                                                      │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/url-loader/node_modules/form-data/package.json  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/url-loader/node_modules/is-promise/package.json │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/url-loader/node_modules/node-fetch/package.json │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/url-loader/node_modules/subscriptions-transpor- │ node-pkg │        0        │    -    │
│ t-ws/node_modules/ws/package.json                                                │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/url-loader/node_modules/subscriptions-transpor- │ node-pkg │        0        │    -    │
│ t-ws/package.json                                                                │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/url-loader/node_modules/tslib/package.json      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/url-loader/node_modules/ws/package.json         │ node-pkg │        2        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/url-loader/package.json                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/utils/node_modules/tslib/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/utils/package.json                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/wrap/node_modules/tslib/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-tools/wrap/package.json                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@graphql-typed-document-node/core/package.json                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@heroku/socksv5/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@joplin/turndown-plugin-gfm/package.json                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@josephg/resolvable/package.json                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@jridgewell/gen-mapping/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@jridgewell/resolve-uri/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@jridgewell/set-array/package.json                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@jridgewell/sourcemap-codec/package.json                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@jridgewell/trace-mapping/package.json                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@kwsites/file-exists/package.json                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@kwsites/promise-deferred/package.json                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@log4js-node/log4js-api/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@mapbox/node-pre-gyp/lib/util/nw-pre-gyp/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@mapbox/node-pre-gyp/node_modules/rimraf/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@mapbox/node-pre-gyp/node_modules/semver/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@mapbox/node-pre-gyp/package.json                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@microsoft/fetch-event-source/package.json                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@nodelib/fs.scandir/package.json                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@nodelib/fs.stat/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@nodelib/fs.walk/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@npmcli/fs/node_modules/semver/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@npmcli/fs/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@npmcli/move-file/node_modules/mkdirp/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@npmcli/move-file/node_modules/rimraf/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@npmcli/move-file/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@opentelemetry/api/package.json                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@protobufjs/aspromise/package.json                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@protobufjs/base64/package.json                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@protobufjs/codegen/package.json                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@protobufjs/eventemitter/package.json                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@protobufjs/fetch/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@protobufjs/float/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@protobufjs/inquire/package.json                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@protobufjs/path/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@protobufjs/pool/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@protobufjs/utf8/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@root/acme/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@root/asn1/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@root/csr/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@root/encoding/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@root/keypairs/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@root/pem/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@root/request/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@root/x509/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@swc/helpers/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@tokenizer/token/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@tootallnate/once/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/accepts/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/body-parser/package.json                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/command-line-args/package.json                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/command-line-usage/package.json                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/connect/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/content-disposition/package.json                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/cookiejar/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/cookies/node_modules/@types/body-parser/package.json    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/cookies/node_modules/@types/express-serve-static-core/- │ node-pkg │        0        │    -    │
│ package.json                                                                     │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/cookies/node_modules/@types/express/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/cookies/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/cors/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/express-serve-static-core/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/express/node_modules/@types/body-parser/package.json    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/express/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/fs-capacitor/package.json                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/geojson/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/http-assert/package.json                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/http-errors/package.json                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/keygrip/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/koa-compose/package.json                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/koa/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/ldapjs/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/long/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/mime/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/node-fetch/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/node/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/pg/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/qs/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/range-parser/package.json                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/readable-stream/node_modules/safe-buffer/package.json   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/readable-stream/package.json                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/root__asn1/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/send/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/serve-static/package.json                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/superagent/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/triple-beam/package.json                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/tunnel/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/websocket/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@types/ws/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@wry/caches/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@wry/equality/node_modules/tslib/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@wry/equality/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@wry/trie/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@xmldom/xmldom/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/@yarnpkg/lockfile/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/abab/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/abbrev/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/abort-controller/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/abstract-logging/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/accepts/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/acme/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/acorn-globals/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/acorn-walk/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/acorn/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/adal-node/node_modules/@xmldom/xmldom/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/adal-node/node_modules/async/package.json                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/adal-node/node_modules/axios/package.json                      │ node-pkg │        2        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/adal-node/node_modules/uuid/package.json                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/adal-node/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/agent-base/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/agentkeepalive/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/aggregate-error/node_modules/indent-string/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/aggregate-error/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ajv/package.json                                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/akismet-api/node_modules/formidable/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/akismet-api/node_modules/mime/package.json                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/akismet-api/node_modules/readable-stream/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/akismet-api/node_modules/semver/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/akismet-api/node_modules/superagent/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/akismet-api/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/algoliasearch/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ansi-styles/node_modules/color-convert/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ansi-styles/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/any-promise/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/anymatch/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apache-arrow/node_modules/@types/node/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apache-arrow/node_modules/undici-types/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apache-arrow/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apg-conv-api/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apg-lib/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-cache-control/package.json                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-datasource/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-fetch/node_modules/cross-fetch/package.json             │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-fetch/node_modules/cross-fetch/polyfill/package.json    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-fetch/node_modules/node-fetch/package.json              │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-fetch/node_modules/whatwg-fetch/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-fetch/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-graphql/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-link/node_modules/tslib/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-link/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-reporting-protobuf/package.json                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-caching/package.json                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-core/node_modules/graphql-tag/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-core/node_modules/graphql-tools/node_modules/uu- │ node-pkg │        0        │    -    │
│ id/package.json                                                                  │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-core/node_modules/graphql-tools/package.json     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-core/node_modules/subscriptions-transport-ws/pa- │ node-pkg │        0        │    -    │
│ ckage.json                                                                       │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-core/node_modules/uuid/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-core/package.json                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-env/node_modules/util.promisify/package.json     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-env/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-errors/package.json                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-express/node_modules/apollo-server-types/packag- │ node-pkg │        0        │    -    │
│ e.json                                                                           │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-express/node_modules/body-parser/package.json    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-express/node_modules/cookie/package.json         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-express/node_modules/debug/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-express/node_modules/encodeurl/package.json      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-express/node_modules/express/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-express/node_modules/finalhandler/package.json   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-express/node_modules/graphql-subscriptions/pack- │ node-pkg │        0        │    -    │
│ age.json                                                                         │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-express/node_modules/graphql-tools/package.json  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-express/node_modules/merge-descriptors/package.- │ node-pkg │        0        │    -    │
│ json                                                                             │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-express/node_modules/ms/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-express/node_modules/path-to-regexp/package.json │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-express/node_modules/qs/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-express/node_modules/raw-body/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-express/node_modules/send/node_modules/encodeur- │ node-pkg │        0        │    -    │
│ l/package.json                                                                   │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-express/node_modules/send/node_modules/ms/packa- │ node-pkg │        0        │    -    │
│ ge.json                                                                          │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-express/node_modules/send/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-express/node_modules/serve-static/package.json   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-express/node_modules/subscriptions-transport-ws- │ node-pkg │        0        │    -    │
│ /package.json                                                                    │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-express/node_modules/uuid/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-express/package.json                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-plugin-base/package.json                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server-types/package.json                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server/node_modules/apollo-server-express/package.json  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server/node_modules/body-parser/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server/node_modules/cookie/package.json                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server/node_modules/debug/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server/node_modules/encodeurl/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server/node_modules/express/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server/node_modules/finalhandler/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server/node_modules/graphql-subscriptions/package.json  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server/node_modules/graphql-tools/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server/node_modules/merge-descriptors/package.json      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server/node_modules/ms/package.json                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server/node_modules/path-to-regexp/package.json         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server/node_modules/qs/package.json                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server/node_modules/raw-body/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server/node_modules/send/node_modules/encodeurl/packag- │ node-pkg │        0        │    -    │
│ e.json                                                                           │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server/node_modules/send/node_modules/ms/package.json   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server/node_modules/send/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server/node_modules/serve-static/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server/node_modules/subscriptions-transport-ws/package- │ node-pkg │        0        │    -    │
│ .json                                                                            │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server/node_modules/uuid/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-server/package.json                                     │ node-pkg │        2        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-tracing/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-upload-client/package.json                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-utilities/node_modules/tslib/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/apollo-utilities/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/append-field/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/are-we-there-yet/node_modules/readable-stream/package.json     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/are-we-there-yet/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/argparse/node_modules/sprintf-js/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/argparse/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/arr-diff/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/arr-flatten/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/arr-union/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/array-back/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/array-buffer-byte-length/package.json                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/array-each/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/array-flatten/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/array-slice/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/array-union/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/array-unique/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/array.prototype.reduce/package.json                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/arraybuffer.prototype.slice/package.json                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/asap/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/asciidoctor-opal-runtime/node_modules/glob/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/asciidoctor-opal-runtime/package.json                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/asciidoctor/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/asn1.js/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/asn1/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/assert-never/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/assert-plus/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/assign-symbols/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/async-function/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/async-limiter/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/async-retry/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/async/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/asynckit/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/at-least-node/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/atob/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/auto-load/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/available-typed-arrays/package.json                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/aws-sdk/node_modules/events/package.json                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/aws-sdk/node_modules/ieee754/package.json                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/aws-sdk/node_modules/punycode/package.json                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/aws-sdk/node_modules/sax/package.json                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/aws-sdk/node_modules/url/package.json                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/aws-sdk/node_modules/uuid/package.json                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/aws-sdk/node_modules/xml2js/node_modules/sax/package.json      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/aws-sdk/node_modules/xml2js/package.json                       │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/aws-sdk/node_modules/xmlbuilder/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/aws-sdk/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/aws-sign2/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/aws4/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/axios/package.json                                             │ node-pkg │        2        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/azure-search-client/package.json                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/azure-search-types/package.json                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/babel-plugin-syntax-trailing-function-commas/package.json      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/babel-preset-fbjs/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/babel-walk/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/backo2/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/backoff/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/balanced-match/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/base/node_modules/define-property/package.json                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/base/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/base64-js/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/base64url/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/bcrypt-pbkdf/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/bcryptjs-then/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/bcryptjs/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/binary-extensions/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/bl/package.json                                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/bluebird/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/bn.js/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/body-parser/node_modules/debug/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/body-parser/node_modules/ms/package.json                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/body-parser/node_modules/qs/package.json                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/body-parser/package.json                                       │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/boolbase/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/brace-expansion/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/braces/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/browser-process-hrtime/package.json                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/browserslist/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/bser/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/bson/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/buffer-equal-constant-time/package.json                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/buffer-from/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/buffer-writer/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/buffer/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/buildcheck/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/bunyan/node_modules/moment/package.json                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/bunyan/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/busboy/node_modules/core-util-is/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/busboy/node_modules/isarray/package.json                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/busboy/node_modules/readable-stream/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/busboy/node_modules/string_decoder/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/busboy/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/bytes/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cacache/node_modules/chownr/package.json                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cacache/node_modules/mkdirp/package.json                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cacache/node_modules/p-map/package.json                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cacache/node_modules/rimraf/package.json                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cacache/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cache-base/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cache-manager/node_modules/async/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cache-manager/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/call-bind-apply-helpers/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/call-bind/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/call-bound/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/camel-case/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/camelcase/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/caniuse-lite/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/caseless/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/chalk-template/node_modules/chalk/package.json                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/chalk-template/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/chalk/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/character-parser/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/charenc/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cheerio-select-tmp/package.json                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cheerio/node_modules/dom-serializer/node_modules/entities/pac- │ node-pkg │        0        │    -    │
│ kage.json                                                                        │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cheerio/node_modules/dom-serializer/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cheerio/node_modules/entities/package.json                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cheerio/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/chokidar/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/chownr/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/chromium-pickle-js/package.json                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ci-info/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/class-utils/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/clean-css/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/clean-stack/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cliui/node_modules/wrap-ansi/package.json                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cliui/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/clone/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/collection-visit/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/color-convert/node_modules/color-name/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/color-convert/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/color-name/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/color-string/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/color-support/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/colorette/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/colorspace/node_modules/color/package.json                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/colorspace/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/combined-stream/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/command-exists/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/command-line-args/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/command-line-usage/node_modules/array-back/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/command-line-usage/node_modules/typical/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/command-line-usage/package.json                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/commander/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/component-emitter/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/component-props/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/compressible/node_modules/mime-db/package.json                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/compressible/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/compression/node_modules/bytes/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/compression/node_modules/debug/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/compression/node_modules/ms/package.json                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/compression/node_modules/safe-buffer/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/compression/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/concat-map/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/concat-stream/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/connect-session-knex/node_modules/commander/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/connect-session-knex/node_modules/debug/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/connect-session-knex/node_modules/knex/package.json            │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/connect-session-knex/node_modules/ms/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/connect-session-knex/node_modules/pg-connection-string/packag- │ node-pkg │        0        │    -    │
│ e.json                                                                           │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/connect-session-knex/package.json                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/console-control-strings/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/constantinople/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/content-disposition/package.json                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/content-type/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cookie-parser/node_modules/cookie/package.json                 │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cookie-parser/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cookie-signature/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cookie/package.json                                            │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cookiejar/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cookies/node_modules/depd/package.json                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cookies/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/copy-descriptor/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/core-js-pure/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/core-util-is/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cors/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cpu-features/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cross-fetch/node_modules/node-fetch/package.json               │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cross-fetch/package.json                                       │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cross-fetch/polyfill/package.json                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cross-spawn/node_modules/path-key/package.json                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cross-spawn/node_modules/which/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cross-spawn/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/crypt/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/css-select/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/css-what/package.json                                          │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cssfilter/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cssom/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cssstyle/node_modules/cssom/package.json                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cssstyle/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/cuint/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/custom-error-instance/package.json                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/dashdash/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/data-urls/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/data-view-buffer/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/data-view-byte-length/package.json                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/data-view-byte-offset/package.json                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/dataloader/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/date-utils/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/db-errors/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/debug/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/decamelize/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/decimal.js/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/decode-uri-component/package.json                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/decompress-response/package.json                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/deep-is/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/define-data-property/package.json                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/define-properties/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/define-property/node_modules/is-descriptor/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/define-property/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/delayed-stream/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/delegates/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/denque/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/depd/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/dependency-graph/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/deprecated-decorator/package.json                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/destroy/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/detect-file/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/detect-libc/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/dezalgo/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/dicer/node_modules/core-util-is/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/dicer/node_modules/isarray/package.json                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/dicer/node_modules/readable-stream/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/dicer/node_modules/string_decoder/package.json                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/dicer/package.json                                             │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/diff/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/diff2html/node_modules/highlight.js/package.json               │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/diff2html/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/dir-glob/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/doctypes/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/dom-serializer/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/domelementtype/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/domexception/node_modules/webidl-conversions/package.json      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/domexception/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/domhandler/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/domino/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/dompurify/package.json                                         │ node-pkg │        3        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/domutils/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/dotize/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/dtrace-provider/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/dunder-proto/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ecc-jsbn/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ecdsa-sig-formatter/package.json                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ee-first/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/elasticsearch6/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/elasticsearch7/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/elasticsearch8/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/electron-to-chromium/package.json                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/emoji-regex/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/enabled/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/encodeurl/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/encoding/node_modules/iconv-lite/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/encoding/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/end-of-stream/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/entities/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/env-paths/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/err-code/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/es-abstract/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/es-array-method-boxes-properly/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/es-define-property/package.json                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/es-errors/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/es-object-atoms/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/es-set-tostringtag/package.json                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/es-to-primitive/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/es6-promise/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/escalade/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/escape-html/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/escodegen/node_modules/levn/package.json                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/escodegen/node_modules/optionator/package.json                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/escodegen/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/esm/package.json                                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/esprima/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/estraverse/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/esutils/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/etag/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/event-target-shim/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/eventemitter2/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/eventemitter3/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/events/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/expand-brackets/node_modules/debug/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/expand-brackets/node_modules/ms/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/expand-brackets/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/expand-tilde/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/express-brute/node_modules/underscore/package.json             │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/express-brute/package.json                                     │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/express-session/node_modules/cookie/package.json               │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/express-session/node_modules/debug/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/express-session/node_modules/ms/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/express-session/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/express/node_modules/cookie/package.json                       │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/express/node_modules/debug/package.json                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/express/node_modules/ms/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/express/node_modules/qs/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/express/package.json                                           │ node-pkg │        2        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/extend-shallow/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/extend/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/extglob/node_modules/define-property/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/extglob/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/extract-files/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/extsprintf/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/fast-deep-equal/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/fast-glob/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/fast-json-stable-stringify/package.json                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/fast-levenshtein/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/fast-safe-stringify/package.json                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/fastq/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/fb-watchman/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/fbjs-css-vars/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/fbjs/node_modules/cross-fetch/package.json                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/fbjs/node_modules/cross-fetch/polyfill/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/fbjs/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/fecha/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/file-type/package.json                                         │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/filesize/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/fill-range/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/finalhandler/node_modules/debug/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/finalhandler/node_modules/ms/package.json                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/finalhandler/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/find-replace/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/find-up/node_modules/path-exists/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/find-up/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/find-yarn-workspace-root/package.json                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/findup-sync/node_modules/braces/node_modules/extend-shallow/p- │ node-pkg │        0        │    -    │
│ ackage.json                                                                      │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/findup-sync/node_modules/braces/node_modules/is-extendable/pa- │ node-pkg │        0        │    -    │
│ ckage.json                                                                       │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/findup-sync/node_modules/braces/package.json                   │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/findup-sync/node_modules/define-property/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/findup-sync/node_modules/extend-shallow/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/findup-sync/node_modules/fill-range/node_modules/extend-shall- │ node-pkg │        0        │    -    │
│ ow/package.json                                                                  │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/findup-sync/node_modules/fill-range/node_modules/is-extendabl- │ node-pkg │        0        │    -    │
│ e/package.json                                                                   │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/findup-sync/node_modules/fill-range/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/findup-sync/node_modules/is-extendable/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/findup-sync/node_modules/micromatch/package.json               │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/findup-sync/node_modules/to-regex-range/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/findup-sync/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/fined/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/flagged-respawn/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/flatbuffers/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/fn.name/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/follow-redirects/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/for-each/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/for-in/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/for-own/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/forever-agent/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/form-data/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/formidable/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/forwarded/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/fragment-cache/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/fresh/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/from2/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/fs-capacitor/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/fs-constants/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/fs-extra/node_modules/universalify/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/fs-extra/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/fs-minipass/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/fs.realpath/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/function-bind/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/function.prototype.name/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/functions-have-names/package.json                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/gauge/node_modules/aproba/package.json                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/gauge/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/generate-function/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/gensync/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/get-caller-file/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/get-intrinsic/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/get-proto/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/get-symbol-description/package.json                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/get-value/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/getopts/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/getos/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/getpass/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/glob-parent/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/glob/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/globals/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/globalthis/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/globby/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/gopd/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/graceful-fs/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/graphql-extensions/package.json                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/graphql-list-fields/package.json                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/graphql-rate-limit-directive/node_modules/graphql-tag/package- │ node-pkg │        0        │    -    │
│ .json                                                                            │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/graphql-rate-limit-directive/node_modules/graphql-tools/packa- │ node-pkg │        0        │    -    │
│ ge.json                                                                          │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/graphql-rate-limit-directive/node_modules/uuid/package.json    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/graphql-rate-limit-directive/package.json                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/graphql-subscriptions/package.json                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/graphql-tools/node_modules/tslib/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/graphql-tools/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/graphql-ws/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/graphql/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/group-by/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/har-schema/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/har-validator/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/has-bigints/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/has-property-descriptors/package.json                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/has-proto/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/has-symbols/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/has-tostringtag/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/has-unicode/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/has-value/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/has-values/node_modules/kind-of/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/has-values/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/hasown/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/he/package.json                                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/hexoid/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/highlight.js/package.json                                      │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/hogan.js/node_modules/mkdirp/package.json                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/hogan.js/node_modules/nopt/package.json                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/hogan.js/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/hoist-non-react-statics/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/homedir-polyfill/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/hpagent/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/html-encoding-sniffer/package.json                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/htmlparser2/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/http-cache-semantics/package.json                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/http-errors/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/http-proxy-agent/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/http-signature/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/https-proxy-agent/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/humanize-ms/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/i18next-express-middleware/package.json                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/i18next-node-fs-backend/node_modules/js-yaml/package.json      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/i18next-node-fs-backend/node_modules/json5/package.json        │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/i18next-node-fs-backend/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/i18next/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/iconv-lite/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ieee754/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ignore/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/image-size/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/immutable/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/import-from/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/imurmurhash/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/infer-owner/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/inflight/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/inherits/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ini/package.json                                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/internal-slot/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/interpret/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/into-stream/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/invariant/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ip-address/node_modules/jsbn/package.json                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ip-address/node_modules/sprintf-js/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ip-address/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ip-regex/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ipaddr.js/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-absolute/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-accessor-descriptor/package.json                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-arguments/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-array-buffer/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-async-function/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-bigint/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-binary-path/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-boolean-object/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-buffer/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-callable/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-core-module/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-data-descriptor/package.json                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-data-view/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-date-object/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-descriptor/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-docker/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-extendable/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-extglob/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-finalizationregistry/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-generator-function/package.json                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-glob/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-lambda/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-map/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-number-object/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-number/node_modules/kind-of/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-number/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-plain-object/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-potential-custom-element-name/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-promise/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-property/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-regex/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-relative/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-set/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-shared-array-buffer/package.json                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-stream/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-string/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-symbol/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-typed-array/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-typedarray/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-unc-path/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-weakmap/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-weakref/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-weakset/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-windows/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/is-wsl/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/isarray/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/isexe/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/isobject/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/isomorphic-ws/node_modules/ws/package.json                     │ node-pkg │        2        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/isomorphic-ws/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/isstream/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/iterall/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/jmespath/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/js-base64/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/js-binary/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/js-stringify/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/js-tokens/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/js-yaml/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/jsbi/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/jsbn/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/jsdom/node_modules/parse5/package.json                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/jsdom/node_modules/punycode/package.json                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/jsdom/node_modules/tough-cookie/package.json                   │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/jsdom/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/jsesc/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/json-bignum/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/json-schema-traverse/package.json                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/json-schema/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/json-stable-stringify/node_modules/isarray/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/json-stable-stringify/package.json                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/json-stringify-safe/package.json                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/jsonfile/node_modules/universalify/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/jsonfile/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/jsonify/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/jsonwebtoken/node_modules/semver/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/jsonwebtoken/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/jsprim/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/jstransformer/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/jwa/package.json                                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/jws/package.json                                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/katex/package.json                                             │ node-pkg │        4        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/keygrip/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/kind-of/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/klaw-sync/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/klaw/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/knex/node_modules/commander/package.json                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/knex/node_modules/debug/package.json                           │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/knex/node_modules/uuid/package.json                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/knex/package.json                                              │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/kuler/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ldap-filter/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ldapauth-fork/node_modules/lru-cache/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ldapauth-fork/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ldapjs/node_modules/extsprintf/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ldapjs/node_modules/verror/package.json                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ldapjs/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/liftoff/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/linkify-it/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/locate-path/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/lodash.camelcase/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/lodash.clonedeep/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/lodash.includes/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/lodash.isboolean/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/lodash.isinteger/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/lodash.isnumber/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/lodash.isplainobject/package.json                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/lodash.isstring/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/lodash.once/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/lodash.repeat/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/lodash.sortby/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/lodash/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/logform/node_modules/@colors/colors/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/logform/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/loglevel/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/long-timeout/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/long/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/loose-envify/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/lru-cache/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/luxon/package.json                                             │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/make-dir/node_modules/semver/package.json                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/make-dir/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/make-fetch-happen/node_modules/negotiator/package.json         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/make-fetch-happen/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/make-iterator/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/map-cache/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/map-visit/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/markdown-it-abbr/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/markdown-it-attrs/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/markdown-it-decorate/package.json                              │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/markdown-it-emoji/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/markdown-it-expand-tabs/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/markdown-it-external-links/package.json                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/markdown-it-footnote/package.json                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/markdown-it-imsize/package.json                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/markdown-it-mark/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/markdown-it-mathjax/package.json                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/markdown-it-multimd-table/package.json                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/markdown-it-pivot-table/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/markdown-it-sub/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/markdown-it-sup/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/markdown-it-task-lists/package.json                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/markdown-it/node_modules/entities/package.json                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/markdown-it/package.json                                       │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/math-intrinsics/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/mathjax/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/md5/package.json                                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/mdurl/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/media-typer/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/memory-pager/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/merge-descriptors/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/merge2/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/meros/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/methods/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/micromatch/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/mime-db/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/mime-types/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/mime/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/mimic-response/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/minimalistic-assert/package.json                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/minimatch/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/minimist/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/minipass-collect/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/minipass-fetch/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/minipass-flush/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/minipass-pipeline/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/minipass-sized/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/minipass/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/minizlib/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/mixin-deep/node_modules/is-extendable/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/mixin-deep/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/mkdirp-classic/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/mkdirp/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/moment-timezone/node_modules/moment/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/moment-timezone/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/moment/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/mongodb/package.json                                           │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ms/package.json                                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/mssql/node_modules/tarn/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/mssql/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/multer/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/mv/node_modules/glob/package.json                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/mv/node_modules/rimraf/package.json                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/mv/package.json                                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/mysql2/node_modules/denque/package.json                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/mysql2/node_modules/iconv-lite/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/mysql2/node_modules/lru-cache/package.json                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/mysql2/package.json                                            │ node-pkg │        5        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/named-placeholders/node_modules/lru-cache/package.json         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/named-placeholders/package.json                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/nan/package.json                                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/nan/tools/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/nanoid/package.json                                            │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/nanomatch/node_modules/define-property/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/nanomatch/node_modules/extend-shallow/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/nanomatch/node_modules/is-extendable/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/nanomatch/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/native-duplexpair/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ncp/package.json                                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/nd-table/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/negotiator/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/node-2fa/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/node-addon-api/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/node-cache/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/node-fetch/node_modules/tr46/package.json                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/node-fetch/node_modules/webidl-conversions/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/node-fetch/node_modules/whatwg-url/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/node-fetch/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/node-forge/flash/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/node-forge/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/node-gyp/node_modules/aproba/package.json                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/node-gyp/node_modules/are-we-there-yet/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/node-gyp/node_modules/gauge/package.json                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/node-gyp/node_modules/npmlog/package.json                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/node-gyp/node_modules/readable-stream/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/node-gyp/node_modules/rimraf/package.json                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/node-gyp/node_modules/semver/package.json                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/node-gyp/node_modules/which/package.json                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/node-gyp/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/node-int64/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/node-jose/node_modules/buffer/package.json                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/node-jose/node_modules/pako/package.json                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/node-jose/node_modules/uuid/package.json                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/node-jose/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/node-releases/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/node-uuid/package.json                                         │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/nodemailer/package.json                                        │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/nopt/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/normalize-path/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/notp/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/npmlog/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/nth-check/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/nullthrows/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/nwsapi/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/oauth-sign/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/oauth/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/object-assign/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/object-copy/node_modules/kind-of/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/object-copy/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/object-inspect/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/object-keys/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/object-path/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/object-visit/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/object.assign/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/object.defaults/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/object.getownpropertydescriptors/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/object.map/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/object.pick/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/objection/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/on-finished/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/on-headers/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/once/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/one-time/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/open/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/os-tmpdir/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/own-keys/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/p-is-promise/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/p-limit/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/p-locate/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/p-try/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/packet-reader/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/parse-filepath/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/parse-passwd/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/parse5-htmlparser2-tree-adapter/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/parse5/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/parseurl/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pascal-case/node_modules/lower-case/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pascal-case/node_modules/no-case/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pascal-case/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pascalcase/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-auth0/node_modules/oauth/package.json                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-auth0/node_modules/passport-oauth2/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-auth0/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-azure-ad/node_modules/passport/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-azure-ad/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-cas/node_modules/sax/package.json                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-cas/node_modules/underscore/package.json              │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-cas/node_modules/xml2js/package.json                  │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-cas/node_modules/xmlbuilder/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-cas/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-discord/example/package.json                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-discord/node_modules/oauth/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-discord/node_modules/passport-oauth2/package.json     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-discord/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-dropbox-oauth2/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-facebook/node_modules/oauth/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-facebook/node_modules/passport-oauth2/package.json    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-facebook/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-github2/node_modules/oauth/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-github2/node_modules/passport-oauth2/package.json     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-github2/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-gitlab2/node_modules/oauth/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-gitlab2/node_modules/passport-oauth2/package.json     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-gitlab2/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-google-oauth20/node_modules/oauth/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-google-oauth20/node_modules/passport-oauth2/package.- │ node-pkg │        0        │    -    │
│ json                                                                             │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-google-oauth20/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-jwt/node_modules/jsonwebtoken/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-jwt/node_modules/semver/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-jwt/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-ldapauth/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-local/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-microsoft/example/login/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-microsoft/node_modules/passport-oauth2/package.json   │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-microsoft/package.json                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-oauth/node_modules/oauth/package.json                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-oauth/node_modules/passport-oauth2/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-oauth/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-oauth1/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-oauth2/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-okta-oauth/node_modules/uid2/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-okta-oauth/package.json                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-openidconnect/package.json                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-saml/node_modules/xml2js/node_modules/xmlbuilder/pac- │ node-pkg │        0        │    -    │
│ kage.json                                                                        │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-saml/node_modules/xml2js/package.json                 │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-saml/node_modules/xmlbuilder/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-saml/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-slack-oauth2/node_modules/oauth/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-slack-oauth2/node_modules/passport-oauth2/package.js- │ node-pkg │        0        │    -    │
│ on                                                                               │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-slack-oauth2/node_modules/pkginfo/examples/package.j- │ node-pkg │        0        │    -    │
│ son                                                                              │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-slack-oauth2/node_modules/pkginfo/examples/subdir/pa- │ node-pkg │        0        │    -    │
│ ckage.json                                                                       │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-slack-oauth2/node_modules/pkginfo/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-slack-oauth2/package.json                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-strategy/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-twitch-strategy/node_modules/oauth/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-twitch-strategy/node_modules/passport-oauth2/package- │ node-pkg │        0        │    -    │
│ .json                                                                            │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport-twitch-strategy/package.json                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/passport/package.json                                          │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/patch-package/node_modules/chalk/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/patch-package/node_modules/fs-extra/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/patch-package/node_modules/semver/package.json                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/patch-package/node_modules/slash/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/patch-package/node_modules/universalify/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/patch-package/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/path-is-absolute/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/path-parse/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/path-root-regex/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/path-root/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/path-to-regexp/package.json                                    │ node-pkg │        2        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/path-type/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pause/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/peek-readable/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pem-jwk/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/performance-now/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pg-connection-string/package.json                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pg-cursor/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pg-format/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pg-hstore/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pg-int8/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pg-packet-stream/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pg-pool/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pg-protocol/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pg-pubsub/node_modules/extsprintf/package.json                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pg-pubsub/node_modules/pg-connection-string/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pg-pubsub/node_modules/pg-pool/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pg-pubsub/node_modules/pg/package.json                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pg-pubsub/node_modules/semver/package.json                     │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pg-pubsub/node_modules/verror/package.json                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pg-pubsub/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pg-query-stream/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pg-tsquery/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pg-types/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pg/node_modules/pg-connection-string/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pg/package.json                                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pgpass/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/picocolors/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/picomatch/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pkginfo/examples/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pkginfo/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/posix-character-classes/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/possible-typed-array-names/package.json                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/postgres-array/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/postgres-bytea/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/postgres-date/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/postgres-interval/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/postinstall-postinstall/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/precond/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/prelude-ls/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/process-nextick-args/package.json                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/process/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/promise-inflight/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/promise-retry/node_modules/retry/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/promise-retry/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/promise/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/promised-retry/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/prop-types/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/proxy-addr/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/psl/node_modules/punycode/package.json                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/psl/package.json                                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pug-code-gen/node_modules/pug-attrs/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pug-code-gen/node_modules/void-elements/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pug-code-gen/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pug-error/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pug-filters/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pug-linker/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pug-load/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pug-runtime/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pug-strip-comments/package.json                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pug-walk/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pug/node_modules/is-expression/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pug/node_modules/pug-lexer/package.json                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pug/node_modules/pug-parser/package.json                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pug/node_modules/token-stream/package.json                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pug/package.json                                               │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/pump/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/punycode/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/qr-image/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/qs/package.json                                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/querystring/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/querystringify/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/queue-microtask/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/queue/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/random-bytes/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/range-parser/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/rate-limiter-flexible/package.json                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/raven/node_modules/uuid/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/raven/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/raw-body/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/react-is/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/readable-stream/node_modules/core-util-is/package.json         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/readable-stream/node_modules/safe-buffer/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/readable-stream/node_modules/string_decoder/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/readable-stream/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/readable-web-to-node-stream/package.json                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/readdirp/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/rechoir/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/reflect.getprototypeof/package.json                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/regex-not/node_modules/extend-shallow/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/regex-not/node_modules/is-extendable/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/regex-not/node_modules/safe-regex/package.json                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/regex-not/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/regexp-tree/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/regexp.prototype.flags/package.json                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/rehackt/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/relay-runtime/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/remove-markdown/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/remove-trailing-separator/package.json                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/repeat-element/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/repeat-string/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/request-promise-core/package.json                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/request-promise-native/package.json                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/request-promise/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/request/node_modules/form-data/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/request/node_modules/qs/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/request/node_modules/uuid/package.json                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/request/package.json                                           │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/require-directory/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/require-main-filename/package.json                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/require_optional/node_modules/resolve-from/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/require_optional/node_modules/semver/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/require_optional/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/require_optional/test/nestedTest/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/requires-port/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/resolve-dir/node_modules/global-modules/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/resolve-dir/node_modules/global-prefix/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/resolve-dir/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/resolve-from/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/resolve-url/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/resolve/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/resolve/test/resolver/multirepo/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/resolve/test/resolver/multirepo/packages/package-a/package.js- │ node-pkg │        0        │    -    │
│ on                                                                               │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/resolve/test/resolver/multirepo/packages/package-b/package.js- │ node-pkg │        0        │    -    │
│ on                                                                               │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/resolve/test/resolver/nested_symlinks/mylib/package.json       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ret/package.json                                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/retry/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/reusify/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/rimraf/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/run-parallel/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/safe-array-concat/node_modules/isarray/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/safe-array-concat/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/safe-buffer/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/safe-json-stringify/package.json                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/safe-push-apply/node_modules/isarray/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/safe-push-apply/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/safe-regex-test/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/safe-regex/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/safe-stable-stringify/package.json                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/safer-buffer/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/sanitize-filename/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/saslprep/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/sax/package.json                                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/saxes/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/scim-query-filter-parser/package.json                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/secure-json-parse/benchmarks/package.json                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/secure-json-parse/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/semver/package.json                                            │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/send/node_modules/debug/node_modules/ms/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/send/node_modules/debug/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/send/package.json                                              │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/seq-queue/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/serve-favicon/node_modules/ms/package.json                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/serve-favicon/node_modules/safe-buffer/package.json            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/serve-favicon/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/serve-static/package.json                                      │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/set-blocking/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/set-function-length/package.json                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/set-function-name/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/set-proto/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/set-value/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/setimmediate/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/setprototypeof/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/sha.js/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/shebang-command/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/shebang-regex/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/side-channel-list/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/side-channel-map/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/side-channel-weakmap/package.json                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/side-channel/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/signal-exit/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/signedsource/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/simple-git/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/simple-swizzle/node_modules/is-arrayish/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/simple-swizzle/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/slash/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/smart-buffer/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/snapdragon-node/node_modules/define-property/package.json      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/snapdragon-node/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/snapdragon-util/node_modules/kind-of/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/snapdragon-util/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/snapdragon/node_modules/debug/package.json                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/snapdragon/node_modules/ms/package.json                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/snapdragon/node_modules/source-map/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/snapdragon/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/socks-proxy-agent/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/socks/node_modules/ip-address/package.json                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/socks/node_modules/jsbn/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/socks/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/solr-node/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/source-map-resolve/package.json                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/source-map-url/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/source-map/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/sparse-bitfield/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/split-string/node_modules/extend-shallow/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/split-string/node_modules/is-extendable/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/split-string/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/split2/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/sprintf-js/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/sqlite3/package.json                                           │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/sqlstring/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ssh2-promise/node_modules/ssh2/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ssh2-promise/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ssh2/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/sshpk/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ssri/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/stack-trace/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/static-extend/package.json                                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/statuses/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/stealthy-require/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/stoppable/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/streamsearch/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/string-math/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/string-width/node_modules/emoji-regex/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/string-width/node_modules/is-fullwidth-code-point/package.json │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/string-width/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/string.prototype.trim/package.json                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/string.prototype.trimend/package.json                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/string.prototype.trimstart/package.json                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/string_decoder/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/strip-ansi/node_modules/ansi-regex/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/strip-ansi/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/striptags/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/strtok3/node_modules/@tokenizer/token/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/strtok3/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/subscriptions-transport-ws/node_modules/ws/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/subscriptions-transport-ws/package.json                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/superagent/node_modules/debug/package.json                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/superagent/node_modules/form-data/package.json                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/superagent/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/supports-color/node_modules/has-flag/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/supports-color/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/supports-preserve-symlinks-flag/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/symbol-observable/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/symbol-tree/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/sync-fetch/node_modules/buffer/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/sync-fetch/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/table-layout/node_modules/array-back/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/table-layout/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/tar-fs/package.json                                            │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/tar-stream/node_modules/bl/package.json                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/tar-stream/node_modules/buffer/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/tar-stream/node_modules/readable-stream/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/tar-stream/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/tar/node_modules/chownr/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/tar/node_modules/minipass/package.json                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/tar/node_modules/mkdirp/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/tar/package.json                                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/tarn/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/tedious/node_modules/@types/node/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/tedious/node_modules/bl/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/tedious/node_modules/iconv-lite/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/tedious/node_modules/punycode/package.json                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/tedious/node_modules/readable-stream/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/tedious/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/text-hex/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/thirty-two/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/tildify/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/timed-out/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/tmp/package.json                                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/to-fast-properties/package.json                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/to-function/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/to-object-path/node_modules/kind-of/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/to-object-path/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/to-regex-range/node_modules/is-number/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/to-regex-range/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/to-regex/node_modules/define-property/package.json             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/to-regex/node_modules/extend-shallow/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/to-regex/node_modules/is-extendable/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/to-regex/node_modules/safe-regex/package.json                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/to-regex/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/toidentifier/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/token-types/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/tough-cookie/node_modules/punycode/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/tough-cookie/package.json                                      │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/tr46/node_modules/punycode/package.json                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/tr46/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/triple-beam/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/truncate-utf8-bytes/package.json                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ts-invariant/node_modules/tslib/package.json                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ts-invariant/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/tslib/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/tunnel-agent/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/tunnel/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/turndown/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/tweetnacl/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/twemoji-parser/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/twemoji/node_modules/fs-extra/node_modules/jsonfile/package.j- │ node-pkg │        0        │    -    │
│ son                                                                              │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/twemoji/node_modules/fs-extra/package.json                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/twemoji/node_modules/jsonfile/package.json                     │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/twemoji/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/type-check/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/type-is/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/typed-array-buffer/package.json                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/typed-array-byte-length/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/typed-array-byte-offset/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/typed-array-length/package.json                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/typedarray-to-buffer/package.json                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/typedarray/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/typical/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ua-parser-js/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/uc.micro/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/uid-safe/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/uid2/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/unbox-primitive/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/unc-path-regex/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/underscore/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/undici-types/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/undici/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/union-value/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/unique-filename/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/unique-slug/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/universalify/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/unixify/node_modules/normalize-path/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/unixify/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/unorm/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/unpipe/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/unset-value/node_modules/has-value/node_modules/isobject/pack- │ node-pkg │        0        │    -    │
│ age.json                                                                         │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/unset-value/node_modules/has-value/package.json                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/unset-value/node_modules/has-values/package.json               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/unset-value/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/unxhr/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/update-browserslist-db/package.json                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/uri-js/node_modules/punycode/package.json                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/uri-js/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/urix/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/url-parse/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/use/package.json                                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/uslug/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/utf8-byte-length/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/util-deprecate/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/util/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/utils-merge/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/uuid/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/v8flags/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/valid-url/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/validate.js/package.json                                       │ node-pkg │        1        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/value-or-promise/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/vary/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/vasync/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/verror/node_modules/extsprintf/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/verror/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/w3c-hr-time/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/w3c-xmlserializer/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/webidl-conversions/package.json                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/whatwg-encoding/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/whatwg-mimetype/package.json                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/whatwg-url/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/which-boxed-primitive/package.json                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/which-builtin-type/node_modules/isarray/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/which-builtin-type/package.json                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/which-collection/package.json                                  │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/which-module/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/which-typed-array/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/which/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/wide-align/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/winston-transport/node_modules/readable-stream/package.json    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/winston-transport/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/winston/node_modules/is-stream/package.json                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/winston/node_modules/readable-stream/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/winston/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/with/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/word-wrap/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/wordwrapjs/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/wrap-ansi/package.json                                         │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/wrappy/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/ws/package.json                                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/xml-crypto/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/xml-encryption/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/xml-name-validator/package.json                                │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/xml2js/package.json                                            │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/xmlbuilder/package.json                                        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/xmlchars/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/xpath.js/package.json                                          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/xpath/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/xss/package.json                                               │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/xtend/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/y18n/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/yallist/package.json                                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/yaml/package.json                                              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/yargs-parser/package.json                                      │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/yargs/node_modules/y18n/package.json                           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/yargs/package.json                                             │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/yocto-queue/package.json                                       │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/zen-observable-ts/node_modules/tslib/package.json              │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/zen-observable-ts/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/node_modules/zen-observable/package.json                                    │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ wiki/package.json                                                                │ node-pkg │        0        │    -    │
└──────────────────────────────────────────────────────────────────────────────────┴──────────┴─────────────────┴─────────┘
Legend:
- '-': Not scanned
- '0': Clean (no security findings detected)


Node.js (node-pkg)

Total: 77 (UNKNOWN: 0, LOW: 8, MEDIUM: 36, HIGH: 28, CRITICAL: 5)

┌─────────────────────────────────────┬─────────────────────┬──────────┬──────────┬───────────────────┬────────────────────────────────────┬──────────────────────────────────────────────────────────────┐
│               Library               │    Vulnerability    │ Severity │  Status  │ Installed Version │           Fixed Version            │                            Title                             │
├─────────────────────────────────────┼─────────────────────┼──────────┼──────────┼───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ @babel/traverse (package.json)      │ CVE-2023-45133      │ CRITICAL │ fixed    │ 7.12.13           │ 7.23.2, 8.0.0-alpha.4              │ babel: arbitrary code execution                              │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2023-45133                   │
├─────────────────────────────────────┼─────────────────────┼──────────┤          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ apollo-server (package.json)        │ GHSA-qm7x-rc44-rrqw │ HIGH     │          │ 2.25.2            │ 2.25.3, 3.4.1                      │ Cross-site Scripting Vulnerability in GraphQL Playground     │
│                                     │                     │          │          │                   │                                    │ (distributed by Apollo Server)                               │
│                                     │                     │          │          │                   │                                    │ https://github.com/advisories/GHSA-qm7x-rc44-rrqw            │
│                                     ├─────────────────────┼──────────┤          │                   ├────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│                                     │ GHSA-2p3c-p3qw-69r4 │ MEDIUM   │          │                   │ 2.25.4                             │ The graphql-upload library included in Apollo Server 2 is    │
│                                     │                     │          │          │                   │                                    │ vulnerable to CSRF...                                        │
│                                     │                     │          │          │                   │                                    │ https://github.com/advisories/GHSA-2p3c-p3qw-69r4            │
├─────────────────────────────────────┼─────────────────────┼──────────┤          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ axios (package.json)                │ CVE-2025-27152      │ HIGH     │          │ 0.21.4            │ 1.8.2, 0.30.0                      │ axios: Possible SSRF and Credential Leakage via Absolute URL │
│                                     │                     │          │          │                   │                                    │ in axios Requests...                                         │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2025-27152                   │
│                                     ├─────────────────────┼──────────┤          │                   ├────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│                                     │ CVE-2023-45857      │ MEDIUM   │          │                   │ 1.6.0, 0.28.0                      │ axios: exposure of confidential data stored in cookies       │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2023-45857                   │
│                                     ├─────────────────────┼──────────┤          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│                                     │ CVE-2025-27152      │ HIGH     │          │ 0.27.2            │ 1.8.2, 0.30.0                      │ axios: Possible SSRF and Credential Leakage via Absolute URL │
│                                     │                     │          │          │                   │                                    │ in axios Requests...                                         │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2025-27152                   │
│                                     ├─────────────────────┼──────────┤          │                   ├────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│                                     │ CVE-2023-45857      │ MEDIUM   │          │                   │ 1.6.0, 0.28.0                      │ axios: exposure of confidential data stored in cookies       │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2023-45857                   │
├─────────────────────────────────────┼─────────────────────┼──────────┤          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ body-parser (package.json)          │ CVE-2024-45590      │ HIGH     │          │ 1.20.1            │ 1.20.3                             │ body-parser: Denial of Service Vulnerability in body-parser  │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2024-45590                   │
├─────────────────────────────────────┼─────────────────────┤          │          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ braces (package.json)               │ CVE-2024-4068       │          │          │ 2.3.2             │ 3.0.3                              │ braces: fails to limit the number of characters it can       │
│                                     │                     │          │          │                   │                                    │ handle                                                       │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2024-4068                    │
├─────────────────────────────────────┼─────────────────────┼──────────┤          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ cookie (package.json)               │ CVE-2024-47764      │ LOW      │          │ 0.3.1             │ 0.7.0                              │ cookie: cookie accepts cookie name, path, and domain with    │
│                                     │                     │          │          │                   │                                    │ out of bounds...                                             │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2024-47764                   │
│                                     │                     │          │          ├───────────────────┤                                    │                                                              │
│                                     │                     │          │          │ 0.4.1             │                                    │                                                              │
│                                     │                     │          │          │                   │                                    │                                                              │
│                                     │                     │          │          │                   │                                    │                                                              │
│                                     │                     │          │          ├───────────────────┤                                    │                                                              │
│                                     │                     │          │          │ 0.4.2             │                                    │                                                              │
│                                     │                     │          │          │                   │                                    │                                                              │
│                                     │                     │          │          │                   │                                    │                                                              │
│                                     │                     │          │          ├───────────────────┤                                    │                                                              │
│                                     │                     │          │          │ 0.5.0             │                                    │                                                              │
│                                     │                     │          │          │                   │                                    │                                                              │
│                                     │                     │          │          │                   │                                    │                                                              │
├─────────────────────────────────────┼─────────────────────┼──────────┤          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ cross-fetch (package.json)          │ CVE-2022-1365       │ MEDIUM   │          │ 1.1.1             │ 3.1.5, 2.2.6                       │ cross-fetch: Exposure of Private Personal Information to an  │
│                                     │                     │          │          │                   │                                    │ Unauthorized Actor                                           │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2022-1365                    │
│                                     │                     │          │          ├───────────────────┤                                    │                                                              │
│                                     │                     │          │          │ 3.0.6             │                                    │                                                              │
│                                     │                     │          │          │                   │                                    │                                                              │
│                                     │                     │          │          │                   │                                    │                                                              │
│                                     │                     │          │          ├───────────────────┤                                    │                                                              │
│                                     │                     │          │          │ 3.1.4             │                                    │                                                              │
│                                     │                     │          │          │                   │                                    │                                                              │
│                                     │                     │          │          │                   │                                    │                                                              │
├─────────────────────────────────────┼─────────────────────┼──────────┤          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ cross-spawn (package.json)          │ CVE-2024-21538      │ HIGH     │          │ 7.0.3             │ 7.0.5, 6.0.6                       │ cross-spawn: regular expression denial of service            │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2024-21538                   │
├─────────────────────────────────────┼─────────────────────┤          │          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ css-what (package.json)             │ CVE-2021-33587      │          │          │ 4.0.0             │ 5.0.1                              │ nodejs-css-what: does not ensure that attribute parsing has  │
│                                     │                     │          │          │                   │                                    │ linear time complexity relative...                           │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2021-33587                   │
├─────────────────────────────────────┼─────────────────────┼──────────┤          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ debug (package.json)                │ CVE-2017-16137      │ LOW      │          │ 4.1.1             │ 2.6.9, 3.1.0, 3.2.7, 4.3.1         │ nodejs-debug: Regular expression Denial of Service           │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2017-16137                   │
├─────────────────────────────────────┼─────────────────────┼──────────┼──────────┼───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ dicer (package.json)                │ CVE-2022-24434      │ HIGH     │ affected │ 0.2.5             │                                    │ dicer: nodejs service crash by sending a crafted payload     │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2022-24434                   │
│                                     │                     │          │          ├───────────────────┼────────────────────────────────────┤                                                              │
│                                     │                     │          │          │ 0.3.0             │                                    │                                                              │
│                                     │                     │          │          │                   │                                    │                                                              │
├─────────────────────────────────────┼─────────────────────┤          ├──────────┼───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ dompurify (package.json)            │ CVE-2024-45801      │          │ fixed    │ 2.4.3             │ 2.5.4, 3.1.3                       │ dompurify: XSS vulnerability via prototype pollution         │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2024-45801                   │
│                                     ├─────────────────────┤          │          │                   ├────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│                                     │ CVE-2024-47875      │          │          │                   │ 2.5.0, 3.1.3                       │ dompurify: nesting-based mutation XSS vulnerability          │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2024-47875                   │
│                                     ├─────────────────────┼──────────┤          │                   ├────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│                                     │ CVE-2025-26791      │ MEDIUM   │          │                   │ 3.2.4                              │ dompurify: Mutation XSS in DOMPurify Due to Improper         │
│                                     │                     │          │          │                   │                                    │ Template Literal Handling                                    │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2025-26791                   │
├─────────────────────────────────────┼─────────────────────┤          │          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ express (package.json)              │ CVE-2024-29041      │          │          │ 4.18.2            │ 4.19.2, 5.0.0-beta.3               │ express: cause malformed URLs to be evaluated                │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2024-29041                   │
│                                     ├─────────────────────┼──────────┤          │                   ├────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│                                     │ CVE-2024-43796      │ LOW      │          │                   │ 4.20.0, 5.0.0                      │ express: Improper Input Handling in Express Redirects        │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2024-43796                   │
├─────────────────────────────────────┼─────────────────────┼──────────┼──────────┼───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ express-brute (package.json)        │ GHSA-984p-xq9m-4rjw │ MEDIUM   │ affected │ 1.0.1             │                                    │ Rate Limiting Bypass in express-brute                        │
│                                     │                     │          │          │                   │                                    │ https://github.com/advisories/GHSA-984p-xq9m-4rjw            │
├─────────────────────────────────────┼─────────────────────┼──────────┼──────────┼───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ file-type (package.json)            │ CVE-2022-36313      │ HIGH     │ fixed    │ 15.0.1            │ 16.5.4, 17.1.3                     │ file-type: a malformed MKV file could cause the file type    │
│                                     │                     │          │          │                   │                                    │ detector to...                                               │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2022-36313                   │
├─────────────────────────────────────┼─────────────────────┼──────────┤          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ highlight.js (package.json)         │ GHSA-7wwv-vh3v-89cq │ MEDIUM   │          │ 10.2.1            │ 10.4.1                             │ ReDOS vulnerabities: multiple grammars                       │
│                                     │                     │          │          │                   │                                    │ https://github.com/advisories/GHSA-7wwv-vh3v-89cq            │
│                                     │                     │          │          ├───────────────────┤                                    │                                                              │
│                                     │                     │          │          │ 10.3.1            │                                    │                                                              │
│                                     │                     │          │          │                   │                                    │                                                              │
├─────────────────────────────────────┼─────────────────────┼──────────┤          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ json5 (package.json)                │ CVE-2022-46175      │ HIGH     │          │ 2.0.0             │ 2.2.2, 1.0.2                       │ json5: Prototype Pollution in JSON5 via Parse Method         │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2022-46175                   │
├─────────────────────────────────────┼─────────────────────┼──────────┤          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ katex (package.json)                │ CVE-2024-28243      │ MEDIUM   │          │ 0.12.0            │ 0.16.10                            │ KaTeX is a JavaScript library for TeX math rendering on the  │
│                                     │                     │          │          │                   │                                    │ web....                                                      │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2024-28243                   │
│                                     ├─────────────────────┤          │          │                   │                                    ├──────────────────────────────────────────────────────────────┤
│                                     │ CVE-2024-28245      │          │          │                   │                                    │ KaTeX is a JavaScript library for TeX math rendering on the  │
│                                     │                     │          │          │                   │                                    │ web....                                                      │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2024-28245                   │
│                                     ├─────────────────────┤          │          │                   │                                    ├──────────────────────────────────────────────────────────────┤
│                                     │ CVE-2024-28246      │          │          │                   │                                    │ KaTeX is a JavaScript library for TeX math rendering on the  │
│                                     │                     │          │          │                   │                                    │ web....                                                      │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2024-28246                   │
│                                     ├─────────────────────┤          │          │                   ├────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│                                     │ CVE-2025-23207      │          │          │                   │ 0.16.21                            │ katex: \htmlData does not validate attribute names in KaTeX  │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2025-23207                   │
├─────────────────────────────────────┼─────────────────────┼──────────┤          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ knex (package.json)                 │ CVE-2016-20018      │ HIGH     │          │ 0.21.21           │ 2.4.0                              │ Knex.js has a limited SQL injection vulnerability            │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2016-20018                   │
│                                     │                     │          │          ├───────────────────┤                                    │                                                              │
│                                     │                     │          │          │ 0.21.7            │                                    │                                                              │
│                                     │                     │          │          │                   │                                    │                                                              │
├─────────────────────────────────────┼─────────────────────┤          │          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ luxon (package.json)                │ CVE-2023-22467      │          │          │ 1.25.0            │ 1.28.1, 2.5.2, 3.2.1               │ luxon: Inefficient regular expression complexity in luxon.js │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2023-22467                   │
├─────────────────────────────────────┼─────────────────────┼──────────┤          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ markdown-it (package.json)          │ CVE-2022-21670      │ MEDIUM   │          │ 11.0.1            │ 12.3.2                             │ markdown-it is a Markdown parser. Prior to version 1.3.2,    │
│                                     │                     │          │          │                   │                                    │ special patt ......                                          │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2022-21670                   │
├─────────────────────────────────────┼─────────────────────┤          ├──────────┼───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ markdown-it-decorate (package.json) │ CVE-2020-28459      │          │ affected │ 1.2.2             │                                    │ markdown-it-decorate vulnerable to cross-site scripting      │
│                                     │                     │          │          │                   │                                    │ (XSS)                                                        │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2020-28459                   │
├─────────────────────────────────────┼─────────────────────┤          ├──────────┼───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ micromatch (package.json)           │ CVE-2024-4067       │          │ fixed    │ 3.1.10            │ 4.0.8                              │ micromatch: vulnerable to Regular Expression Denial of       │
│                                     │                     │          │          │                   │                                    │ Service                                                      │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2024-4067                    │
├─────────────────────────────────────┼─────────────────────┤          │          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ mongodb (package.json)              │ CVE-2021-32050      │          │          │ 3.6.5             │ 3.6.10, 4.17.0, 5.8.0              │ Some MongoDB Drivers may erroneously publish events          │
│                                     │                     │          │          │                   │                                    │ containing authent ...                                       │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2021-32050                   │
├─────────────────────────────────────┼─────────────────────┼──────────┤          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ mysql2 (package.json)               │ CVE-2024-21508      │ CRITICAL │          │ 3.1.0             │ 3.9.4                              │ mysql2: Remote Code Execution                                │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2024-21508                   │
│                                     ├─────────────────────┤          │          │                   ├────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│                                     │ CVE-2024-21511      │          │          │                   │ 3.9.7                              │ mysql2: Arbitrary Code Injection due to improper             │
│                                     │                     │          │          │                   │                                    │ sanitization of the timezone parameter...                    │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2024-21511                   │
│                                     ├─────────────────────┼──────────┤          │                   ├────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│                                     │ CVE-2024-21512      │ HIGH     │          │                   │ 3.9.8                              │ mysql2: vulnerable to Prototype Pollution due to improper    │
│                                     │                     │          │          │                   │                                    │ user input sanitization                                      │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2024-21512                   │
│                                     ├─────────────────────┼──────────┤          │                   ├────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│                                     │ CVE-2024-21507      │ MEDIUM   │          │                   │ 3.9.3                              │ mysql2: Improper Input Validation                            │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2024-21507                   │
│                                     ├─────────────────────┤          │          │                   ├────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│                                     │ CVE-2024-21509      │          │          │                   │ 3.9.4                              │ mysql2: Prototype Poisoning                                  │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2024-21509                   │
├─────────────────────────────────────┼─────────────────────┤          │          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ nanoid (package.json)               │ CVE-2024-55565      │          │          │ 3.2.0             │ 5.0.9, 3.3.8                       │ nanoid: nanoid mishandles non-integer values                 │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2024-55565                   │
├─────────────────────────────────────┼─────────────────────┼──────────┤          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ node-fetch (package.json)           │ CVE-2022-0235       │ HIGH     │          │ 1.7.3             │ 3.1.1, 2.6.7                       │ node-fetch: exposure of sensitive information to an          │
│                                     │                     │          │          │                   │                                    │ unauthorized actor                                           │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2022-0235                    │
│                                     │                     │          │          ├───────────────────┤                                    │                                                              │
│                                     │                     │          │          │ 2.6.1             │                                    │                                                              │
│                                     │                     │          │          │                   │                                    │                                                              │
│                                     │                     │          │          │                   │                                    │                                                              │
│                                     │                     │          │          │                   │                                    │                                                              │
│                                     │                     │          │          │                   │                                    │                                                              │
│                                     │                     │          │          │                   │                                    │                                                              │
│                                     │                     │          │          │                   │                                    │                                                              │
├─────────────────────────────────────┼─────────────────────┼──────────┤          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ node-uuid (package.json)            │ CVE-2015-8851       │ MEDIUM   │          │ 1.4.1             │ >=1.4.4                            │ nodejs-node-uuid: insecure entropy source - Math.random()    │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2015-8851                    │
├─────────────────────────────────────┼─────────────────────┤          │          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ nodemailer (package.json)           │ GHSA-9h6g-pr28-7cqp │          │          │ 6.9.1             │ 6.9.9                              │ nodemailer ReDoS when trying to send a specially crafted     │
│                                     │                     │          │          │                   │                                    │ email                                                        │
│                                     │                     │          │          │                   │                                    │ https://github.com/advisories/GHSA-9h6g-pr28-7cqp            │
├─────────────────────────────────────┼─────────────────────┤          │          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ passport (package.json)             │ CVE-2022-25896      │          │          │ 0.4.1             │ 0.6.0                              │ passport: incorrect session regeneration                     │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2022-25896                   │
├─────────────────────────────────────┼─────────────────────┤          │          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ passport-oauth2 (package.json)      │ CVE-2021-41580      │          │          │ 1.2.0             │ 1.6.1                              │ Improper Access Control in passport-oauth2                   │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2021-41580                   │
├─────────────────────────────────────┼─────────────────────┼──────────┤          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ path-to-regexp (package.json)       │ CVE-2024-45296      │ HIGH     │          │ 0.1.7             │ 1.9.0, 0.1.10, 8.0.0, 3.3.0, 6.3.0 │ path-to-regexp: Backtracking regular expressions cause ReDoS │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2024-45296                   │
│                                     ├─────────────────────┤          │          │                   ├────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│                                     │ CVE-2024-52798      │          │          │                   │ 0.1.12                             │ path-to-regexp: path-to-regexp Unpatched `path-to-regexp`    │
│                                     │                     │          │          │                   │                                    │ ReDoS in 0.1.x                                               │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2024-52798                   │
├─────────────────────────────────────┼─────────────────────┼──────────┤          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ pug (package.json)                  │ CVE-2024-36361      │ MEDIUM   │          │ 3.0.2             │ 3.0.3                              │ Pug allows JavaScript code execution if an application       │
│                                     │                     │          │          │                   │                                    │ accepts untrusted input                                      │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2024-36361                   │
├─────────────────────────────────────┼─────────────────────┤          ├──────────┼───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ request (package.json)              │ CVE-2023-28155      │          │ affected │ 2.88.2            │                                    │ The Request package through 2.88.1 for Node.js allows a      │
│                                     │                     │          │          │                   │                                    │ bypass of SSRF...                                            │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2023-28155                   │
├─────────────────────────────────────┼─────────────────────┼──────────┼──────────┼───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ semver (package.json)               │ CVE-2022-25883      │ HIGH     │ fixed    │ 4.3.2             │ 7.5.2, 6.3.1, 5.7.2                │ nodejs-semver: Regular expression denial of service          │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2022-25883                   │
│                                     │                     │          │          ├───────────────────┤                                    │                                                              │
│                                     │                     │          │          │ 7.3.8             │                                    │                                                              │
│                                     │                     │          │          │                   │                                    │                                                              │
├─────────────────────────────────────┼─────────────────────┼──────────┤          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ send (package.json)                 │ CVE-2024-43799      │ LOW      │          │ 0.18.0            │ 0.19.0                             │ send: Code Execution Vulnerability in Send Library           │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2024-43799                   │
├─────────────────────────────────────┼─────────────────────┤          │          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ serve-static (package.json)         │ CVE-2024-43800      │          │          │ 1.15.0            │ 1.16.0, 2.1.0                      │ serve-static: Improper Sanitization in serve-static          │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2024-43800                   │
├─────────────────────────────────────┼─────────────────────┼──────────┤          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ sqlite3 (package.json)              │ CVE-2022-43441      │ HIGH     │          │ 5.1.4             │ 5.1.5                              │ A code execution vulnerability exists in the Statement       │
│                                     │                     │          │          │                   │                                    │ Bindings functi ...                                          │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2022-43441                   │
├─────────────────────────────────────┼─────────────────────┤          │          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ tar-fs (package.json)               │ CVE-2024-12905      │          │          │ 2.1.1             │ 1.16.4, 2.1.2, 3.0.8               │ tar-fs: link following and path traversal via maliciously    │
│                                     │                     │          │          │                   │                                    │ crafted tar file                                             │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2024-12905                   │
├─────────────────────────────────────┼─────────────────────┼──────────┤          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ tough-cookie (package.json)         │ CVE-2023-26136      │ MEDIUM   │          │ 2.5.0             │ 4.1.3                              │ tough-cookie: prototype pollution in cookie memstore         │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2023-26136                   │
│                                     │                     │          │          ├───────────────────┤                                    │                                                              │
│                                     │                     │          │          │ 3.0.1             │                                    │                                                              │
│                                     │                     │          │          │                   │                                    │                                                              │
├─────────────────────────────────────┼─────────────────────┼──────────┤          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ underscore (package.json)           │ CVE-2021-23358      │ CRITICAL │          │ 1.6.0             │ 1.12.1                             │ nodejs-underscore: Arbitrary code execution via the template │
│                                     │                     │          │          │                   │                                    │ function                                                     │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2021-23358                   │
│                                     │                     │          │          ├───────────────────┤                                    │                                                              │
│                                     │                     │          │          │ 1.8.3             │                                    │                                                              │
│                                     │                     │          │          │                   │                                    │                                                              │
│                                     │                     │          │          │                   │                                    │                                                              │
├─────────────────────────────────────┼─────────────────────┼──────────┼──────────┼───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ validate.js (package.json)          │ CVE-2020-26308      │ MEDIUM   │ affected │ 0.13.1            │                                    │ validate.js Regular Expression Denial of Service             │
│                                     │                     │          │          │                   │                                    │ vulnerability                                                │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2020-26308                   │
├─────────────────────────────────────┼─────────────────────┼──────────┼──────────┼───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ ws (package.json)                   │ CVE-2024-37890      │ HIGH     │ fixed    │ 7.4.5             │ 5.2.4, 6.2.3, 7.5.10, 8.17.1       │ nodejs-ws: denial of service when handling a request with    │
│                                     │                     │          │          │                   │                                    │ many HTTP headers...                                         │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2024-37890                   │
│                                     │                     │          │          │                   │                                    │                                                              │
│                                     │                     │          │          │                   │                                    │                                                              │
│                                     │                     │          │          │                   │                                    │                                                              │
│                                     │                     │          │          │                   │                                    │                                                              │
│                                     ├─────────────────────┼──────────┤          │                   ├────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│                                     │ CVE-2021-32640      │ MEDIUM   │          │                   │ 7.4.6, 6.2.2, 5.2.3                │ nodejs-ws: Specially crafted value of the                    │
│                                     │                     │          │          │                   │                                    │ `Sec-Websocket-Protocol` header can be used to...            │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2021-32640                   │
│                                     │                     │          │          │                   │                                    │                                                              │
│                                     │                     │          │          │                   │                                    │                                                              │
│                                     │                     │          │          │                   │                                    │                                                              │
│                                     │                     │          │          │                   │                                    │                                                              │
├─────────────────────────────────────┼─────────────────────┤          │          ├───────────────────┼────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ xml2js (package.json)               │ CVE-2023-0842       │          │          │ 0.4.19            │ 0.5.0                              │ node-xml2js: xml2js is vulnerable to prototype pollution     │
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2023-0842                    │
│                                     │                     │          │          ├───────────────────┤                                    │                                                              │
│                                     │                     │          │          │ 0.4.23            │                                    │                                                              │
│                                     │                     │          │          │                   │                                    │                                                              │
│                                     │                     │          │          ├───────────────────┤                                    │                                                              │
│                                     │                     │          │          │ 0.4.4             │                                    │                                                              │
│                                     │                     │          │          │                   │                                    │                                                              │
└─────────────────────────────────────┴─────────────────────┴──────────┴──────────┴───────────────────┴────────────────────────────────────┴──────────────────────────────────────────────────────────────┘
```

[Sommaire des scans](#2-scan-de-vuln)

# Scan apache

```
azureuser@wawaa:~/travail$ trivy image apache
2025-03-31T20:38:46Z    INFO    [vuln] Vulnerability scanning is enabled
2025-03-31T20:38:46Z    INFO    [secret] Secret scanning is enabled
2025-03-31T20:38:46Z    INFO    [secret] If your scanning is slow, please try '--scanners vuln' to disable secret scanning
2025-03-31T20:38:46Z    INFO    [secret] Please see also https://trivy.dev/v0.61/docs/scanner/secret#recommendation for faster secret detection
2025-03-31T20:39:22Z    INFO    Detected OS     family="ubuntu" version="24.04"
2025-03-31T20:39:22Z    INFO    [ubuntu] Detecting vulnerabilities...   os_version="24.04" pkg_num=135
2025-03-31T20:39:22Z    INFO    Number of language-specific files       num=0

Report Summary

┌────────────────────────────────────────┬────────┬─────────────────┬─────────┐
│                 Target                 │  Type  │ Vulnerabilities │ Secrets │
├────────────────────────────────────────┼────────┼─────────────────┼─────────┤
│ apache (ubuntu 24.04)                  │ ubuntu │       24        │    -    │
├────────────────────────────────────────┼────────┼─────────────────┼─────────┤
│ /etc/ssl/private/ssl-cert-snakeoil.key │  text  │        -        │    1    │
└────────────────────────────────────────┴────────┴─────────────────┴─────────┘
Legend:
- '-': Not scanned
- '0': Clean (no security findings detected)


apache (ubuntu 24.04)

Total: 24 (UNKNOWN: 0, LOW: 10, MEDIUM: 14, HIGH: 0, CRITICAL: 0)

┌────────────────────┬────────────────┬──────────┬──────────┬─────────────────────────┬─────────────────────────┬──────────────────────────────────────────────────────────────┐
│      Library       │ Vulnerability  │ Severity │  Status  │    Installed Version    │      Fixed Version      │                            Title                             │
├────────────────────┼────────────────┼──────────┼──────────┼─────────────────────────┼─────────────────────────┼──────────────────────────────────────────────────────────────┤
│ coreutils          │ CVE-2016-2781  │ LOW      │ affected │ 9.4-3ubuntu6            │                         │ coreutils: Non-privileged session can escape to the parent   │
│                    │                │          │          │                         │                         │ session in chroot                                            │
│                    │                │          │          │                         │                         │ https://avd.aquasec.com/nvd/cve-2016-2781                    │
├────────────────────┼────────────────┤          │          ├─────────────────────────┼─────────────────────────┼──────────────────────────────────────────────────────────────┤
│ gpgv               │ CVE-2022-3219  │          │          │ 2.4.4-2ubuntu17         │                         │ gnupg: denial of service issue (resource consumption) using  │
│                    │                │          │          │                         │                         │ compressed packets                                           │
│                    │                │          │          │                         │                         │ https://avd.aquasec.com/nvd/cve-2022-3219                    │
├────────────────────┼────────────────┼──────────┼──────────┼─────────────────────────┼─────────────────────────┼──────────────────────────────────────────────────────────────┤
│ libc-bin           │ CVE-2025-0395  │ MEDIUM   │ fixed    │ 2.39-0ubuntu8.3         │ 2.39-0ubuntu8.4         │ glibc: buffer overflow in the GNU C Library's assert()       │
│                    │                │          │          │                         │                         │ https://avd.aquasec.com/nvd/cve-2025-0395                    │
│                    ├────────────────┼──────────┼──────────┤                         ├─────────────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2016-20013 │ LOW      │ affected │                         │                         │ sha256crypt and sha512crypt through 0.6 allow attackers to   │
│                    │                │          │          │                         │                         │ cause a denial of...                                         │
│                    │                │          │          │                         │                         │ https://avd.aquasec.com/nvd/cve-2016-20013                   │
├────────────────────┼────────────────┼──────────┼──────────┤                         ├─────────────────────────┼──────────────────────────────────────────────────────────────┤
│ libc6              │ CVE-2025-0395  │ MEDIUM   │ fixed    │                         │ 2.39-0ubuntu8.4         │ glibc: buffer overflow in the GNU C Library's assert()       │
│                    │                │          │          │                         │                         │ https://avd.aquasec.com/nvd/cve-2025-0395                    │
│                    ├────────────────┼──────────┼──────────┤                         ├─────────────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2016-20013 │ LOW      │ affected │                         │                         │ sha256crypt and sha512crypt through 0.6 allow attackers to   │
│                    │                │          │          │                         │                         │ cause a denial of...                                         │
│                    │                │          │          │                         │                         │ https://avd.aquasec.com/nvd/cve-2016-20013                   │
├────────────────────┼────────────────┼──────────┼──────────┼─────────────────────────┼─────────────────────────┼──────────────────────────────────────────────────────────────┤
│ libcap2            │ CVE-2025-1390  │ MEDIUM   │ fixed    │ 1:2.66-5ubuntu2         │ 1:2.66-5ubuntu2.2       │ libcap: pam_cap: Fix potential configuration parsing error   │
│                    │                │          │          │                         │                         │ https://avd.aquasec.com/nvd/cve-2025-1390                    │
├────────────────────┼────────────────┼──────────┼──────────┼─────────────────────────┼─────────────────────────┼──────────────────────────────────────────────────────────────┤
│ libcurl4t64        │ CVE-2025-0167  │ LOW      │ affected │ 8.5.0-2ubuntu10.6       │                         │ When asked to use a `.netrc` file for credentials **and** to │
│                    │                │          │          │                         │                         │ follow...                                                    │
│                    │                │          │          │                         │                         │ https://avd.aquasec.com/nvd/cve-2025-0167                    │
├────────────────────┼────────────────┼──────────┤          ├─────────────────────────┼─────────────────────────┼──────────────────────────────────────────────────────────────┤
│ libexpat1          │ CVE-2024-8176  │ MEDIUM   │          │ 2.6.1-2ubuntu0.2        │                         │ libexpat: expat: Improper Restriction of XML Entity          │
│                    │                │          │          │                         │                         │ Expansion Depth in libexpat                                  │
│                    │                │          │          │                         │                         │ https://avd.aquasec.com/nvd/cve-2024-8176                    │
├────────────────────┼────────────────┼──────────┤          ├─────────────────────────┼─────────────────────────┼──────────────────────────────────────────────────────────────┤
│ libgcrypt20        │ CVE-2024-2236  │ LOW      │          │ 1.10.3-2build1          │                         │ libgcrypt: vulnerable to Marvin Attack                       │
│                    │                │          │          │                         │                         │ https://avd.aquasec.com/nvd/cve-2024-2236                    │
├────────────────────┼────────────────┼──────────┼──────────┼─────────────────────────┼─────────────────────────┼──────────────────────────────────────────────────────────────┤
│ libgnutls30t64     │ CVE-2024-12243 │ MEDIUM   │ fixed    │ 3.8.3-1.1ubuntu3.2      │ 3.8.3-1.1ubuntu3.3      │ gnutls: GnuTLS Impacted by Inefficient DER Decoding in       │
│                    │                │          │          │                         │                         │ libtasn1 Leading to Remote...                                │
│                    │                │          │          │                         │                         │ https://avd.aquasec.com/nvd/cve-2024-12243                   │
├────────────────────┼────────────────┤          ├──────────┼─────────────────────────┼─────────────────────────┼──────────────────────────────────────────────────────────────┤
│ libpam-modules     │ CVE-2024-10041 │          │ affected │ 1.5.3-5ubuntu5.1        │                         │ pam: libpam: Libpam vulnerable to read hashed password       │
│                    │                │          │          │                         │                         │ https://avd.aquasec.com/nvd/cve-2024-10041                   │
│                    ├────────────────┤          │          │                         ├─────────────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2024-10963 │          │          │                         │                         │ pam: Improper Hostname Interpretation in pam_access Leads to │
│                    │                │          │          │                         │                         │ Access Control Bypass                                        │
│                    │                │          │          │                         │                         │ https://avd.aquasec.com/nvd/cve-2024-10963                   │
├────────────────────┼────────────────┤          │          │                         ├─────────────────────────┼──────────────────────────────────────────────────────────────┤
│ libpam-modules-bin │ CVE-2024-10041 │          │          │                         │                         │ pam: libpam: Libpam vulnerable to read hashed password       │
│                    │                │          │          │                         │                         │ https://avd.aquasec.com/nvd/cve-2024-10041                   │
│                    ├────────────────┤          │          │                         ├─────────────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2024-10963 │          │          │                         │                         │ pam: Improper Hostname Interpretation in pam_access Leads to │
│                    │                │          │          │                         │                         │ Access Control Bypass                                        │
│                    │                │          │          │                         │                         │ https://avd.aquasec.com/nvd/cve-2024-10963                   │
├────────────────────┼────────────────┤          │          │                         ├─────────────────────────┼──────────────────────────────────────────────────────────────┤
│ libpam-runtime     │ CVE-2024-10041 │          │          │                         │                         │ pam: libpam: Libpam vulnerable to read hashed password       │
│                    │                │          │          │                         │                         │ https://avd.aquasec.com/nvd/cve-2024-10041                   │
│                    ├────────────────┤          │          │                         ├─────────────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2024-10963 │          │          │                         │                         │ pam: Improper Hostname Interpretation in pam_access Leads to │
│                    │                │          │          │                         │                         │ Access Control Bypass                                        │
│                    │                │          │          │                         │                         │ https://avd.aquasec.com/nvd/cve-2024-10963                   │
├────────────────────┼────────────────┤          │          │                         ├─────────────────────────┼──────────────────────────────────────────────────────────────┤
│ libpam0g           │ CVE-2024-10041 │          │          │                         │                         │ pam: libpam: Libpam vulnerable to read hashed password       │
│                    │                │          │          │                         │                         │ https://avd.aquasec.com/nvd/cve-2024-10041                   │
│                    ├────────────────┤          │          │                         ├─────────────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2024-10963 │          │          │                         │                         │ pam: Improper Hostname Interpretation in pam_access Leads to │
│                    │                │          │          │                         │                         │ Access Control Bypass                                        │
│                    │                │          │          │                         │                         │ https://avd.aquasec.com/nvd/cve-2024-10963                   │
├────────────────────┼────────────────┼──────────┤          ├─────────────────────────┼─────────────────────────┼──────────────────────────────────────────────────────────────┤
│ libssl3t64         │ CVE-2024-41996 │ LOW      │          │ 3.0.13-0ubuntu3.5       │                         │ openssl: remote attackers (from the client side) to trigger  │
│                    │                │          │          │                         │                         │ unnecessarily expensive server-side...                       │
│                    │                │          │          │                         │                         │ https://avd.aquasec.com/nvd/cve-2024-41996                   │
├────────────────────┼────────────────┼──────────┼──────────┼─────────────────────────┼─────────────────────────┼──────────────────────────────────────────────────────────────┤
│ libtasn1-6         │ CVE-2024-12133 │ MEDIUM   │ fixed    │ 4.19.0-3build1          │ 4.19.0-3ubuntu0.24.04.1 │ libtasn1: Inefficient DER Decoding in libtasn1 Leading to    │
│                    │                │          │          │                         │                         │ Potential Remote DoS                                         │
│                    │                │          │          │                         │                         │ https://avd.aquasec.com/nvd/cve-2024-12133                   │
├────────────────────┼────────────────┼──────────┼──────────┼─────────────────────────┼─────────────────────────┼──────────────────────────────────────────────────────────────┤
│ login              │ CVE-2024-56433 │ LOW      │ affected │ 1:4.13+dfsg1-4ubuntu3.2 │                         │ shadow-utils: Default subordinate ID configuration in        │
│                    │                │          │          │                         │                         │ /etc/login.defs could lead to compromise                     │
│                    │                │          │          │                         │                         │ https://avd.aquasec.com/nvd/cve-2024-56433                   │
├────────────────────┼────────────────┤          │          ├─────────────────────────┼─────────────────────────┼──────────────────────────────────────────────────────────────┤
│ openssl            │ CVE-2024-41996 │          │          │ 3.0.13-0ubuntu3.5       │                         │ openssl: remote attackers (from the client side) to trigger  │
│                    │                │          │          │                         │                         │ unnecessarily expensive server-side...                       │
│                    │                │          │          │                         │                         │ https://avd.aquasec.com/nvd/cve-2024-41996                   │
├────────────────────┼────────────────┤          │          ├─────────────────────────┼─────────────────────────┼──────────────────────────────────────────────────────────────┤
│ passwd             │ CVE-2024-56433 │          │          │ 1:4.13+dfsg1-4ubuntu3.2 │                         │ shadow-utils: Default subordinate ID configuration in        │
│                    │                │          │          │                         │                         │ /etc/login.defs could lead to compromise                     │
│                    │                │          │          │                         │                         │ https://avd.aquasec.com/nvd/cve-2024-56433                   │
└────────────────────┴────────────────┴──────────┴──────────┴─────────────────────────┴─────────────────────────┴──────────────────────────────────────────────────────────────┘

/etc/ssl/private/ssl-cert-snakeoil.key (secrets)

Total: 1 (UNKNOWN: 0, LOW: 0, MEDIUM: 0, HIGH: 1, CRITICAL: 0)

HIGH: AsymmetricPrivateKey (private-key)
═════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════
Asymmetric Private Key
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
 /etc/ssl/private/ssl-cert-snakeoil.key:1 (added by 'RUN /bin/sh -c apt install -y apache2 # ')
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1 [ -----BEGIN PRIVATE KEY-----*******************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************-----END PRIVATE KEY
   2
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
```

- [Sommaire des scans](#2-scan-de-vuln)

# Scan nginx

```
azureuser@wawaa:~$ trivy image nginx
2025-03-31T20:42:33Z    INFO    [vuln] Vulnerability scanning is enabled
2025-03-31T20:42:33Z    INFO    [secret] Secret scanning is enabled
2025-03-31T20:42:33Z    INFO    [secret] If your scanning is slow, please try '--scanners vuln' to disable secret scanning
2025-03-31T20:42:33Z    INFO    [secret] Please see also https://trivy.dev/v0.61/docs/scanner/secret#recommendation for faster secret detection
2025-03-31T20:42:54Z    INFO    [javadb] Downloading Java DB...
2025-03-31T20:42:54Z    INFO    [javadb] Downloading artifact...        repo="mirror.gcr.io/aquasec/trivy-java-db:1"
705.31 MiB / 705.31 MiB [------------------------------------------------------------------------------------------------------------------------------------------------------------] 100.00% 9.58 MiB p/s 1m14s
2025-03-31T20:44:08Z    INFO    [javadb] Artifact successfully downloaded       repo="mirror.gcr.io/aquasec/trivy-java-db:1"
2025-03-31T20:44:08Z    INFO    [javadb] Java DB is cached for 3 days. If you want to update the database more frequently, "trivy clean --java-db" command clears the DB cache.
2025-03-31T20:44:08Z    INFO    Detected OS     family="debian" version="12.10"
2025-03-31T20:44:08Z    INFO    [debian] Detecting vulnerabilities...   os_version="12" pkg_num=149
2025-03-31T20:44:08Z    INFO    Number of language-specific files       num=0
2025-03-31T20:44:08Z    WARN    Using severities from other vendors for some vulnerabilities. Read https://trivy.dev/v0.61/docs/scanner/vulnerability#severity-selection for details.

Report Summary

┌──────────────────────┬────────┬─────────────────┬─────────┐
│        Target        │  Type  │ Vulnerabilities │ Secrets │
├──────────────────────┼────────┼─────────────────┼─────────┤
│ nginx (debian 12.10) │ debian │       148       │    -    │
└──────────────────────┴────────┴─────────────────┴─────────┘
Legend:
- '-': Not scanned
- '0': Clean (no security findings detected)


nginx (debian 12.10)

Total: 148 (UNKNOWN: 0, LOW: 98, MEDIUM: 36, HIGH: 12, CRITICAL: 2)

┌────────────────────┬─────────────────────┬──────────┬──────────────┬─────────────────────────┬──────────────────┬──────────────────────────────────────────────────────────────┐
│      Library       │    Vulnerability    │ Severity │    Status    │    Installed Version    │  Fixed Version   │                            Title                             │
├────────────────────┼─────────────────────┼──────────┼──────────────┼─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ apt                │ CVE-2011-3374       │ LOW      │ affected     │ 2.6.1                   │                  │ It was found that apt-key in apt, all versions, do not       │
│                    │                     │          │              │                         │                  │ correctly...                                                 │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2011-3374                    │
├────────────────────┼─────────────────────┤          │              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ bash               │ TEMP-0841856-B18BAF │          │              │ 5.2.15-2+b7             │                  │ [Privilege escalation possible to other user than root]      │
│                    │                     │          │              │                         │                  │ https://security-tracker.debian.org/tracker/TEMP-0841856-B1- │
│                    │                     │          │              │                         │                  │ 8BAF                                                         │
├────────────────────┼─────────────────────┤          │              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ bsdutils           │ CVE-2022-0563       │          │              │ 1:2.38.1-5+deb12u3      │                  │ util-linux: partial disclosure of arbitrary files in chfn    │
│                    │                     │          │              │                         │                  │ and chsh when compiled...                                    │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2022-0563                    │
├────────────────────┼─────────────────────┤          ├──────────────┼─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ coreutils          │ CVE-2016-2781       │          │ will_not_fix │ 9.1-1                   │                  │ coreutils: Non-privileged session can escape to the parent   │
│                    │                     │          │              │                         │                  │ session in chroot                                            │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2016-2781                    │
│                    ├─────────────────────┤          ├──────────────┤                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2017-18018      │          │ affected     │                         │                  │ coreutils: race condition vulnerability in chown and chgrp   │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2017-18018                   │
├────────────────────┼─────────────────────┤          │              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ curl               │ CVE-2024-2379       │          │              │ 7.88.1-10+deb12u12      │                  │ curl: QUIC certificate check bypass with wolfSSL             │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-2379                    │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2025-0725       │          │              │                         │                  │ libcurl: Buffer Overflow in libcurl via zlib Integer         │
│                    │                     │          │              │                         │                  │ Overflow                                                     │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2025-0725                    │
├────────────────────┼─────────────────────┤          │              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ gcc-12-base        │ CVE-2022-27943      │          │              │ 12.2.0-14               │                  │ binutils: libiberty/rust-demangle.c in GNU GCC 11.2 allows   │
│                    │                     │          │              │                         │                  │ stack exhaustion in demangle_const                           │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2022-27943                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2023-4039       │          │              │                         │                  │ gcc: -fstack-protector fails to guard dynamic stack          │
│                    │                     │          │              │                         │                  │ allocations on ARM64                                         │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-4039                    │
├────────────────────┼─────────────────────┤          │              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ gpgv               │ CVE-2022-3219       │          │              │ 2.2.40-1.1              │                  │ gnupg: denial of service issue (resource consumption) using  │
│                    │                     │          │              │                         │                  │ compressed packets                                           │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2022-3219                    │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2025-30258      │          │              │                         │                  │ gnupg: verification DoS due to a malicious subkey in the     │
│                    │                     │          │              │                         │                  │ keyring                                                      │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2025-30258                   │
├────────────────────┼─────────────────────┼──────────┤              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ libabsl20220623    │ CVE-2025-0838       │ MEDIUM   │              │ 20220623.1-1            │                  │ abseil-cpp: Heap Buffer overflow in Abseil                   │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2025-0838                    │
├────────────────────┼─────────────────────┼──────────┤              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ libaom3            │ CVE-2023-6879       │ CRITICAL │              │ 3.6.0-1+deb12u1         │                  │ aom: heap-buffer-overflow on frame size change               │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-6879                    │
│                    ├─────────────────────┼──────────┼──────────────┤                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2023-39616      │ HIGH     │ will_not_fix │                         │                  │ AOMedia v3.0.0 to v3.5.0 was discovered to contain an        │
│                    │                     │          │              │                         │                  │ invalid read mem...                                          │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-39616                   │
├────────────────────┼─────────────────────┼──────────┼──────────────┼─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ libapt-pkg6.0      │ CVE-2011-3374       │ LOW      │ affected     │ 2.6.1                   │                  │ It was found that apt-key in apt, all versions, do not       │
│                    │                     │          │              │                         │                  │ correctly...                                                 │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2011-3374                    │
├────────────────────┼─────────────────────┤          │              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ libblkid1          │ CVE-2022-0563       │          │              │ 2.38.1-5+deb12u3        │                  │ util-linux: partial disclosure of arbitrary files in chfn    │
│                    │                     │          │              │                         │                  │ and chsh when compiled...                                    │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2022-0563                    │
├────────────────────┼─────────────────────┤          │              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ libc-bin           │ CVE-2010-4756       │          │              │ 2.36-9+deb12u10         │                  │ glibc: glob implementation can cause excessive CPU and       │
│                    │                     │          │              │                         │                  │ memory consumption due to...                                 │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2010-4756                    │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2018-20796      │          │              │                         │                  │ glibc: uncontrolled recursion in function                    │
│                    │                     │          │              │                         │                  │ check_dst_limits_calc_pos_1 in posix/regexec.c               │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2018-20796                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2019-1010022    │          │              │                         │                  │ glibc: stack guard protection bypass                         │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2019-1010022                 │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2019-1010023    │          │              │                         │                  │ glibc: running ldd on malicious ELF leads to code execution  │
│                    │                     │          │              │                         │                  │ because of...                                                │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2019-1010023                 │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2019-1010024    │          │              │                         │                  │ glibc: ASLR bypass using cache of thread stack and heap      │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2019-1010024                 │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2019-1010025    │          │              │                         │                  │ glibc: information disclosure of heap addresses of           │
│                    │                     │          │              │                         │                  │ pthread_created thread                                       │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2019-1010025                 │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2019-9192       │          │              │                         │                  │ glibc: uncontrolled recursion in function                    │
│                    │                     │          │              │                         │                  │ check_dst_limits_calc_pos_1 in posix/regexec.c               │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2019-9192                    │
├────────────────────┼─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│ libc6              │ CVE-2010-4756       │          │              │                         │                  │ glibc: glob implementation can cause excessive CPU and       │
│                    │                     │          │              │                         │                  │ memory consumption due to...                                 │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2010-4756                    │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2018-20796      │          │              │                         │                  │ glibc: uncontrolled recursion in function                    │
│                    │                     │          │              │                         │                  │ check_dst_limits_calc_pos_1 in posix/regexec.c               │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2018-20796                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2019-1010022    │          │              │                         │                  │ glibc: stack guard protection bypass                         │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2019-1010022                 │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2019-1010023    │          │              │                         │                  │ glibc: running ldd on malicious ELF leads to code execution  │
│                    │                     │          │              │                         │                  │ because of...                                                │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2019-1010023                 │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2019-1010024    │          │              │                         │                  │ glibc: ASLR bypass using cache of thread stack and heap      │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2019-1010024                 │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2019-1010025    │          │              │                         │                  │ glibc: information disclosure of heap addresses of           │
│                    │                     │          │              │                         │                  │ pthread_created thread                                       │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2019-1010025                 │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2019-9192       │          │              │                         │                  │ glibc: uncontrolled recursion in function                    │
│                    │                     │          │              │                         │                  │ check_dst_limits_calc_pos_1 in posix/regexec.c               │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2019-9192                    │
├────────────────────┼─────────────────────┼──────────┤              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ libcap2            │ CVE-2025-1390       │ MEDIUM   │              │ 1:2.66-4                │                  │ libcap: pam_cap: Fix potential configuration parsing error   │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2025-1390                    │
├────────────────────┼─────────────────────┼──────────┤              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ libcurl4           │ CVE-2024-2379       │ LOW      │              │ 7.88.1-10+deb12u12      │                  │ curl: QUIC certificate check bypass with wolfSSL             │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-2379                    │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2025-0725       │          │              │                         │                  │ libcurl: Buffer Overflow in libcurl via zlib Integer         │
│                    │                     │          │              │                         │                  │ Overflow                                                     │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2025-0725                    │
├────────────────────┼─────────────────────┼──────────┼──────────────┼─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ libdav1d6          │ CVE-2023-32570      │ MEDIUM   │ will_not_fix │ 1.0.0-2+deb12u1         │                  │ VideoLAN dav1d before 1.2.0 has a thread_task.c race         │
│                    │                     │          │              │                         │                  │ condition that ca ......                                     │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-32570                   │
├────────────────────┼─────────────────────┤          ├──────────────┼─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ libde265-0         │ CVE-2023-51792      │          │ affected     │ 1.0.11-1+deb12u2        │                  │ Buffer Overflow vulnerability in libde265 v1.0.12 allows a   │
│                    │                     │          │              │                         │                  │ local attac ...                                              │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-51792                   │
│                    ├─────────────────────┤          ├──────────────┤                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2024-38949      │          │ fix_deferred │                         │                  │ Heap Buffer Overflow vulnerability in Libde265 v1.0.15       │
│                    │                     │          │              │                         │                  │ allows attacker ...                                          │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-38949                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2024-38950      │          │              │                         │                  │ Heap Buffer Overflow vulnerability in Libde265 v1.0.15       │
│                    │                     │          │              │                         │                  │ allows attacker ...                                          │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-38950                   │
├────────────────────┼─────────────────────┼──────────┼──────────────┼─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ libexpat1          │ CVE-2023-52425      │ HIGH     │ affected     │ 2.5.0-1+deb12u1         │                  │ expat: parsing large tokens can trigger a denial of service  │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-52425                   │
│                    ├─────────────────────┤          ├──────────────┤                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2024-8176       │          │ will_not_fix │                         │                  │ libexpat: expat: Improper Restriction of XML Entity          │
│                    │                     │          │              │                         │                  │ Expansion Depth in libexpat                                  │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-8176                    │
│                    ├─────────────────────┼──────────┼──────────────┤                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2024-50602      │ MEDIUM   │ affected     │                         │                  │ libexpat: expat: DoS via XML_ResumeParser                    │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-50602                   │
│                    ├─────────────────────┼──────────┤              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2023-52426      │ LOW      │              │                         │                  │ expat: recursive XML entity expansion vulnerability          │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-52426                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2024-28757      │          │              │                         │                  │ expat: XML Entity Expansion                                  │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-28757                   │
├────────────────────┼─────────────────────┤          │              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ libgcc-s1          │ CVE-2022-27943      │          │              │ 12.2.0-14               │                  │ binutils: libiberty/rust-demangle.c in GNU GCC 11.2 allows   │
│                    │                     │          │              │                         │                  │ stack exhaustion in demangle_const                           │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2022-27943                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2023-4039       │          │              │                         │                  │ gcc: -fstack-protector fails to guard dynamic stack          │
│                    │                     │          │              │                         │                  │ allocations on ARM64                                         │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-4039                    │
├────────────────────┼─────────────────────┼──────────┼──────────────┼─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ libgcrypt20        │ CVE-2024-2236       │ MEDIUM   │ fix_deferred │ 1.10.1-3                │                  │ libgcrypt: vulnerable to Marvin Attack                       │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-2236                    │
│                    ├─────────────────────┼──────────┼──────────────┤                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2018-6829       │ LOW      │ affected     │                         │                  │ libgcrypt: ElGamal implementation doesn't have semantic      │
│                    │                     │          │              │                         │                  │ security due to incorrectly encoded plaintexts...            │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2018-6829                    │
├────────────────────┼─────────────────────┤          │              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ libgnutls30        │ CVE-2011-3389       │          │              │ 3.7.9-2+deb12u4         │                  │ HTTPS: block-wise chosen-plaintext attack against SSL/TLS    │
│                    │                     │          │              │                         │                  │ (BEAST)                                                      │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2011-3389                    │
├────────────────────┼─────────────────────┼──────────┤              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ libgssapi-krb5-2   │ CVE-2024-26462      │ MEDIUM   │              │ 1.20.1-2+deb12u2        │                  │ krb5: Memory leak at /krb5/src/kdc/ndr.c                     │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-26462                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2025-24528      │          │              │                         │                  │ krb5: overflow when calculating ulog block size              │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2025-24528                   │
│                    ├─────────────────────┼──────────┤              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2018-5709       │ LOW      │              │                         │                  │ krb5: integer overflow in dbentry->n_key_data in             │
│                    │                     │          │              │                         │                  │ kadmin/dbutil/dump.c                                         │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2018-5709                    │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2024-26458      │          │              │                         │                  │ krb5: Memory leak at /krb5/src/lib/rpc/pmap_rmt.c            │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-26458                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2024-26461      │          │              │                         │                  │ krb5: Memory leak at /krb5/src/lib/gssapi/krb5/k5sealv3.c    │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-26461                   │
├────────────────────┼─────────────────────┤          │              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ libheif1           │ CVE-2023-49463      │          │              │ 1.15.1-1+deb12u1        │                  │ libheif v1.17.5 was discovered to contain a segmentation     │
│                    │                     │          │              │                         │                  │ violation via ...                                            │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-49463                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2024-25269      │          │              │                         │                  │ libheif <= 1.17.6 contains a memory leak in the function     │
│                    │                     │          │              │                         │                  │ JpegEncoder:: ......                                         │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-25269                   │
├────────────────────┼─────────────────────┤          │              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ libjbig0           │ CVE-2017-9937       │          │              │ 2.1-6.1                 │                  │ libtiff: memory malloc failure in tif_jbig.c could cause     │
│                    │                     │          │              │                         │                  │ DOS.                                                         │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2017-9937                    │
├────────────────────┼─────────────────────┼──────────┤              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ libk5crypto3       │ CVE-2024-26462      │ MEDIUM   │              │ 1.20.1-2+deb12u2        │                  │ krb5: Memory leak at /krb5/src/kdc/ndr.c                     │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-26462                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2025-24528      │          │              │                         │                  │ krb5: overflow when calculating ulog block size              │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2025-24528                   │
│                    ├─────────────────────┼──────────┤              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2018-5709       │ LOW      │              │                         │                  │ krb5: integer overflow in dbentry->n_key_data in             │
│                    │                     │          │              │                         │                  │ kadmin/dbutil/dump.c                                         │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2018-5709                    │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2024-26458      │          │              │                         │                  │ krb5: Memory leak at /krb5/src/lib/rpc/pmap_rmt.c            │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-26458                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2024-26461      │          │              │                         │                  │ krb5: Memory leak at /krb5/src/lib/gssapi/krb5/k5sealv3.c    │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-26461                   │
├────────────────────┼─────────────────────┼──────────┤              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│ libkrb5-3          │ CVE-2024-26462      │ MEDIUM   │              │                         │                  │ krb5: Memory leak at /krb5/src/kdc/ndr.c                     │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-26462                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2025-24528      │          │              │                         │                  │ krb5: overflow when calculating ulog block size              │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2025-24528                   │
│                    ├─────────────────────┼──────────┤              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2018-5709       │ LOW      │              │                         │                  │ krb5: integer overflow in dbentry->n_key_data in             │
│                    │                     │          │              │                         │                  │ kadmin/dbutil/dump.c                                         │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2018-5709                    │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2024-26458      │          │              │                         │                  │ krb5: Memory leak at /krb5/src/lib/rpc/pmap_rmt.c            │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-26458                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2024-26461      │          │              │                         │                  │ krb5: Memory leak at /krb5/src/lib/gssapi/krb5/k5sealv3.c    │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-26461                   │
├────────────────────┼─────────────────────┼──────────┤              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│ libkrb5support0    │ CVE-2024-26462      │ MEDIUM   │              │                         │                  │ krb5: Memory leak at /krb5/src/kdc/ndr.c                     │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-26462                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2025-24528      │          │              │                         │                  │ krb5: overflow when calculating ulog block size              │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2025-24528                   │
│                    ├─────────────────────┼──────────┤              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2018-5709       │ LOW      │              │                         │                  │ krb5: integer overflow in dbentry->n_key_data in             │
│                    │                     │          │              │                         │                  │ kadmin/dbutil/dump.c                                         │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2018-5709                    │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2024-26458      │          │              │                         │                  │ krb5: Memory leak at /krb5/src/lib/rpc/pmap_rmt.c            │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-26458                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2024-26461      │          │              │                         │                  │ krb5: Memory leak at /krb5/src/lib/gssapi/krb5/k5sealv3.c    │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-26461                   │
├────────────────────┼─────────────────────┼──────────┤              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ libldap-2.5-0      │ CVE-2023-2953       │ HIGH     │              │ 2.5.13+dfsg-5           │                  │ openldap: null pointer dereference in ber_memalloc_x         │
│                    │                     │          │              │                         │                  │ function                                                     │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-2953                    │
│                    ├─────────────────────┼──────────┤              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2015-3276       │ LOW      │              │                         │                  │ openldap: incorrect multi-keyword mode cipherstring parsing  │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2015-3276                    │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2017-14159      │          │              │                         │                  │ openldap: Privilege escalation via PID file manipulation     │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2017-14159                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2017-17740      │          │              │                         │                  │ openldap: contrib/slapd-modules/nops/nops.c attempts to free │
│                    │                     │          │              │                         │                  │ stack buffer allowing remote attackers to cause...           │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2017-17740                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2020-15719      │          │              │                         │                  │ openldap: Certificate validation incorrectly matches name    │
│                    │                     │          │              │                         │                  │ against CN-ID                                                │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2020-15719                   │
├────────────────────┼─────────────────────┤          │              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ libmount1          │ CVE-2022-0563       │          │              │ 2.38.1-5+deb12u3        │                  │ util-linux: partial disclosure of arbitrary files in chfn    │
│                    │                     │          │              │                         │                  │ and chsh when compiled...                                    │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2022-0563                    │
├────────────────────┼─────────────────────┼──────────┤              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ libpam-modules     │ CVE-2024-10041      │ MEDIUM   │              │ 1.5.2-6+deb12u1         │                  │ pam: libpam: Libpam vulnerable to read hashed password       │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-10041                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2024-22365      │          │              │                         │                  │ pam: allowing unprivileged user to block another user        │
│                    │                     │          │              │                         │                  │ namespace                                                    │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-22365                   │
├────────────────────┼─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│ libpam-modules-bin │ CVE-2024-10041      │          │              │                         │                  │ pam: libpam: Libpam vulnerable to read hashed password       │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-10041                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2024-22365      │          │              │                         │                  │ pam: allowing unprivileged user to block another user        │
│                    │                     │          │              │                         │                  │ namespace                                                    │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-22365                   │
├────────────────────┼─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│ libpam-runtime     │ CVE-2024-10041      │          │              │                         │                  │ pam: libpam: Libpam vulnerable to read hashed password       │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-10041                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2024-22365      │          │              │                         │                  │ pam: allowing unprivileged user to block another user        │
│                    │                     │          │              │                         │                  │ namespace                                                    │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-22365                   │
├────────────────────┼─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│ libpam0g           │ CVE-2024-10041      │          │              │                         │                  │ pam: libpam: Libpam vulnerable to read hashed password       │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-10041                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2024-22365      │          │              │                         │                  │ pam: allowing unprivileged user to block another user        │
│                    │                     │          │              │                         │                  │ namespace                                                    │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-22365                   │
├────────────────────┼─────────────────────┼──────────┤              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ libpng16-16        │ CVE-2021-4214       │ LOW      │              │ 1.6.39-2                │                  │ libpng: hardcoded value leads to heap-overflow               │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2021-4214                    │
├────────────────────┼─────────────────────┤          │              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ libsmartcols1      │ CVE-2022-0563       │          │              │ 2.38.1-5+deb12u3        │                  │ util-linux: partial disclosure of arbitrary files in chfn    │
│                    │                     │          │              │                         │                  │ and chsh when compiled...                                    │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2022-0563                    │
├────────────────────┼─────────────────────┼──────────┤              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ libssl3            │ CVE-2024-13176      │ MEDIUM   │              │ 3.0.15-1~deb12u1        │                  │ openssl: Timing side-channel in ECDSA signature computation  │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-13176                   │
├────────────────────┼─────────────────────┼──────────┤              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ libstdc++6         │ CVE-2022-27943      │ LOW      │              │ 12.2.0-14               │                  │ binutils: libiberty/rust-demangle.c in GNU GCC 11.2 allows   │
│                    │                     │          │              │                         │                  │ stack exhaustion in demangle_const                           │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2022-27943                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2023-4039       │          │              │                         │                  │ gcc: -fstack-protector fails to guard dynamic stack          │
│                    │                     │          │              │                         │                  │ allocations on ARM64                                         │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-4039                    │
├────────────────────┼─────────────────────┤          │              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ libsystemd0        │ CVE-2013-4392       │          │              │ 252.36-1~deb12u1        │                  │ systemd: TOCTOU race condition when updating file            │
│                    │                     │          │              │                         │                  │ permissions and SELinux security contexts...                 │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2013-4392                    │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2023-31437      │          │              │                         │                  │ An issue was discovered in systemd 253. An attacker can      │
│                    │                     │          │              │                         │                  │ modify a...                                                  │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-31437                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2023-31438      │          │              │                         │                  │ An issue was discovered in systemd 253. An attacker can      │
│                    │                     │          │              │                         │                  │ truncate a...                                                │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-31438                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2023-31439      │          │              │                         │                  │ An issue was discovered in systemd 253. An attacker can      │
│                    │                     │          │              │                         │                  │ modify the...                                                │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-31439                   │
├────────────────────┼─────────────────────┼──────────┼──────────────┼─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ libtiff6           │ CVE-2023-52355      │ HIGH     │ will_not_fix │ 4.5.0-6+deb12u2         │                  │ libtiff: TIFFRasterScanlineSize64 produce too-big size and   │
│                    │                     │          │              │                         │                  │ could cause OOM                                              │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-52355                   │
│                    ├─────────────────────┼──────────┤              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2023-6277       │ MEDIUM   │              │                         │                  │ libtiff: Out-of-memory in TIFFOpen via a craft file          │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-6277                    │
│                    ├─────────────────────┼──────────┼──────────────┤                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2017-16232      │ LOW      │ affected     │                         │                  │ libtiff: Memory leaks in tif_open.c, tif_lzw.c, and          │
│                    │                     │          │              │                         │                  │ tif_aux.c                                                    │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2017-16232                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2017-17973      │          │              │                         │                  │ libtiff: heap-based use after free in                        │
│                    │                     │          │              │                         │                  │ tiff2pdf.c:t2p_writeproc                                     │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2017-17973                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2017-5563       │          │              │                         │                  │ libtiff: Heap-buffer overflow in LZWEncode tif_lzw.c         │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2017-5563                    │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2017-9117       │          │              │                         │                  │ libtiff: Heap-based buffer over-read in bmp2tiff             │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2017-9117                    │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2018-10126      │          │              │                         │                  │ libtiff: NULL pointer dereference in the jpeg_fdct_16x16     │
│                    │                     │          │              │                         │                  │ function in jfdctint.c                                       │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2018-10126                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2022-1210       │          │              │                         │                  │ tiff: Malicious file leads to a denial of service in TIFF    │
│                    │                     │          │              │                         │                  │ File...                                                      │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2022-1210                    │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2023-1916       │          │              │                         │                  │ libtiff: out-of-bounds read in extractImageSection() in      │
│                    │                     │          │              │                         │                  │ tools/tiffcrop.c                                             │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-1916                    │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2023-3164       │          │              │                         │                  │ libtiff: heap-buffer-overflow in extractImageSection()       │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-3164                    │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2023-6228       │          │              │                         │                  │ libtiff: heap-based buffer overflow in cpStripToTile() in    │
│                    │                     │          │              │                         │                  │ tools/tiffcp.c                                               │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-6228                    │
├────────────────────┼─────────────────────┼──────────┤              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ libtinfo6          │ CVE-2023-50495      │ MEDIUM   │              │ 6.4-4                   │                  │ ncurses: segmentation fault via _nc_wrap_entry()             │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-50495                   │
├────────────────────┼─────────────────────┼──────────┤              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ libudev1           │ CVE-2013-4392       │ LOW      │              │ 252.36-1~deb12u1        │                  │ systemd: TOCTOU race condition when updating file            │
│                    │                     │          │              │                         │                  │ permissions and SELinux security contexts...                 │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2013-4392                    │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2023-31437      │          │              │                         │                  │ An issue was discovered in systemd 253. An attacker can      │
│                    │                     │          │              │                         │                  │ modify a...                                                  │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-31437                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2023-31438      │          │              │                         │                  │ An issue was discovered in systemd 253. An attacker can      │
│                    │                     │          │              │                         │                  │ truncate a...                                                │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-31438                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2023-31439      │          │              │                         │                  │ An issue was discovered in systemd 253. An attacker can      │
│                    │                     │          │              │                         │                  │ modify the...                                                │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-31439                   │
├────────────────────┼─────────────────────┤          │              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ libuuid1           │ CVE-2022-0563       │          │              │ 2.38.1-5+deb12u3        │                  │ util-linux: partial disclosure of arbitrary files in chfn    │
│                    │                     │          │              │                         │                  │ and chsh when compiled...                                    │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2022-0563                    │
├────────────────────┼─────────────────────┼──────────┤              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ libxml2            │ CVE-2024-25062      │ HIGH     │              │ 2.9.14+dfsg-1.3~deb12u1 │                  │ libxml2: use-after-free in XMLReader                         │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-25062                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2024-56171      │          │              │                         │                  │ libxml2: Use-After-Free in libxml2                           │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-56171                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2025-24928      │          │              │                         │                  │ libxml2: Stack-based buffer overflow in xmlSnprintfElements  │
│                    │                     │          │              │                         │                  │ of libxml2                                                   │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2025-24928                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2025-27113      │          │              │                         │                  │ libxml2: NULL Pointer Dereference in libxml2 xmlPatMatch     │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2025-27113                   │
│                    ├─────────────────────┼──────────┤              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2022-49043      │ MEDIUM   │              │                         │                  │ libxml: use-after-free in xmlXIncludeAddNode                 │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2022-49043                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2023-39615      │          │              │                         │                  │ libxml2: crafted xml can cause global buffer overflow        │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-39615                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2023-45322      │          │              │                         │                  │ libxml2: use-after-free in xmlUnlinkNode() in tree.c         │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-45322                   │
│                    ├─────────────────────┼──────────┤              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2024-34459      │ LOW      │              │                         │                  │ libxml2: buffer over-read in xmlHTMLPrintFileContext in      │
│                    │                     │          │              │                         │                  │ xmllint.c                                                    │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-34459                   │
├────────────────────┼─────────────────────┼──────────┼──────────────┼─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ libxslt1.1         │ CVE-2024-55549      │ HIGH     │ fixed        │ 1.1.35-1                │ 1.1.35-1+deb12u1 │ libxslt: Use-After-Free in libxslt (xsltGetInheritedNsList)  │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-55549                   │
│                    ├─────────────────────┤          │              │                         │                  ├──────────────────────────────────────────────────────────────┤
│                    │ CVE-2025-24855      │          │              │                         │                  │ libxslt: Use-After-Free in libxslt numbers.c                 │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2025-24855                   │
│                    ├─────────────────────┼──────────┼──────────────┤                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2015-9019       │ LOW      │ affected     │                         │                  │ libxslt: math.random() in xslt uses unseeded randomness      │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2015-9019                    │
├────────────────────┼─────────────────────┼──────────┤              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ login              │ CVE-2023-4641       │ MEDIUM   │              │ 1:4.13+dfsg1-1+b1       │                  │ shadow-utils: possible password leak during passwd(1) change │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-4641                    │
│                    ├─────────────────────┼──────────┤              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2007-5686       │ LOW      │              │                         │                  │ initscripts in rPath Linux 1 sets insecure permissions for   │
│                    │                     │          │              │                         │                  │ the /var/lo ......                                           │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2007-5686                    │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2023-29383      │          │              │                         │                  │ shadow: Improper input validation in shadow-utils package    │
│                    │                     │          │              │                         │                  │ utility chfn                                                 │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-29383                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2024-56433      │          │              │                         │                  │ shadow-utils: Default subordinate ID configuration in        │
│                    │                     │          │              │                         │                  │ /etc/login.defs could lead to compromise                     │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-56433                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ TEMP-0628843-DBAD28 │          │              │                         │                  │ [more related to CVE-2005-4890]                              │
│                    │                     │          │              │                         │                  │ https://security-tracker.debian.org/tracker/TEMP-0628843-DB- │
│                    │                     │          │              │                         │                  │ AD28                                                         │
├────────────────────┼─────────────────────┤          │              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ mount              │ CVE-2022-0563       │          │              │ 2.38.1-5+deb12u3        │                  │ util-linux: partial disclosure of arbitrary files in chfn    │
│                    │                     │          │              │                         │                  │ and chsh when compiled...                                    │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2022-0563                    │
├────────────────────┼─────────────────────┼──────────┤              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ ncurses-base       │ CVE-2023-50495      │ MEDIUM   │              │ 6.4-4                   │                  │ ncurses: segmentation fault via _nc_wrap_entry()             │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-50495                   │
├────────────────────┤                     │          │              │                         ├──────────────────┤                                                              │
│ ncurses-bin        │                     │          │              │                         │                  │                                                              │
│                    │                     │          │              │                         │                  │                                                              │
├────────────────────┼─────────────────────┤          │              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ nginx              │ CVE-2024-7347       │          │              │ 1.27.4-1~bookworm       │                  │ nginx: specially crafted MP4 file may cause denial of        │
│                    │                     │          │              │                         │                  │ service                                                      │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-7347                    │
│                    ├─────────────────────┼──────────┤              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2009-4487       │ LOW      │              │                         │                  │ nginx: Absent sanitation of escape sequences in web server   │
│                    │                     │          │              │                         │                  │ log                                                          │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2009-4487                    │
│                    ├─────────────────────┤          ├──────────────┤                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2013-0337       │          │ will_not_fix │                         │                  │ The default configuration of nginx, possibly 1.3.13 and      │
│                    │                     │          │              │                         │                  │ earlier, uses ......                                         │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2013-0337                    │
│                    ├─────────────────────┤          ├──────────────┤                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2023-44487      │          │ affected     │                         │                  │ HTTP/2: Multiple HTTP/2 enabled web servers are vulnerable   │
│                    │                     │          │              │                         │                  │ to a DDoS attack...                                          │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-44487                   │
├────────────────────┼─────────────────────┼──────────┤              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ openssl            │ CVE-2024-13176      │ MEDIUM   │              │ 3.0.15-1~deb12u1        │                  │ openssl: Timing side-channel in ECDSA signature computation  │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-13176                   │
├────────────────────┼─────────────────────┤          │              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ passwd             │ CVE-2023-4641       │          │              │ 1:4.13+dfsg1-1+b1       │                  │ shadow-utils: possible password leak during passwd(1) change │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-4641                    │
│                    ├─────────────────────┼──────────┤              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2007-5686       │ LOW      │              │                         │                  │ initscripts in rPath Linux 1 sets insecure permissions for   │
│                    │                     │          │              │                         │                  │ the /var/lo ......                                           │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2007-5686                    │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2023-29383      │          │              │                         │                  │ shadow: Improper input validation in shadow-utils package    │
│                    │                     │          │              │                         │                  │ utility chfn                                                 │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-29383                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2024-56433      │          │              │                         │                  │ shadow-utils: Default subordinate ID configuration in        │
│                    │                     │          │              │                         │                  │ /etc/login.defs could lead to compromise                     │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2024-56433                   │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ TEMP-0628843-DBAD28 │          │              │                         │                  │ [more related to CVE-2005-4890]                              │
│                    │                     │          │              │                         │                  │ https://security-tracker.debian.org/tracker/TEMP-0628843-DB- │
│                    │                     │          │              │                         │                  │ AD28                                                         │
├────────────────────┼─────────────────────┼──────────┤              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ perl-base          │ CVE-2023-31484      │ HIGH     │              │ 5.36.0-7+deb12u1        │                  │ perl: CPAN.pm does not verify TLS certificates when          │
│                    │                     │          │              │                         │                  │ downloading distributions over HTTPS...                      │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-31484                   │
│                    ├─────────────────────┼──────────┤              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2011-4116       │ LOW      │              │                         │                  │ perl: File:: Temp insecure temporary file handling           │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2011-4116                    │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2023-31486      │          │              │                         │                  │ http-tiny: insecure TLS cert default                         │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-31486                   │
├────────────────────┼─────────────────────┤          │              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ sysvinit-utils     │ TEMP-0517018-A83CE6 │          │              │ 3.06-4                  │                  │ [sysvinit: no-root option in expert installer exposes        │
│                    │                     │          │              │                         │                  │ locally exploitable security flaw]                           │
│                    │                     │          │              │                         │                  │ https://security-tracker.debian.org/tracker/TEMP-0517018-A8- │
│                    │                     │          │              │                         │                  │ 3CE6                                                         │
├────────────────────┼─────────────────────┤          │              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ tar                │ CVE-2005-2541       │          │              │ 1.34+dfsg-1.2+deb12u1   │                  │ tar: does not properly warn the user when extracting setuid  │
│                    │                     │          │              │                         │                  │ or setgid...                                                 │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2005-2541                    │
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤
│                    │ TEMP-0290435-0B57B5 │          │              │                         │                  │ [tar's rmt command may have undesired side effects]          │
│                    │                     │          │              │                         │                  │ https://security-tracker.debian.org/tracker/TEMP-0290435-0B- │
│                    │                     │          │              │                         │                  │ 57B5                                                         │
├────────────────────┼─────────────────────┤          │              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ util-linux         │ CVE-2022-0563       │          │              │ 2.38.1-5+deb12u3        │                  │ util-linux: partial disclosure of arbitrary files in chfn    │
│                    │                     │          │              │                         │                  │ and chsh when compiled...                                    │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2022-0563                    │
├────────────────────┤                     │          │              │                         ├──────────────────┤                                                              │
│ util-linux-extra   │                     │          │              │                         │                  │                                                              │
│                    │                     │          │              │                         │                  │                                                              │
│                    │                     │          │              │                         │                  │                                                              │
├────────────────────┼─────────────────────┼──────────┼──────────────┼─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤
│ zlib1g             │ CVE-2023-45853      │ CRITICAL │ will_not_fix │ 1:1.2.13.dfsg-1         │                  │ zlib: integer overflow and resultant heap-based buffer       │
│                    │                     │          │              │                         │                  │ overflow in zipOpenNewFileInZip4_6                           │
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-45853                   │
└────────────────────┴─────────────────────┴──────────┴──────────────┴─────────────────────────┴──────────────────┴──────────────────────────────────────────────────────────────┘
```

[Sommaire des scans](#2-scan-de-vuln)