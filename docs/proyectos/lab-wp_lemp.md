---
layout: page
parent: DevOps
title: LAB - WP + LEMP
permalink: /lab-wp_lemp/
---

# BASIC: nginx, lemp, certbot, wordpress

- [Video PeladoNerd](https://www.youtube.com/watch?v=_LQv96MdtCk)
- [Doc DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-host-a-website-using-cloudflare-and-nginx-on-ubuntu-18-04)


## Servidor web

Servicio que expone puertos (generalmente 80 http / 443 https) y levanta código html debidamente configurado
+ php, ruby, python necesitan plugins


### SSH

```bash
$ ssh -p 2102 -i /.ssh/id_rsa plataformas@173.230.136.84
```
173.230.136.84--> Welcome to nginx!


## NGINX

/etc = configuraciones + servicio
$ cd /etc/nginx/
(...) sites-enabled sites-available

$ cd /etc/nginx/sites-available/
$ sudo cp default test2.arbusta.net
$ sudo cp default prueba.arbusta.net

$ sudo nano test2.arbusta.net
	server {
		listen 80; #quitar default_server (sólo un config de nginx puede ser default_server)
        listen [::]:80;
		root /var/www/test;
		server_name test.arbusta.net;
$ sudo mc
	File > Relative symlink (available to enabled)
$ sudo nginx -t
$ sudo nginx -s reload
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

## Cloudflare

- DNS: crear registros A de cada sitio / subdominio
+ puede editarse el archivo host local para "simular" los DNS ["IP" prueba.arbusta.net test2.arbustanet]

### sitio

- como no existe aun la carpeta root --> error 404
$ sudo mkdir /var/www/test2
$ sudo mkdir /var/www/prueba
$ sudo nano /var/www/test2/index.html #"Hola Mundo"for testing purposes


Si es multisitio y existe la carpeta pero no encuentra contenido --> error 403

### LEMP
- LAMP = apache
- LEMP = "e"nginx
+ php-fpm = para que nginx "interprete" php como web

$ sudo apt-get install php-fpm

$ sudo nano /etc/nginx/sites-available/test2.arbusta.net

Por defecto, en Ubuntu php-fpm no usa puerto tcp por eso usamos socket
# pass PHP scripts to FastCGI server
        #
        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
        #
        #       # With php-fpm (or other unix sockets):
                fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
        #       # With php-cgi (or other tcp sockets):
        #       fastcgi_pass 127.0.0.1:9000;
        }

$ sudo nginx -t
$ sudo systemctl reload nginx
$ sudo systemctl status nginx.service

#### PHP

$ sudo nano /var/www/test2/info.php
http://test2.arbusta.net/info.php
PHP Version 7.4.3

Chequear versión de fpm:
- archivo nginx
# With php-fpm (or other unix sockets):
                fastcgi_pass unix:/var/run/php/php7.4-fpm.sock

$ ls /var/run/php/php7.4-fpm.sock 
/var/run/php/php7.4-fpm.sock

- errores 500 = configuración del server (no del cliente)
$ sudo tail -f /var/log/nginx/error.log


Si por ejemplo vuelvo a comentar las líneas donde indico a nginx dónde y cómo interpretar php, al ingresar a la URL anterior en lugar de mostrar la info, me descarga el archivo info.php


### mysql

$ sudo apt-get install mysql-server php-mysql
https://www.phpmyadmin.net/
$ sudo wget https://files.phpmyadmin.net/phpMyAdmin/5.0.2/phpMyAdmin-5.0.2-all-languages.zip
$ sudo unzip php...
$ sudo mv phpMyAdmin-5.0.2-all-languages phpmyadmin

http://test2.arbusta.net/phpmyadmin --> error 403 Forbidden
$ sudo tail -f /var/log/nginx/error.log
directory index of "/var/www/test2/phpmyadmin/" is forbidden

$ sudo nano /etc/nginx/sites-available/test2.arbusta.net
# Add index.php to the list if you are using PHP
        index index.php index.html index.htm index.nginx-debian.html;

http://test2.arbusta.net/phpmyadmin/

### secure mysql

$ sudo mysqladmin --user=root password "xEtYHm8H6ngdQRs8heg7"
mysqladmin: [Warning] Using a password on the command line interface can be insecure.
Warning: Since password will be sent to server in plain text, use ssl connection to ensure password safety.
$ sudo mysql -u root -p

http://test2.arbusta.net/phpmyadmin/
x mysqli::real_connect(): (HY000/1698): Access denied for user 'root'@'localhost'

mysql> CREATE USER 'phpmyadmin'@'localhost' IDENTIFIED BY '..phpmyadmin2020..';
Query OK, 0 rows affected (0.01 sec)

mysql> GRANT ALL PRIVILEGES ON *.* TO 'phpmyadmin'@'localhost' WITH GRANT OPTION;
Query OK, 0 rows affected (0.01 sec)

http://test2.arbusta.net/phpmyadmin/
phpmyadmin
..phpmyadmin2020..
![image](https://i.imgur.com/FblLjg4.png "php login")


## CERTBOT (DNS públicos, 80 / 443)

http://test2.arbusta.net/
http://prueba.arbusta.net/

Certbot: https://certbot.eff.org/

Preparar instalación certbot

sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository universe
sudo apt-get update

Instalación certbot
sudo apt-get install certbot python3-certbot-nginx

Correr certbot
sudo certbot --nginx
	aunque es automático, se recomienda dejar mail
	seleccionar él o los dominios
	redirect

Congratulations! You have successfully enabled https://prueba.arbusta.net

You should test your configuration at:
https://www.ssllabs.com/ssltest/analyze.html?d=prueba.arbusta.net
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/prueba.arbusta.net/fullchain.pem


Testear renovación 
$ sudo certbot renew --dry-run
$ cat /etc/cron.d/certbot
0 */12 * * * root test -x /usr/bin/certbot -a \! -d /run/systemd/system && perl -e 'sleep int(rand(43200))' && certbot -q renew

## WORDPRESS

+ php + sql + nginx

# Instalar dependencias 

sudo apt install php-fpm php-common php-mbstring php-xmlrpc php-soap php-gd php-xml php-intl php-mysql php-cli php-mcrypt php-ldap php-zip php-curl

# Crear DB

mysql -u root -p

CREATE DATABASE wordpress_site;
CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY 'VGip6NLXF7LvMrJ2MmXX';
GRANT ALL PRIVILEGES ON wordpress_site . * TO 'wordpressuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;

# Bajar e instalar wordpress

wget https://wordpress.org/latest.tar.gz
tar -zxvf latest.tar.gz
sudo mv wordpress/* /var/www/test2
chown -R www-data:www-data /var/www/test2
sudo find /var/www/test2 -type d -exec chmod 0755 {} \;
sudo find /var/www/test2 -type f -exec chmod 0644 {} \;

# Configurar wordpress

cd /var/www/test2
sudo mv wp-config-sample.php wp-config.php
sudo nano wp-config.php
(Configurar DB_NAME, DB_USER, DB_PASSWORD, DB_HOST)

https://test2.arbusta.net/
![wp install](https://i.imgur.com/db0UyNL.png "wp install")

admin
WSSMzP^Laj1WIDA%#t

![escritorio wp](https://i.imgur.com/yRbOV5D.png)


## Setup firewall

```bash
$ ufw status
Status: inactive
$ ufw allow http
$ ufw allow https
$ ufw allow 2102
$ ufw enable
```

## Server Proxy (nginx)

Creamos 2 servers de backend (corriendo una app) "detrás" del nginx
- cualquier servicio http
- que nginx llegue a estas IP (interna o externa)

nginx test2.arbusta.net
	--> app1:IP:8080
	--> app2:IP:8080

nginx recibe request del browser y redirige en lugar de procesar

Ejemplo peladonerd, con apps que devuelven los headers
```bash
$ nano /etc/nginx/sites-enabled/test2.arbusta.net
location /app {
                proxy_pass http://ip-app1:8080;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
        }
```
location /app {
                proxy_pass https://www.google.com.ar;
Le esta pasando a google la url google.com/app

## UPSTREAM

- set de servidores
- para balancear el tráfico entre servidores

upstream backend {
  server x.x.x.x;
  
  
el upstream lo puedes colocar en el mismo archivo, para que no toques el config seria:  upstream backend { server ip:port } server { #server config }, asi seria mas facil de administrar en el mismo archivo, si te das cuenta el http hace un include de los archivos de configuracion y todo queda dentro de http {}

$ sudo nano /etc/nginx/nginx.conf
	- el bloque upstream debe estar en el bloque http (no server, como se configura en el virtual host con nginx de sites)

upstream <nombre> {
	server ip1:8080;
	server ip28080;
	}

en el vhost nginx:
location /app {
                proxy_pass https://<nombre>;

Para ver el tráfico (al ejecutar, refrescar el navegador):
$ tcpdump -i any -nn port 8080

https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/

Para balancear manualmente (5xrequest a servidor 1)
upstream <nombre> {
	server ip1:8080 weight=5;




# Instalar WordPress con LEMP en Ubuntu 20.04

- [Tutorial How To Install Linux, Nginx, MySQL, PHP (LEMP stack) on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-ubuntu-20-04)
- [Tutorial How to Install WordPress with LEMP on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-with-lemp-on-ubuntu-20-04)

## Server + DNS

- server: Ubuntu 20.04
- IP: 104.131.114.139
- hostname: pla-multisite-wp
- SSH Host: multisite
- DNS: lalibrecf.com.ar, staging.lacolifata.com.ar

## Preparación Server

- ver Cheatsheet - Alta de Servidores

## Instalación LEMP + dependencias

```bash
# instalar webserver
$ sudo apt install nginx
# instalar y configurar mysql
$ sudo apt install mysql-server
$ sudo mysql_secure_installation
# instalar extensiones php
$ sudo apt install php-fpm php-mysql
$ sudo apt install php-curl php-gd php-intl php-mbstring php-soap php-xml php-xmlrpc php-zip
# carpetas y permisos
$ sudo mkdir /var/www/lalibrecf
$ sudo mkdir /var/www/lacolifata
$ sudo chown -R $USER:$USER /var/www/lalibrecf/
$ sudo chown -R $USER:$USER /var/www/lacolifata/
# configuración webserver
$ sudo nano /etc/nginx/sites-available/lalibrecf
$ sudo nano /etc/nginx/sites-available/lacolifata
$ sudo ln -s /etc/nginx/sites-available/lalibrecf /etc/nginx/sites-enabled/
$ sudo ln -s /etc/nginx/sites-available/lacolifata /etc/nginx/sites-enabled/
$ sudo nginx -t
$ sudo systemctl reload nginx
# test webservers
$ nano /var/www/lalibrecf/index.html
$ nano /var/www/lacolifata/index.html
# test php + nginx
$ nano /var/www/lalibrecf/info.php
$ rm /var/www/lalibrecf/info.php
```

## HTTPS con Certbot

- [Guía](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-20-04)

Para instalar y configurar los certificados SSL

```bash
# Obviamente depende de los sitios a deployar:
$ sudo certbot --nginx -d lalibrecf.com.ar -d www.lalibrecf.com.ar
$ sudo certbot --nginx -d staging.lacolifata.com.ar -d www.staging.lacolifata.com.ar
```

## BD y Usuarios BD mysql

Configuro los users y BD para los sitios de La Libre (versión WP) y La Colifata

```mysql
> CREATE DATABASE lalibre DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
> CREATE DATABASE lacolifata DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
> CREATE USER 'lacolifatadbuser'@'localhost' IDENTIFIED BY 'acáhayunacontraseña';
> GRANT ALL ON lacolifata.* TO 'lacolifatadbuser'@'localhost';
> CREATE USER 'lalibredbuser'@'localhost' IDENTIFIED BY 'acáhayunacontraseña';
> GRANT ALL ON lalibre.* TO 'lalibredbuser'@'localhost';
```

## Instalar Wordpress

```bash
$ cd /tmp
# Descargo y descomprimo WP
$ curl -LO https://wordpress.org/latest.tar.gz
$ tar xzvf latest.tar.gz
# Copio el archivo config de WP
$ cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php
# Copio WP, por ejemplo para la carpeta del sitio de La Colifata
$ sudo cp -a /tmp/wordpress/. /var/www/lacolifata
$ sudo chown -R www-data:www-data /var/www/lacolifata
# WordPress secret key generator, para incluir en wp-config (completar db_name, db_user, db_pass)
curl -s https://api.wordpress.org/secret-key/1.1/salt/
```

Por último se completa la instalación desde el admin web, salvo que el sitio ya exista y corresponda hacer una migración o levantar un backup.

## Migración La Colifata

El sitio estaba deployado usando [easyengine](https://easyengine.io/), pero no me pude apropiar 100% de la herramienta así que opté por la versión old school con LEMP. Empiezo con backup WP + BD + snapshot (lacolifatawp-easyengine20210226).

- Backup con [script en easyengine](https://community.easyengine.io/t/backup-strategies/12014/2)
- Copio files + BD al servidor de destino

```bash
ssh multisite
# Copio el backup completo al nuevo servidor y descomprimo
sudo cp /home/diego/html.tar.xz /var/www/lacolifata
sudo tar -xf /var/www/lacolifata/html.tar.xz
# Chequeo que estén OK las variables de WP (db_user, db_name, db_pass, db_host)
sudo nano wp-config.php
sudo chown -R www-data:www-data /var/www/lacolifata
sudo systemctl reload nginx
```

### Migración BD mysql

- [Guía](https://www.digitalocean.com/community/tutorials/how-to-import-and-export-databases-in-mysql-or-mariadb)

```bash
# Primero importo / reemplazo la BD existente (porque ya la había migrado como test antes)
$ mysql -u username -p new_database < data-dump.sql
$ mysql -u lacolifatadbuser -p lacolifata < /var/www/lacolifata/test.lacolifata.com.ar.sql
```

Actualizo tablas, campos y demás en mysql, considerando que estoy migrando desde dominio test a staging:

```mysql
> show databases;
> use lacolifata;
# estos comandos pueden ser redundantes pero me aseguro que se actualice todo
> UPDATE wp_options SET option_value = replace(option_value, 'testing.lacolifata.com.ar', 'staging.lacolifata.com.ar') WHERE option_name = 'home' OR option_name = 'siteurl';UPDATE wp_posts SET guid = replace(guid, 'testing.lacolifata.com.ar','staging.lacolifata.com.ar');UPDATE wp_posts SET post_content = replace(post_content, 'testing.lacolifata.com.ar', 'staging.lacolifata.com.ar');
> UPDATE wp_postmeta SET meta_value = replace(meta_value,'testing.lacolifata.com.ar','staging.lacolifata.com.ar');
> SELECT * from wp_options WHERE option_name = 'home' OR option_name = 'siteurl';
> UPDATE wp_options SET option_value = "https://staging.lacolifata.com.ar" WHERE option_name = 'home' OR option_name = 'siteurl';
```
