## Primeros pasos en Compose

<aside>
⚙ Las instrucciones van en un archivo .yml el cual puede tener cualquier nombre

</aside>

El archivo de docker compose se divide en cuatro partes:

- `version`: Versión de docker compose
- `services` : Los servicios deseados
- `volumes` : (opcional) Volúmenes a crear
- `networks`: (opcional) Redes a crear

Primer contenedor nginx con docker compose:

Con docker:
`docker run -d --name nginx -p 80:80 nginx`

Con Docker Compose:

```yaml
version: '3'
services: 
  web:
    container_name: nginx-1
    ports:
      - "8080:80"
    image: nginx
```

- Levantar contenedor: `docker-compose up -d` 
o si tiene un archivo con nombre diferente a <docker-compose.yml>: `docker-compose -f <docker-compose.yml> up -d`
- Borrar contenedor: `docker-compose down` o `docker-compose -f <docker-compose.yml> down`

## Variables de entorno en Compose

Para definir variables de entorno explícitamente en el documento usamos el parámetro `environment`

```yaml
version: '3'
services: 
  db:
    image: mysql:5.7
    container_name: mysql
    ports:
      - "3306:3306"
    environment:
     - "MYSQL_ROOT_PASSWORD=1234"
```

Otra forma de definir variables de entorno es usando un archivo por aparte con nuestras variables y en nuestro documento de docker-compose hacer referencia con el parámetro `env_file`

common.env:

```
MYSQL_ROOT_PASSWORD=1234
I_AM=Astrik
```

docker-compose:

```yaml
version: '3'
services: 
  db:
    image: mysql:5.7
    container_name: mysql
    ports:
      - "3306:3306"
    env_file:
      - common.env
```

Levantamos nuestro contenedor: `docker-compose up -d`

Ejecutamos “env” en nuestro contenedor para verificar las variables de entorno: 
`docker exec mysql bash -c "env”`

![env](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/562a3f70-f9e7-49d8-84f4-4b59e2139057/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220730%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220730T074310Z&X-Amz-Expires=86400&X-Amz-Signature=5f191ec17347ad44b355874493b8d47d1acfc5f9285bb1ecd4f4aa245eb7d4ca&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

## Volúmenes en Compose

Para crear un volumen usamos la parte de `volumes` de nuestro archivo y para usar volúmenes usamos el parámetro `volumes` dentro de nuestro service

```yaml
version: '3'
services: 
  web:
    image: nginx
    container_name: nginx-1
    ports:
      - "8080:80"
      # Uso del volumen creado
    volumes: 
      - "vol2:usr/share/nginx/hmtl"
# Creacion del volumen
volumes:
  vol2:
```

Al correr nuestro docker compose creara el volumen “<directory>_vol2” el cual sera persistente y no se borrara aunque borremos el contenedor con `docker-compose down`

De igual forma podríamos usar un host volume:

```yaml
version: '3'
services: 
  web:
    image: nginx
    container_name: nginx-1
    ports:
      - "8080:80"
      # Uso del volumen host
    volumes: 
      - "/home/docker/volumes/nginx:usr/share/nginx/hmtl"
```

## Redes en Compose

Crearemos una red llamada “net-test” y dos contenedores apache que usaran la misma red:

```yaml
version: '3'
services: 
  web:
    image: httpd
    container_name: httpd-1
    ports:
      - "8080:80"
    # Uso de la red creada
    networks:
      - net-test
  # Segundo contenedor
  web2:
    image: httpd
    container_name: httpd-2
    ports:
      - "8081:80"
    # Uso de la red creada
    networks:
      - net-test
# Creación de la red
networks:
  net-test:
```

Corremos nuestro archivo con docker-compose: `docker-compose up -d`

Además de poder hacer referencia dentro de la red a los contenedores por su nombre o ip tambien podemos usar el nombre del servicio, en este caso web y web2

![ping web2](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/6d05ebb4-57af-46d8-b849-c6402f6ea277/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220730%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220730T074418Z&X-Amz-Expires=86400&X-Amz-Signature=5e6d19f6993b30fd1b915c5f64ccc683d353d09bce5c926b7936776e53ba5269&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

## Construir imágenes en Compose

Para la construir una imagen primero debemos tener nuestro Dockerfile

```docker
FROM centos:7

RUN mkdir /opt/test
```

Tenemos dos formas de construir nuestra imagen dependiendo la situación

1. Si tenemos nuestro Dockerfile con el nombre “Dockerfile” y en la misma carpeta, basta con el parámetro `build` dentro de un service y el valor de “`.`”
    
    ```yaml
    version: '3'
    services:
      web:
        container_name: web
        image: web-test:v1
        build: .
    ```
    
2. Por el contrario si tiene un nombre diferente o se encuentra en una carpeta diferente, existen parámetros que le podemos pasar a build
    1. `context`: Carpeta
    2. `dockerfile`: Nombre del archivo
    
    ```yaml
    version: '3'
    services:
      web:
        container_name: web
        image: web-test:v1
        build:
          context: ./dockerfiles
          dockerfile: Dockerfile1
    ```
    

Para construir nuestra imagen ejecutamos el siguiente comando:
`docker-compose -f <docker-compose> build`

Y despues lo podremos correr:
`docker-compose -f <docker-compose> up -d`

## Sobrescribe el CMD de un contenedor con Compose

Dentro de nuestro service podemos especificar el comando con el que correrá nuestro contenedor dentro del parámetro `command`:

```yaml
version: '3'
services: 
  web:
    image: centos:7
    command: python -m SimpleHTTPServer 8080
    ports:
     - "80818080"
```

## Limitar recursos con Compose

### Version 2

En la version 2 de Docker Compose se usaban las keys “`cpuset`” y “`mem_limit`” dentro de nuestro service:

```yaml
version: '2'
services:
  web:
    container_name: nginx
    mem_limit: 20m
    cpuset: "0"
    image: nginx:alpine
```

### Version 3

A partir de la version 3 la configuración de los recursos se hacen en la key `deploy` la cual tiene una key `resources`

[Compose file deploy reference](https://docs.docker.com/compose/compose-file/deploy/#resources)

```yaml
version: '3'
services:
  web:
    container_name: nginx
    image: nginx:alpine
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 20m
```

## Política de reinicio de contenedores

Docker nos permite automatizar el reinicio de los contenedores, al ejecutar `docker run --restart` o con la key `restart` en compose dentro de nuestro service

| no | No reinicia (por defecto) |
| --- | --- |
| on-failure[:max-retries] | Reinicia el contenedor si sucede un error (se retorna un codigo diferente a 0). Opcionalmente se puede limitar la cantidad de intentos de reinicio usando la opción :max-retries |
| always | Siempre reinicia el contenedor si se detiene. |
| unless-stopped | Similar a always, excepción de cuando el contenedor se detiene (manualmente o de otra manera). |

**Ejemplo con always:**

```docker
FROM centos:7

COPY start.sh /start.sh

RUN chmod +x /start.sh

CMD /start.sh
```

```bash
#! /bin/bash

echo "initiated"
sleep 6
echo "finished"
```

```yaml
version: '3'
services:
  test:
    container_name: test-restart
    image: restart-image
    build:
      context: ./dockerfiles
      dockerfile: Dockerfile
    restart: always
```

Si ejecutamos el contenedor anterior, se va a iniciar y finalizar cada 6s aproximadamente

![logs test-restart](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/5e6933f2-0b02-4065-b15c-3a6736b8a215/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220730%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220730T074523Z&X-Amz-Expires=86400&X-Amz-Signature=1d9ed2ebc20df933e1a7ff0b29723d7b18ae699cec3e94517f6b6e998322417c&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

## Mas opciones de Docker Compose

<aside>
💡 Si ejecutamos solo el comando `docker-compose` nos dará los comandos que tenemos disponibles

</aside>

- Definir un nombre al proyecto: `docker-compose -p <name>`
- Usar un archivo que se llame diferente a “docker-compose”: `docker-compose -f <archivo>`

```yaml
Usage:  docker compose [OPTIONS] COMMAND
```

```yaml
Docker Compose
```

```yaml
Options:
--ansi string                Control when to print ANSI control characters ("never"|"always"|"auto") (default "auto")
--compatibility              Run compose in backward compatibility mode
--env-file string            Specify an alternate environment file.
-f, --file stringArray           Compose configuration files
--profile stringArray        Specify a profile to enable
--project-directory string   Specify an alternate working directory
(default: the path of the, first specified, Compose file)
-p, --project-name string        Project name
```

```yaml
Commands:
build       Build or rebuild services
convert     Converts the compose file to platform's canonical format
cp          Copy files/folders between a service container and the local filesystem
create      Creates containers for a service.
down        Stop and remove containers, networks
events      Receive real time events from containers.
exec        Execute a command in a running container.
images      List images used by the created containers
kill        Force stop service containers.
logs        View output from containers
ls          List running compose projects
pause       Pause services
port        Print the public port for a port binding.
ps          List containers
pull        Pull service images
push        Push service images
restart     Restart containers
rm          Removes stopped service containers
run         Run a one-off command on a service.
start       Start services
stop        Stop services
top         Display the running processes
unpause     Unpause services
up          Create and start containers
version     Show the Docker Compose version information
```

```yaml
Run 'docker compose COMMAND --help' for more information on a command.
```

## Instalando WordPress + MySQL

```yaml
version: '3'

services:
  # MYSQL
  db:
    container_name: wp-mysql
    image: mysql:5.7
    volumes:
      - $PWD/data/mysql:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: 1234
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    ports:
      - "3306:3306"
    networks:
      - my_net

  # WORDPRESS
  wp:
    container_name: wp-web
    image: wordpress:latest
    depends_on:
      - db
    volumes:
      - $PWD/data/html:/var/lib/mysql
    ports:
      - "80:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
    networks:
      - my_net
networks:
  my_net:
    name: wp-mysql
```

Para correr nuestro WordPress con MySQL persistente solo ejecutamos la siguiente linea:
`docker-compose -f docker-compose up -d`

Si es la primera vez, creara las carpetas ./data/mysql y ./data/html para usarlas como hosts volumes, por el contrario si ya lo hemos corrido con anterioridad y tenemos las carpetas con la data correspondiente, tomara esa data para nuestros contenedores y no tendremos que volver a configurar.

## **Pon a prueba tus conocimientos!**

¡Hola!

La idea de este articulo es que le des solución al siguiente problema utilizando lo que has aprendido.

Deberás crear un docker-compose para cada definición de contenedores de los ejercicios anteriores:

Puntos a tener en cuenta:

- Docker Compose V2
- La imagen (De apache y php) debe poder construirse desde el docker-compose
- Se deben definir los volúmenes solicitados en el ejericicio anterior, más las variables de entorno, límite y demás requisitos especificados en los ejercicios anteriores.
    - 50Mb límites de RAM
    - Solo podrá acceder a la CPU 0
    - Debe tener dos variables de entorno:
        - ENV = dev
        - VIRTUALIZATION = docker
    - El webserver debe ser accesible vía puerto 5555 en el navegador
    - En /opt/source1 (Debes crear el directorio en tu máquina local) debe persistir el código que se incluya en el webserver. En este caso, para pruebas, utilizarás un phpinfo que debe sobrevivir a la eliminación del contenedor.

Adicional a esto, te piden:

- Crear un docker-compose v3 con dos servicios: db y admin.
- En el servicio DB, debe ir una db con mysql:5.7 y las credenciales de tu preferencia.
- En el admin, debes usar la imagen oficial de phpmyadmin, y por medio de redes, comunicarla con mysql. Debes exponer el puerto de tu preferencia y para validar que funcione, debes loguearte en el UI de phpmyadmin vía navegador, usando las credenciales del root de mysql.

Que te diviertas!

### Solución

Contenedor apache + php

```yaml
version: '2'

services:
  web:
    container_name: php-apache
    image: php:8.1.9RC1-apache
    volumes:
      - /opt/source1:/var/www/html
    environment:
      ENV: dev
      VIRTUALIZACION: docker
    ports:
      - "5555:80"
```

Contenedores MySQL y PHPMyAdmin conectados

```yaml
version: '3'

services:
  # MYSQL
  db:
    container_name: mysql
    image: mysql:5.7
    volumes:
      - $PWD/data/mysql:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: 1234
      MYSQL_DATABASE: init
      MYSQL_USER: astrik
      MYSQL_PASSWORD: astrik
    ports:
      - "3306:3306"
    networks:
      - my_net

  # phpmyadmin
  admin:
    container_name: phpMyAdmin
    image: phpmyadmin
    depends_on:
      - db
    environment:
      PMA_HOST: db:3306
    ports:
      - "80:80"
    networks:
      - my_net
      
networks:
  my_net:
    name: mysql
```
