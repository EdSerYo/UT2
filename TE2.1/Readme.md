## Tarea Evaluable 2.1. Dockerización de stack LAMP

### Presentación

Tarea del módulo de Desarrollo Aplicaciones Web sobre Docker, desarrollada por:
- Sergio Holguera [GitHub](https://github.com/EdSerYo) [DockerHub](https://hub.docker.com/u/edseryo)


### Recursos previos

Se necesita tener instalado en el sistema DOCKER y Docker-compose. Para ello dirigirnos a la web oficial [DOCKER](https://www.docker.com/products/docker-desktop/) y descargar e instalar según version de nuestro sistema.

### Objetivos

- Conocer las ventajas que nos proporciona el uso de la tecnología de contenedores.
- Conocer los conceptos principales sobre el despliegue de aplicaciones web utilizando contenedores.
- Conocer los conceptos fundamentales sobre Docker.
- Trabajar con imágenes Docker.
- Trabajar con Docker y docker compose (orquestación de contenedores).
- Desplegar aplicaciones web sencillas en contenedores.

### Desarrollo

#### Paso 1. Descargar recursos.

En este paso crearemos y prepararemos una carpeta de trabajo. Para eso hacemos:
1. Crear una carpeta de trabajo llamada **"UT2/T2.1"** ya sea desde terminal con *"mkdir"* o desde el explorador de archivos con *"Nueva Carpeta"* y creamos la siguiente estructura de árbol.

![Imagen Paso 1.1](./img/Imagen1.1.gif) ![Imagen Paso 1.2](./img/Imagen1.2.jpg)

2. Descargar el archivo [Recurso](https://github.com/jssfpciclos/DAW_daweb/blob/main/UT2/TE2.1/res/Tarea2.1.recursos.rar), copiarla a la carpeta de trabajo **"UT2/src/docker-lamp"** y la descomprimimos.

![Imagen Paso 1.3](./img/imagen1.3.jpg)

#### Paso 2. Imagen docker PHP

En este  paso vamos a crear una imagen Docker que incluya Apache y PHP, a partir de la imagen oficial de PHP 8.0.0 con Apache, e incluyendo el driver de MySQL para PHP. Para esto hay dos métodos. Construir una imagen paso a paso o con un **DOCKERFILE**. A continuación se desarrollará el método paso a paso.

##### Paso a paso #####

1. Descargar la imagen. Para ello utilizamos el siguiente comando:

    > docker run -ti --name daw_te2_1 php:8.0.0-apache /bin/bash 

    - **docker run** -> comando para crear contenedores de las imagenes.
    - **-ti** -> parametro para que el contenedor que creeemos sea interactivo.
    - **--name daw_te2_1** -> con *--name* le asignaremos un nombre al contenedor
    - **php:8.0.0-apache** -> nombre de la imagen que trabajaremos. Lo que viene despues de ":" es la versión de la imagen. Esto significa que trabajaremos con una imagen de php obteniendo la version 8.0.0 que contiene apache ya instalado.

    ![Imagen Paso 2.1](./img/Imagen2.1.gif)

    Despues de ejecutar el comando podemos observar que en nuestra consola estamos logueado dentro del contenedor. Todo comando que ejecutemos aqui se queda dentro del contenedor.

    ![Imagen Paso 2.2](./img/Imagen2.2.jpg)

2. Instalar el driver de MySQL.

    > docker-php-ext-install mysqli

    Con este comando instalamos el driver dentro del contenedor

    ![Imagen Paso 2.3](./img/Imagen2.3.gif)
   
3. Instalar librerias del driver MySQL

    Para ello debemos ejecutar varios comandos seguidos.

    > apt-get update
    >
    > apt-get install -y sendmail libpng-dev 
    >
    > apt-get install -y libzip-dev 
    >
    > apt-get install -y zlib1g-dev 
    >
    > apt-get install -y libonig-dev 
    >
    > rm -rf /var/lib/apt/lists/* 
    >
    > docker-php-ext-install zip
    >
    > docker-php-ext-install mbstring
    >
    > docker-php-ext-install zip
    >
    > docker-php-ext-install gd

    Todo esto son librerias necesarias para que el driver MySQL funcione

4. Y ya por último habilitamos el módulo *"rewrite"* de apache

    > a2enmod rewrite

<br><br>
Y tras todo estos pasos ya tendriamos lista un contenedor para trabajar. Después de esto tendriamos que construir una imagen de este contenedor. Para ello:

- Salimos del contenedor con **exit** y volvemos a nuestro entorno principal

- Ejecutamos el codigo:

    > docker ps -a

    Este comando nos muestra a todos los contenedores que estén o no en ejecucion de este comando nos fijamos mayormente en el id del container

    ![Imagen Paso 2.4](./img/Imagen2.4.jpg)

- Creamos una imagen del contenedor anterior. Para ello ejecutamos el siguiente código:

    > docker commit 786cf5994ade DAW_PHP_Apache_MySQl

    Con **commit** creamos una imagen. Para ello introducimos dos parámetros el id del contenedor base y el nombre de la imagen que queremos crear.

    ![Imagen Paso 2.5](./img/Imagen2.5.jpg)


##### DOCKERFILE #####

Como se puede ver líneas más arriba. la ejecucion de paso a paso es muy tediosa. Para simplificar esto se utilizan los archivos **DOCKERFILE**. Este archivo es un archivo de texto plano que contiene una serie de instrucciones necesarias para crear una imagen que, posteriormente, se convertirá en una sola aplicación utilizada para un determinado propósito.

El archivo lo crearemos en **"TE2.1/src/"** tiene el siguiente codigo:

> FROM php:8.0.0-apache
> 
> RUN docker-php-ext-install mysqli
> RUN apt-get update \
>    && apt-get install -y sendmail libpng-dev \
>    && apt-get install -y libzip-dev \
>    && apt-get install -y zlib1g-dev \
>    && apt-get install -y libonig-dev \
>    && rm -rf /var/lib/apt/lists/* \
>    && docker-php-ext-install zip
>
> RUN docker-php-ext-install mbstring
> RUN docker-php-ext-install zip
> RUN docker-php-ext-install gd
>
> RUN a2enmod rewrite

Como se puede observar son el mismo código que en el apartado anterior diferenciando dos comandos.

- **FROM**. Con este comando indicamos de donde procede la imagen con la que vamos a trabajar.
- **RUN**. Con este ejecutaremos comando dentro dentro de la imagen durante el proceso de creacion.

Una cosa importante es la creación de capas dentro de una imagen. Cada capa representa un cambio realizado en la imagen. Las capas son de solo lectura y se pueden compartir entre varios contenedores. Pero contra menos capas realicemos en nuestra imagen, mejor rendimiento puede tenerr. Con un archivo **DOKERFILE** podemos reducir esto enlazando varias secuencia. En nuestro caso, todo esas instalaciones de librerias para el driver de **MySQL** se han reducido en una sola línea de comando.

Ya solo nos faltaría construir la imagen. Para ello introducimos:

> docker build -t php-apache8.0-sdf:1.0 .

- Con **build** construimos una imagen para indicarle que utilizaremos un **Dockerfile** pondremos un "." al final de la setencia. 
- Con **-t** indicamos que vamos a introducir una version.
- **php-apache8.0-sdf:1.0** es el nombre de la imagen. Con ":" añadimos la versión de la imagen.

![Imagen Paso 2.6](./img/Imagen2.6.gif)

Para comprobar que nuestra imagen se ha creado ejecutamos:

> docker images

![Imagen Paso 2.7](./img/Imagen2.7.jpg)


Como se puede observar, simplifica mucho la creacion de una imagen. Con un archivo y un solo comando creamos una imagen lista para funcionar.

Por último borraremos esta imagen para que no interfiera en el resto de la práctica

> docker rmi ad82f768cc87
>
> docker images

![Imagen Paso 2.8](./img/Imagen2.8.jpg)


#### Paso 3. Docker-compose

Para crear un servicio de varias imagenes y contenedores utilizaremos el **docker-compose**. Esta función se respalda de un archivo **YML** donde estará la lista de todas las instrucciones que incluiremos en la creación del servicio.

Principalmente se divide en tres partes: *servicios, volumenes y redes*

##### SERVICIOS #####

En servicios incluiremos todas las imagenes que vamos a trabajar. En cada imagen definiremos sus características

- www:

    Esta es la primera imagen que construiremos. Esta imagen se basará en la imagen que hemos construido en el apartado anterior. Para construirla aqui no hace falta escribir todo lo anterior, si no que haremos refeencia al archivo **Dockerfile** que hemos construido anteriormente (este es el motivo que por el que hemos eliminado la imagen que se creó)
    
    El código qon el que trabajaremos será:

    > www:<br>
    >   &nbsp;&nbsp;&nbsp;build: .<br>
    >   &nbsp;&nbsp;&nbsp;image: daw/lamp-apache-php8-sdf:1.0<br>
    >   &nbsp;&nbsp;&nbsp;ports: <br>
    >       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- "9000:80"<br>
    >   &nbsp;&nbsp;&nbsp;volumes:<br>
    >       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- ./www:/var/www/html<br>
    >   &nbsp;&nbsp;&nbsp;depends_on:<br>
    >       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-db<br>
    >   &nbsp;&nbsp;&nbsp;networks:<br>
    >       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-lamp-network

    Este código tan raro que hemos introducido lo vamos a desarrollar poco a poco:

    - **www:**. Este es nombre del servicio. En esta imagen construiremos la base de todo nuestro servicio. contendrá PHP, Apache y MySQL, además de que contendrá nuestros archivos.
    - **build**. Con esto definiremos como construiremos la imagen. Como tenemos un **Dockerfile** utilizaremos "." para hacer referencia a él que está en la misma carpeta que el archivo del *docker-compose*.
    - **image**. Definiremos el nombre de la imagen que construiremos y su version.
    - **ports**. Aquí definiremos el/los puerto/s que trabajará la imagen. Cada uno de ellos estará definido en la lista. Esto se configura poniendo primero el puerto del host y despues el del contenedor separado ambos por ":". En el ejemplo, el host (nuestro ordenador) tendrá el puerto 9000 y el contenedor tendrá el puerto 80
    - **volumes:**. Una de la partes importantes de **Docker** es que los contenedores pueden tener una carpeta compartida nuestro sistema. Aqui definiremos esta/s carpeta/s. A igual que los puertos, cada una de las carpetas se definen en una lista
    - **depends_on**. Un contenedor puede ser individual o puede estar relacionado con otro/s. Aqui se puede definir esta dependencia.
    - **networks**. Aqui se le designará la red con la que trabajará. Puede ser una red propia de **Docker** o una customizada (ir al apartado de redes de más adelante)

    Para poder probar esta parte ponemos el siguiente comando en la consola de comando

    > docker-compose up --build

    ![Imagen Paso 3.1](./img/Imagen3.1.gif)

    Podemos observar que una vez terminado la construcción de la imagen no podemos hacer nada con la consola. Docker-compose se queda en modo *"attached"*. Esto significa que se queda en modo espera. Por la consola irá mostrando los logs que se irán producciendo en el contenedor. 

    ![Imagen Paso 3.2](./img/Imagen3.2.jpg)

    Para salir de este modo basta con pulsar la combinación de tecla **"Ctrl + c"**. Esto procederá a cerrar el contenedor y eliminarlo.

    Para no está en el modo *attached* cuando ejecutamos el compose, podemos utiliza el modo *detached*. De este modo podemos tener el control de la consola todo el rato. Para ello modificamos el código anterior añadiendo **"-d"** al código introducido.

    > docker-compose up -d --build 
    
    <br>

- db:

    Este servicio se encargará de gestionar la base de datos. El código que lo define es:

    > db:<br>
    >   &nbsp;&nbsp;&nbsp;image: mysql:8.0<br>
    >   &nbsp;&nbsp;&nbsp;container_name: lamp-mysql-sdf<br>
    >   &nbsp;&nbsp;&nbsp;ports: <br>
    >        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- "3399:3306"<br>
    >   &nbsp;&nbsp;&nbsp;command: --default-authentication-plugin=mysql_native_password<br>
    >   &nbsp;&nbsp;&nbsp;environment:<br>
    >       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MYSQL_DATABASE: dbname<br>
    >       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MYSQL_ROOT_PASSWORD: test <br>
    >       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MYSQL_USER: lamp<br>
    >       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MYSQL_PASSWORD: lamp<br>
    >   &nbsp;&nbsp;&nbsp;volumes:<br>
    >       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- ./dump:/docker-entrypoint-initdb.d<br>
    >       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- ./conf:/etc/mysql/conf.d<br>
    >       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- db_data:/var/lib/mysql<br>
    >   &nbsp;&nbsp;&nbsp;networks:<br>
    >       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- lamp-network<br>

    Procedamos al desarrollo de esto:
    - **db:**. El nombre del servicio.
    - **container_name:**. Aqui definiremos el nombre del container que se cree durante el proceso.
    - **ports**. Aquí definiremos el/los puerto/s que trabajará la imagen. Cada uno de ellos estará definido en la lista. Esto se configura poniendo primero el puerto del host y despues el del contenedor separado ambos por ":". En el ejemplo, el host (nuestro ordenador) tendrá el puerto 9000 y el contenedor tendrá el puerto 80.
    - **environment**. Aqui se declaran las variables de entorno del contenedor. Por ejemplo, en este desarrollo se declaran las claves de conexion a la base de datos, contraseñas y/o usuarios 
    - **volumes:**. Una de la partes importantes de **Docker** es que los contenedores pueden tener una carpeta compartida nuestro sistema. Aqui definiremos esta/s carpeta/s. A igual que los puertos, cada una de las carpetas se definen en una lista
    - **networks**. Aqui se le designará la red con la que trabajará. Puede ser una red propia de **Docker** o una customizada (ir al apartado de redes de más adelante)

    De nuevo ejecutamos **docker-compose** para probar que todo va bien.
    

    > docker-compose up --build<br>


    ![Imagen Paso 3.3](./img/Imagen3.3.gif)

    Como podemos observar si utilizamos el **[Workbench](https://dev.mysql.com/downloads/workbench/)**, u otro programa que deseemos, podemos acceder a la base de datos sin problemas.

- phpmyadmin

    Este ultimo servicio creará un contenedor para tener como recurso a **PHPAdmin** como servicio de conexion a la base de datos a través del navegador web. Se conectará al contenedor de la base de datos para obtener la información.

    Como de constumbre tendremos el siguiente código:

    > phpmyadmin:<br>
    >  &nbsp;&nbsp;&nbsp;image: phpmyadmin/phpmyadmin<br>
    >  &nbsp;&nbsp;&nbsp;container_name: lamp-phpmyadmin-sdf<br>
    >  &nbsp;&nbsp;&nbsp;depends_on:<br> 
    >      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- db<br>
    >  &nbsp;&nbsp;&nbsp;ports:<br>
    >      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- 8900:80<br>
    >  &nbsp;&nbsp;&nbsp;environment:<br>
    >      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MYSQL_USER: lamp<br>
    >      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MYSQL_PASSWORD: lamp<br>
    >      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MYSQL_ROOT_PASSWORD: test <br>
    >  &nbsp;&nbsp;&nbsp;networks:<br>
    >      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- lamp-network<br>

    Destripando un poco el código anterior tenemos que:

    - **phpmyadmin:**. El nombre del servicio.
    - **image**. Declaramos cual es la imagen base con la que trabajaremos.
    - **container_name:**. Aqui definiremos el nombre del container que se cree durante el proceso.
    - **depends_on**. Un contenedor puede ser individual o puede estar relacionado con otro/s. Aqui se puede definir esta dependencia/relacion.
    - **ports**. Aquí definiremos el/los puerto/s que trabajará la imagen. Cada uno de ellos estará definido en la lista. Esto se configura poniendo primero el puerto del host y despues el del contenedor separado ambos por ":". En el ejemplo, el host (nuestro ordenador) tendrá el puerto 9000 y el contenedor tendrá el puerto 80.
    - **environment**. Aqui se declaran las variables de entorno del contenedor. Por ejemplo, en este desarrollo se declaran las claves de conexion a la base de datos, contraseñas y/o usuarios 
    - **networks**. Aqui se le designará la red con la que trabajará. Puede ser una red propia de **Docker** o una customizada (ir al apartado de redes de más adelante)

    De nuevo ejecutamos **docker-compose** para probar que todo va bien.

    > docker-compose up --build<br>

    ![Imagen Paso 3.4](./img/Imagen3.4.gif)

    Para poder probar que todo ha ido bien debemos ir esta vez al explorador web (Chrome, Safari, Brave...) y poner en la barra de navegación:

    > localhost:8900

    Esto nos abrirá directamente la pagina web que tenemos en nuestra carpeta **/www/** y se nos mostrará por el navegador.

    ![Imagen Paso 3.5](./img/Imagen3.5.jpg)


