## Nginx Docker Compose

Very simple Nginx projects with docker compose exposing an nginx server running on port 80 (HTTP) with redirect to 443 (HTTPs) with self signed certificates.

The following folders are important

- certs : contains the self signed certificates
- conf.d : contains our nginx config
- html : contains our html files

## SSL

To create the self-signed certificates : 

```
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes -subj "/C=US/O=Nginx/OU=Nginx Github Test/CN=my-tst-playground.com"

openssl pkcs12 -export -in cert.pem -inkey key.pem -out cert_key.pfx
```

## Nginx config

```
server {
    listen 80;
    server_name my-tst-playground.com;
    location / {
        return 301 https://$host$request_uri;
    }    
}
server {
    ssl_certificate /etc/certs/cert.pem;
    ssl_certificate_key /etc/certs/key.pem;    
    listen 443 ssl;
    server_name my-tst-playground.com;
    

    location / {
        root /usr/share/nginx/html;
        try_files $uri /index.html;
    }
}
```

As you can see we have 

- 2 server definitions (http and https) 
- a redirect from http to https
- our self signed certificates referenced
- a simple index.html


## Docker compose

The docker compose file is reaally simple:

```
version: '3'

services:
  nginx:
    image: nginx:latest
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./conf.d:/etc/nginx/conf.d
      - ./certs/cert.pem:/etc/certs/cert.pem
      - ./certs/key.pem:/etc/certs/key.pem
      - ./html:/usr/share/nginx/html

```