---
title: Unidad 4 - Docker
sidebar:
  open: true
---

Para la instalación de Docker seguimos la [documentación oficial de instalación](https://docs.docker.com/engine/install/debian/)

## Comandos:
- `docker run hello-world` → si no encuentra la imagen localmente la descarga y se ejecuta el programa. Cuando se ejecuta se para el contenedor
- `docker images` → muestra las imágenes que tenemos descargadas
- `docker ps` → contenedores que están ejecutándose
- `docker ps -a` → contenedores que están parados
- `docker rm nombrecontenedor` → borra el contenedor
- `docker rmi hello-world:latest` → borra las imágenes
- `docker run -it --name contenedor1 ubuntu bash` → arranca un contenedor con ubuntu, con el nombre contenedor1, y con el `-it` ejecuta el comando bash en ubuntu de forma interactiva.
- `docker start contenedor1` → para arrancar de nuevo el contenedor. También sirven `stop`, `restart`.
- `docker atach contenedor1` → para conectarse de nuevo al contenedor
- `docker rm contenedor` → borra el contenedor. con `-rf` lo borra aunque se esté ejcutando.
- `docker run -d --name my-apache-app -p 8080:80 httpd:2.4` → `-d` porque es un demonio. `-p` es un DNAT: cuando acceda al puerto 8080 del host nos llevará al puerto 80 del contenedor
- `docker exec` → ejecutar una instrucción en un contenedor que se está ejecutando

## Tips:
- No se pueden borrar imágenes que tengan un contenedor a partir de esa imagen
- Añadir el usuario del sistema al grupo docker para poder manejarlo sin privilegios, aunque el contenedor siempre se ejecuta desde root.