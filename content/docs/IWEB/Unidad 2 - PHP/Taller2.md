---
title: Taller 2 - Configuración Apache2 + fpm-php
type: docs
---

## Configuración fpm-php
Para deshabilitar el módulo de apache2 que permite la ejecución de PHP ejecutamos el comando `a2dismod php8.2`, y reiniciamos el servicio de apache2. Tras esto, instalamos `php8.3-fpm`.

Comprobamos que están corriendo procesos de php-fpm con el comando `ps aux | grep php-fpm`:

![`ps aux | grep php-fpm`](/images/t2-3.png)

## Configuración de apache2 con fpm-php
Para configurar apache2 para que funcione con php-fpm activamos los siguientes módulos: `a2enmod proxy_fcgi setenvif`. Activaremos php-fpm para todos los virtualhost con el comando `a2enconf php8.2-fpm`, teniendo en cuenta que está escuchando en un socket UNIX. Reiniciamos el servicio de apache2 y ya tendríamos de nuevo funcionando la app Biblioteca.

![Captura de la aplicación Biblioteca funcionando con fpm-php](/images/t2-4.png)

Además, comprobamos en el `info.php` que está corriendo con el módulo fpm-php:

![PHP corriendo con fpm-php](/images/t2-2.png)

## Cambio a socket TCP
Ahora cambiaremos la configuración de php-fpm para que en vez de que escuche por un socket UNIX, utilice el puerto tcp/9000. Para ello, editaremos en el fichero `/etc/php/8.2/fpm/pool.d/www.conf` el parámetro `listen`, dejándolo de la siguiente forma:

`listen = 127.0.0.1:9000`

Tras esto deberemos reiniciar el módulo fpm-php `systemctl restart php8.2-fpm`. Ahora deberemos editar también la conexión desde apache2, editando el fichero `/etc/apache2/conf-available/php8.2-fpm`. En él editaremos editaremos el siguiente parámetro:

`SetHandler "proxy:fcgi://127.0.0.1:9000"`

Reiniciamos el servicio de apache2 y ya tendríamos de nuevo funcionando la app Biblioteca, esta vez utilizando un socket TCP.

## `memory_limit` en fpm-php
Para cambiar el `memory_limit` con el módulo fpm-php se edita el fichero `/etc/php/8.2/fpm/php.ini`, estableciendo el parámetro en `256M`.

![Editado el parámetro `memory_limit`](/images/t2-5.png)