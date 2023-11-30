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

## Migración a VPS
En primer lugar, prepararemos el servidor VPS (de ahora en adelante `cerebelum`) con una pila LEMP. Instalamos los siguientes paquetes: `apt install nginx mariadb-server php php-mysql php-fpm php-xml`

Una vez instalados esos paquetes, exportaremos las bases de datos de nuestro servidor local. Esto lo hacemos con el comando `mysqldump -u root -p drupal > migraciondrupal.sql` como root, y de la misma forma con Nextcloud.

Una vez exportado, con el comando `scp` pasaremos este archivo a `cerebelum`, nuestro servidor VPS. Lo hacemos con el siguiente comando: `scp migraciondrupal.sql miguel@cerebelum.miguelfigueroa.es:/home/miguel`, igual para el archivo de la base de datos con Nextcloud.

Una vez ya en el VPS, accedemos a mariadb y creamos una nueva base de datos con mismo nombre. Entramos en ella y con el comando `source migraciondrupal.sql` importamos toda la información. Crearemos ahora de igual manera que en el servidor local un usuario para cada base de datos, con mismo usuario y contraseña.

Creamos el virtualhost en nginx. Importante descomentar las siguientes líneas:
```
location ~ \.php$ {
    include snippets/fastcgi-php.conf;
    #fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
}
```

Editamos también lo similar a `a2enmod rewrite` de apache, con los siguientes parámetros:
```
        location / {
                try_files $uri $uri/ /index.php?$args;
                # try_files $uri $uri/ =404;
        }
```

Es importante también eliminar los sitios que estén en `sites-enabled` que estén usando el mismo nombre de servidor y puerto.

Una vez configurada la base de datos y el virtual host de Drupal, moveremos la aplicación al VPS de nuevo con el comando `scp`, con la opción `-r`. Comprimiremos las carpetas de la aplicación para ganar tiempo.

Teniendo ya la carpeta de la aplicación en el VPS y colocada en el director `/var/www/html`, nos dirigiremos al fichero `sites/default/setting.php` de Drupal para cambiar la dirección IP de la base de datos, sustituyéndola por `localhost`.

![Alt text](/images/p-7.png)

Aquí observamos la configuración que hicimos en el servidor local. Deberemos establecer el parámetro `'host => 'localhost'`.

Con esto editado, reiniciamos `nginx` y ya tenemos la web de la aplicación Drupal disponible: http://www.miguelfigueroa.es

Repetiríamos el procedimiento con Nextcloud, y lo tenemos funcionando en la siguiente URL: http://cloud.miguelfigueroa.es

## Configuración de HTTPS en el VPS