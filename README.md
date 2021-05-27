# HTTPS con Let’s Encrypt, Docker y Docker Compose

Para esta práctica usamos Docker Compose para instalar una imagen de Prestashop y agregarle una url con certificado SSL mediante Let's Encrypt usando la imagen de HTTPS-PORTAL. Para ello creamos un archivo docker-compose que contenga lo siguiente:

- Imagen de mysql para la base de datos:

    ```
    mysql:
      image: mysql
      command: --default-authentication-plugin=mysql_native_password
      ports: 
        - 3306:3306
      environment: 
        - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
        - MYSQL_DATABASE=${MYSQL_DATABASE}
        - MYSQL_USER=${MYSQL_USER}
        - MYSQL_PASSWORD=${MYSQL_PASSWORD}
      volumes: 
        - mysql_data:/var/lib/mysql
      networks: 
        - backend-network
      restart: always
    ```

    El apartado de enviroment esta contenido en un archivo .env:

    ```
    MYSQL_USER=ps_user
    MYSQL_ROOT_PASSWORD=root
    MYSQL_DATABASE=prestashop
    MYSQL_PASSWORD=ps_password
    ```

- Una imagen de phpMyAdmin para administrar la base de datos:

    ```
    phpmyadmin:
      image: phpmyadmin
      ports:
        - 8080:80
      environment: 
        - PMA_ARBITRARY=1
      networks: 
        - backend-network
        - frontend-network
      restart: always
      depends_on: 
        - mysql
    ```

- Una imagen de Prestashop:

    ```
    prestashop:
      image: prestashop/prestashop
      environment: 
        - DB_SERVER=mysql
      volumes:
        - prestashop_data:/var/www/html
      networks: 
        - backend-network
        - frontend-network
      restart: always
      depends_on: 
        - mysql
    ```

- Una imagen de HTTPS-PORTAL que dará el certificado SSL a nuestra web:

    ```
    https-portal:
      image: steveltn/https-portal:1
      ports:
        - 80:80
        - 443:443
      restart: always
      environment:
        DOMAINS: 'httpsscriptiaw.ddns.net -> http://prestashop:80'
        STAGE: 'production' # Don't use production until staging works
        # FORCE_RENEW: 'true'
      networks:
        - frontend-network
    ```

    En el apartado de enviroment enlazamos la url que hemos creado para la web con el servidor de prestashop en el puerto 80 que es donde se crea en principio.

Una vez ejecutado Docker Compose instalamos Prestashop desde la url como el la anterior práctica pero con un añadido. Al acabar la instalación debemos, desde phpMyAdmin o desde la consola de comandos entrando al contenedor de mysql, ejecutar las siguiente líneas:

![phpMyAdmin-1](https://raw.githubusercontent.com/arr588/iaw-https-docker/main/img/2.png)

![phpMyAdmin-2](https://raw.githubusercontent.com/arr588/iaw-https-docker/main/img/3.png)