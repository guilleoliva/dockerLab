
### COMANDOS DOCKER COMPOSE
Aquí los comandos más usados de docker-compose, asumiendo que el usuario ha llevado a cabo el ciclo de creación y mantenimiento de contenedores mediante el manager oficial de docker, y que mantiene un determinado repositorio de imágenes (en local o remoto).

Arrancar un entorno en primer plano, es decir, la ejecución del contenedor es un proceso que hereda de la consola. La salida estándar y de error del contenedor se mostrará por pantalla. Como no se le ha pasado ningún argumento más, Docker Compose tomará el fichero con nombre exacto docker-compose.yml en la ruta en la que se está posicionado en ese momento.

<code>
$ docker-compose up

###### Si se quiere pasar como argumento el fichero de especificación:

$ docker-compose -f <fichero> up

###### Ejecutar en segundo plano:

$ docker-compose -f <fichero> up -d

###### Parar un entorno:
$ docker-compose -f <fichero> kill

###### Para ver los contenedores parados se haría

$ docker ps -a
Eliminar por completo un entorno:
$ docker-compose -f <fichero.yml> rm
Comprobar los logs de los contenedores (lo explicado en el primer punto):
$ docker-compose logs
Comprobar un fichero con la especificación yml del entorno
$ docker-compose -f <fichero.yml> config
</code>


Docker Compose

Docker Compose es una herramienta para definir y correr aplicaciones multicontenedores. Hacemos uso de un archivo docker-compose para configurar los servicios que necesita la aplicación. Luego haciendo uso de un simple comando, creamos los contenedores necesarios e iniciamos todos los servicios especificados en la configuración.
Compose es bueno para desarrollo, pruebas y flujos de trabajo de integración continua.
En el caso de los desarrolladores, tienen la habilidad de correr las aplicaciones en un ambiente aislado e interactuar con la aplicación. El archivo de compose provee una forma de documentar y configurar todas las dependencias de la aplicación y al final solo tiene que hacer uso de un comando para subir todo el ambiente.
Otro punto importante en los procesos de Despliegue e integración continua es el banco de pruebas. Este banco de pruebas requiere de un ambiente donde realizar las pruebas. Compose provee una manera de crear y destruir los ambientes de prueba aislados para ese banco de pruebas. Definiendo todo el ambiente en un archivo compose se puede crear y destruir los ambientes con pocos comandos.
Instalando Docker Compose

Antes de proceder con la instalación de compose es bueno consultar la página de lanzamiento https://github.com/docker/compose/releases y ajustar el comando de instalación acorde con la última versión estable.
$ curl -L https://github.com/docker/compose/releases/download/1.6.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

$ chmod +x /usr/local/bin/docker-compose



Comprobamos la instalación
$ docker-compose --version
Usando Compose

Vamos a ver el uso de Compose en un caso simple, definiendo un ambiente para Wordpress.
Descargamos el CMS de WordPress y lo descomprimimos.
$ curl https://wordpress.org/latest.tar.gz | tar -xvzf –
El directorio resultante es wordpress/, esto lo podemos cambiar a nuestro gusto
$ mv wordpress/ jsitech/
Accedemos al directorio
$ cd jsitech/
El Próximo paso es crear un Dockerfile dentro del directorio, que definirá el contenedor que estará ejecutando la aplicación.
   $ vi Dockerfile
    FROM ubuntu:14.04

    RUN apt-get update && apt-get install php5 php5-mysql -y

    ADD . /codigo
    
    
    Este Dockerfile crea el Contenedor base con los requisitos para ejecutar el cms de WordPress y agrega los archivos correspondiente
El próximo paso es crear un archivo docker-compose.yml que iniciará los servicios y una instancia separada de MySQL.
$ nano docker-compose.yml

 web:
     build: .
     command: php -S 0.0.0.0:8080 -t /jsitech
     ports:
     - "8080:8080"

     links:
     - db

     volumes:
     - .:/jsitech

    db:
     image: mysql
     environment:
     MYSQL_ROOT_PASSWORD: jsitech
     MYSQL_DATABASE: jsitech
     MYSQL_USER: jsitech
     MYSQL_PASSWORD: jsitech
Vamos a explica brevemente los argumentos
web : Nombre que define el servicio
build : crea el contenedor, en este caso pasamos . , para que haga uso del Dockerfile que creamos recientemente.
command : El comando que le pasamos al contenedor al momento de su ejecución.
ports: Mapeo de Puertos
links : Definimos el enlace de los contenedores
volumes : Creamos el volumen que contiene nuestro código
db: Nombre que define nuestro contenedor con la base de datos
image: la imagen donde basará su contenedor
environment: aquí pasamos las variables, en este caso es un contenedor con MySQL y le pasamos la base de datos a crear y las credenciales.
Ya aquí tenemos el ambiente multi-contenedor listo, pero antes de lanzar nuestro proyecto de wordpress debemos configurar wp-config.php con las credenciales que le pasos en las variables

  // ** MySQL settings - You can get this info from your web host ** //

    /** The name of the database for WordPress */
    define('DB_NAME', 'jsitech');

    /** MySQL database username */
    define('DB_USER', 'jsitech');

    /** MySQL database password */
    define('DB_PASSWORD', 'jsitech');

    /** MySQL hostname */
    define('DB_HOST', 'db:3306');
    
    Ya con todo definido en los archivos correspondiente, solo es lanzar el proyecto multi-contenedor. Ejecutamos el comando
# docker-compose up
Esto se encargará de crear las imágenes necesarias y lanzar los contenedores correspondientes a la web y la base de datos.
    
Vamos a ver si todo funciona bien, si recuerdan creamos un mapeo de puertos 8080:8080, lo que quiere decir que debemos poder acceder al contenedor con la IP del host de docker mediante ese puerto.
http://192.168.1.9:8080