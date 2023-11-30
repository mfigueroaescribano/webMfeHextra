---
title: Práctica - Instalación/migración de aplicaciones web PHP
type: docs
---
Enunciado de la práctica: [link](https://fp.josedomingo.org/iaw/2_php/practica.html)

## Preparación del escenario
Para la realización de esta práctica he creado dos máquinas con KVM con debian12. Cada una tiene una interfaz NAT con la que salen a internet, y otra enrutada interna de rango `10.0.0.0/24` sólo entre ellas para conseguir una red muy aislada.

Se han establecido la siguientes IPs estáticas para facilitar las conexiones entre ambas máquinas a través de su red interna:

- servidorweb: 10.0.0.10
- servidorbd:  10.0.0.20

## Instalación de un CMS PHP en mi servidor local
### Configuración de apache2 y php
El CMS que instalaremos para esta práctica será **Drupal**. En el servidor web, instalaremos `apache2 php8.2-fpm`, ya que haremos funcionar apache2 con el módulo php-fpm. Configuraremos apache2 para que funcione con este módulo, explicado en [este enlace](http://web.miguelfigueroa.es/docs/iweb/unidad-2-php/taller2/#configuraci%c3%b3n-de-apache2-con-fpm-php). Tras esto, configuraremos un Virtualhost en apache2:

```        
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html/drupal
        ServerName www.miguelfigueroa.org

        <Directory /var/www/html/drupal/>
            Options Indexes FollowSymLinks
            AllowOverride All
            Require all granted
        </Directory>    

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Activamos también el módulo *Rewrite* de apache2, con el comando `a2enmod rewrite`.

Configuramos la resolución estática en nuestro cliente para que se acceda a través de www.miguelfigueroa.org.

![Fichero `/etc/hosts`](/images/p-1.png)

Instalamos también los módulos de PHP requeridos para el CMS que vamos a instalar: `apt install -y php-apcu php-gd php-mbstring php-uploadprogress php-xml php8.2-mysql`

Al acceder a la URL www.miguelfigueroa.org ya nos encontramos con la guía de instalación de Drupal.

![Instalación de Drupal](/images/p-2.png)


### Configuración de la base de datos
Pasamos ahora a configurar la base de datos en el `servidor_bd_miguel`. Instalamos el paquete `mariadb-server` y crearemos una base de datos y usuario para acceder desde otra máquina para Drupal:

```
mysql -u root -p
CREATE DATABASE drupal;
CREATE USER 'drupal'@'%' IDENTIFIED BY 'iesgn';
GRANT ALL PRIVILEGES ON drupal.* TO 'drupal'@'%';
FLUSH PRIVILEGES;
```
Tras esto, editaremos el fichero de configuración de MariaDB (`/etc/mysql/mariadb.conf.d/50-server.cnf`)para habilitar el acceso remoto. Comentaremos la siguiente linea:
```
# bind-address = 127.0.0.1
```

Reiniciamos `mariadb` y ya tendríamos configurada la máquina que alojará la base de datos, y pasamos a introducir esta configuración en el proceso de instalación de Drupal.

### Instalación de Drupal

Si introducimos todas las configuraciones correctamente continuará el proceso de instalación y se completará:

![Inicio de Drupal](/images/p-3.png)

Instalaremos también algún módulo para comprobar el funcionamiento. Para ello, nos vamos a *Inicio > Administrar > Ampliar* y descargamos el módulo de *Captcha* por ejemplo.

![Módulo Captcha](/images/p-4.png)

## Instalación de NextCloud
Descargaremos la [última versión](https://download.nextcloud.com/server/releases/latest.zip) de Nextcloud en el directorio `/var/www/html`.

Descomprimiremos el zip y dejaremos su contenido en la carpeta `nextcloud`. Tras esto, generaremos un nuevo Virtualhost para acceder desde el nombre `cloud.miguelfigueroa.org`.

🤚 Importante recordar que deberemos añadir la dirección `cloud.miguelfigueroa.org` al fichero `/etc/hosts` y confirmar los permisos del director `/var/www/html` al usuario y grupo `www-data`.

Para Nextcloud, requerimos en especial de los módulos de php ` php-zip` y `php-curl`. Tras instalarlos, reiniciamos apache2.

Crearemos también una nueva base de datos en nuestro servidor que está alojando las base de datos, con las siguientes instrucciones:

```
mysql -u root -p
CREATE DATABASE nextcloud;
CREATE USER 'nextcloud'@'%' IDENTIFIED BY 'iesgn';
GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'%';
FLUSH PRIVILEGES;
```
Con esto realizado, accedemos a la URL `www.cloud.miguelfigueroa.org` y ya nos encontramos el proceso de instalación de Nextcloud:

![Instalación de Nextcloud](/images/p-5.png)

Tras introducir los datos de creación del usuario de adminsitrador y la conexióń de base de datos, finalizamos la instalación:

![Finalizada la instalación de Nextcloud](/images/p-6.png)