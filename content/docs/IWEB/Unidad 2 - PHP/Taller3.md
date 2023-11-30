---
title: Taller 3 - Instalación de WordPress en un servidor LEMP
type: docs
---
## Preparación escenario LEMP
La realización de este taller lo haremos con un nuevo contenedor creado en Proxmox, con un debian12. Instalaremos nginx, mariadb-server y php con el módulo fpm-php.

`apt install nginx mariadb-server php php-mysql php8.2-fpm`

### MariaDB
Crearemos ahora una base de datos en MariaDB. Para ello, accedemos `mysql -u root -p` y ejecutamos las siguientes instrucciones:

```
create database wordpress;
use wordpress;
create user 'usuario'@'localhost';
grant all privileges on wordpress.* to 'usuario'@'localhost' identified by 'iesgn'
flush privileges;
```

### Configuración de php-fpm
Desinstalaremos apache2 de la máquina, para evitar posibles conflictos. Nos dirigimos ahora al fichero de configuración de php-fpm `/etc/php/8.2/fpm/pool.d/www.conf` y nos aseguramos que tenemos todos los parámetros correctos (por defecto deberían estarlo), especialmente este: `listen=/run/php/php8.2-fpm.sock`.

## Instalación de Wordpress
Descargamos ahora Wordpress en su última versión desde la página oficial en la ruta `/var/www/html`, con el comando `wget https://es.wordpress.org/latest-es_ES.zip` y extraemos el archivo descargado con unzip.

Cambiamos los permisos a `www-data` para todo el directorio `html`. Ahora, en la ruta `/etc/nginx/sites-availables` creamos un nuevo archivo con copia del `default` indicando los siguientes parámetros:

```
server {
    listen 80;
    listen [::]:80;
    server_name blog.miguelfigueroa.org www.blog.miguelfigueroa.org;
    root /var/www/html/wordpress;
    index index.php;
}
```

Tras esto, lo añadiremos a `sites-enabled` creando un enlace simbólico con el comando `ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled`. Nos aseguramos que en el sites-enabled no haya otro sitio saliendo por el mismo puerto, si no se generará un error.

Recargamos los servicios de php8.2-fpm y nginx y al acceder a la IP de nuestra máquina por el puerto 80 (HTTP) encontramos la página inicial de instalación de Wordpress:

![Configuración inicial de Wordpress](/images/t3-1.png)

En este punto insertaremos la configuración de la nueva base de datos que creamos [más arriba](https://web.miguelfigueroa.es/docs/iweb/unidad-2-php/taller3/#mariadb)

Al introducir los datos correctamente de la base de datos finaliza la instalación, y ya nos lleva a la página de inicio de ejemplo:

![Página de inicio de Wordpress](/images/t3-2.png)

Probamos a crear una nueva entrada en el blog para comprobar que todo está funcionando correctamente:

![Entrada de prueba](/images/t3-3.png)

Para comprobar toda la información del servidor LEMP y que está funcionando correctamente, generamos un archivo en el DocumentRoot de Wordpress con el siguiente contenido: ` <?php phpinfo(); ?>`:

![Wordpress funcionando con fpm-php](/images/t3-4.png)

![Wordpress funcionando con nginx](/images/t3-5.png)

## Configuración de URLs amigables
En el fichero de configuración de Wordpress en nginx ya indicamos las URL para acceder a la página de inicio con el dominio www.blog.miguelfigueroa.org, configurando el fichero `/etc/hosts` en nuestro cliente. Nos falta ahora que todas las URL de la página de Wordpress sigan también este dominio.

En el panel de Wordpress, iremos a `Ajustar > Enlaces permanentes` y escogeremos la opción de "Mes y nombre", para hacer nuestras entradas mas reconocibles.