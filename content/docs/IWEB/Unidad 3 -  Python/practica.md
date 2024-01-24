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