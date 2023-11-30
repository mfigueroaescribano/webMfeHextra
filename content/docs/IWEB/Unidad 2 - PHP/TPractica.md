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
El CMS que instalaremos para esta práctica será **Drupal**. En el servidor web, instalaremos `apache2 php8.2-fpm`, ya que haremos funcionar apache2 con el módulo php-fpm. Configuraremos apache2 para que funcione con este módulo, explicado en [este enlace](http://localhost:1313/docs/iweb/unidad-2-php/taller2/#configuraci%c3%b3n-de-apache2-con-fpm-php). Tras esto, configuraremos un Virtualhost en apache2:

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

Configuramos la resolución estática en nuestro cliente para que se acceda a través de www.miguelfigueroa.org.
![Fichero `/etc/hosts`](/images/p-1.png)