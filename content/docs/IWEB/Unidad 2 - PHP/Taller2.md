---
title: Taller 2 - Configuración Apache2 + fpm-php
type: docs
---

Para deshabilitar el módulo de apache2 que permite la ejecución de PHP ejecutamos el comando `a2dismod php8.2`, y reiniciamos el servicio de apache2. Tras esto, instalamos `php8.3-fpm`.

Para configurar apache2 para que funcione con php-fpm activamos los siguientes módulos: `a2enmod proxy_fcgi setenvif`. Activaremos php-fpm para todos los virtualhost con el comando `a2enconf php8.2-fpm`, teniendo en cuenta que está escuchando en un socket UNIX. Reiniciamos el servicio de apache2 y ya tendríamos de nuevo funcionando la app Biblioteca.

![Captura de acceso a aplicación funcionando con fpm-php](/images/t2-1.png)

Además, comprobamos en el `info.php` que está corriendo con el módulo fpm-php:

![PHP corriendo con fpm-php](/images/t2-2.png)