---
title: Práctica - Despliegue de aplicaciones python

type: docs
---

# Tarea 1 - Entorno de desarrollo
Creamos una página en Openstack de debian12, donde realizaremos toda la práctica. IP de la máquina 172.22.201.180.

Previamente instalaremos `python3-venv` para poder crear entornos virtuales.

Creamos un nuevo entorno virtual con el comando `python3 -m venv venv`, lo activamos e importamos todo lo necesario del repositorio que nos incluye en el fichero `requirements.txt` que hemos importado. Esto lo hacemos con el comando `pip install -r django_tutorial/requirements.txt`

Una vez instaladas todas las dependencias necesarias para el proyecto, comprobaremos con qué base de datos vamos a trabajar. Para ello, debemos dirigirnos al archivo `settings.py` y dirigirnos al apartdo `DATABASES`. Encontramos la siguiente información, donde aparece el nombre también:
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

Comenzamos creando la base de datos a partir del modelo definido y creamos también el usuario administrador con los siguientes comandos:
```
python3 manage.py migrate
python3 manage.py createsuperuser
```
El usuario y contraseña serán `debian`. Ahora ejecutaremos el servidor web de desarrollo con el siguiente comando, de manera que lo podamos ver desde cualquier dirección y que se ejecute por el puerto 8800. Deberemos cambiar previamente en el `setttings.py` el parámetro `ALLOWED_HOSTS = ['*']`.
```
python3 manage.py runserver 0.0.0.0:8800
```
**¡IMPORTANTE!**: deberemos abrir ese puerto en Openstack para poder acceder.

Accedemos desde nuestra máquina a la IP de la máquina con el puerto indicado anteriormente al servidor y ya la tenemos funcionando:

![Alt text](/images/py-p-1.png)

Si nos vamos a la ruta `/admin` e introducimos el usuario y contraseña de administrador creado anteriormente también comprobamos que está funcionando correctamente:

![Alt text](/images/py-p-2.png)

Creamos dos nuevas preguntas desde el panel de administración y al dirigirnos a la ruta `http://172.22.201.180:8800/polls/` ya encontramos ambas encuestas:

![Alt text](/images/py-p-3.png)

Ahora configuraremos apache2 como servidor web con el módulo wsgi. Este es el contenido del fichero en el `sites-enabled` de apache2:

```
VirtualHost *:80>
    ServerName practica.miguelfigueroa.org
    DocumentRoot /home/debian/django_tutorial

    WSGIScriptAlias / /home/debian/django_tutorial/django_tutorial/wsgi.py
    WSGIDaemonProcess venv_practicadjango python-path=/home/debian/django_tutorial:/home/debian/venv/lib/python3.11/site-packages
    WSGIProcessGroup venv_practicadjango

    <Directory /home/debian/django_tutorial/django_tutorial>
        <Files wsgi.py>
            Require all granted
        </Files>
    </Directory>

        Alias /static/ /home/debian/django_tutorial/static/
        <Directory /home/debian/django_tutorial/static/>
                Require all granted
        </Directory>

    <Directory "/home/debian/django_tutorial">
        Require all granted
        Allow from all
    </Directory>
</VirtualHost>

```
Debemos generar el contenido estático para el servidor web. Añadiremos la siguiente línea en el `settings.py`:
```
STATIC_ROOT = '/home/debian/django_tutorial/static/'

```
Y luego ejecutamos este comando para generar el contenido:
```
python3 manage.py collectstatic
```
**¡IMPORTANTE!**: hay que verificar que el directorio donde se encuentra todo el contenido tenga acceso el usuario y grupo `www-data`.

Comprobamos que ya podemos acceder a la aplicación desde el navegador con la dirección deseada, y que se muestra correctamente:

![Alt text](/images/py-p-4.png)

# Tarea 2 - Entorno de producción

Clonamos el repositorio en el VPS `cerebelum`, creamos el entorno virtual e importamos el fichero `requirements.txt`. Instalamos el paquete `libmariadb-dev pkg-config`, y el módulo `mysqlclient`.

Ahora crearemos una base de datos en mysql, con el comando:
```
sudo mysql
CREATE DATABASE django;
CREATE USER 'django'@'localhost' IDENTIFIED BY 'django';
GRANT ALL PRIVILEGES ON django.* TO 'django'@'localhost';
FLUSH PRIVILEGES;
```

Una vez creada, nos dirijimos al fichero `settings.py` para introducir los ajustes de la base de datos creada.
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'django',
        'USER': 'django',
        'PASSWORD': 'django',
        'HOST': 'localhost',
        'PORT': '',
    }
}
```
Volvemos a nuestro servidor de desarrollo para exportar los datos de nuestra base de datos de sqlite, activando antes el entorno virtual. 
```
python3 django_tutorial/manage.py dumpdata > dbbackup
```
Subimos ese cambio al repositorio y lo descargamos en el VPS. Ahora ejecutamos los siguientes comandos para importar la base de datos:
```
python3 manage.py migrate
python3 manage.py loaddata database.json
```
Una vez importados correctamente, pasamos a configurar el servidor de aplicaciones uwsgi. Instalamos el módulo `uwsgi`, y crearemos un fichero `.ini` con la configuración:
```
[uwsgi]
http = :8080
chdir = /home/miguel/django_tutorial
wsgi-file = django_tutorial/wsgi.py
processes = 4
threads = 2
```
Ahora crearemos una unidad systemd para controlarla con systemctl. Creamos el fichero `/etc/systemd/system/django_tutorial.service` con el siguiente contenido:
```
[Unit]
Description=django_tutorial
After=network.target

[Install]
WantedBy=multi-user.target

[Service]
User=miguel
Group=www-data
Restart=always

ExecStart=/home/miguel/venv/bin/uwsgi /home/miguel/venv/django_tutorial.ini
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID

WorkingDirectory=/home/miguel/django_tutorial
Environment=PYTHONPATH='/home/miguel/django_tutorial:/home/miguel/venv/lib/python3.11/site-packages'

PrivateTmp=true

```
Pasamos ahora a configurar nginx, creando un nuevo archivo en `sites-enabled` con la siguente configuración:
```
server {

        root /home/miguel/django_tutorial;

        # Add index.php to the list if you are using PHP
        index index.html index.php;

        server_name python.miguelfigueroa.es;

        location / {
                proxy_pass http://localhost:8080;
                include proxy_params;
        }

}

```
Agregamos un alias al dominio `www.miguelfigueroa.es`, con el valor `python` y que apunte a `cerebelum.miguelfigueroa.es`, y ya lo tenemos:

![Alt text](/images/py-p-5.png)

Cambiaremos en el fichero `settings.py` el parámetro `DEBUG` al valor `False`.

Nos faltaría generar el contenido estático para el sitio. Para ello ejecutamos el comando `python3 manage.py collectstatic` y lo situamos en la ruta `/var/www/html/django`. Esto deberemos configurarlo también en nginx, quedando el fichero finalmente así:
```
server {

        root /home/miguel/django_tutorial;

        # Add index.php to the list if you are using PHP
        index index.html index.php;

        server_name python.miguelfigueroa.es;

        location / {
                proxy_pass http://localhost:8080/;
                include proxy_params;
        }


        location /static/ {
                alias /var/www/html/django/static/;
}

        error_log /var/log/nginx/django_tutorial_error.log;

}

```

Enlace de la aplicación funcionando: http://python.miguelfigueroa.es

![Alt text](/images/py-p-6.png)

# Tarea 3 - Modificación de nuestra aplicación

Editamos el fichero `polls/templates/polls/index.html` para que aparezca el nombre al principio de la página:
```
{% load static %}

<h2>Miguel Figueroa Escribano</h2>

<link rel="stylesheet" type="text/css" href="{% static 'polls/style.css' %}">

{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
    <li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}

```

Procedemos ahora a cambiar la imagen de fondo. Para ello, nos dirijimos al directorio `polls/static/polls/images` y sustitiuremos el fichero `background.jpg` por una imagen descargada de internet.
```
wget -O background.jpg 'https://lh3.googleusercontent.com/p/AF1QipM0By9PgTe5mK_OfcyrDvp2UGOZdcdBuoWcJqw3=s680-w680-h510'
```

Subimos estos cambios al repositorio desde el servidor de desarrollo. Deberemos cambiar manualmente el contenido estático en producción, pero el resto nos bastará con hacer un pull del repositorio. Deberemos reiniciar el servicio de `nginx` y el que creamos de `uwsgi` para aplicar los cambios. Además es recomendable no usar caché, o comprobar los cambios en una ventana de incógnito. Debemos tener en cuenta que hay que cambiar también la configuración de la base de datos a la que estamos usando en el VPS, en este caso mysql.

Finalmente activamos HTTPS en nuestra aplicación y redirijimos el tráfico de HTTP a HTTPS.

Ya está la app activa HTTPs: https://python.miguelfigueroa.es