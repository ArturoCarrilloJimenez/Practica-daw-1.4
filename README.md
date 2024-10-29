# Practica-daw-1.4

Esta practica es una continuación de la practica 1.2 te recomido ver antes la [documentación de esta](https://github.com/ArturoCarrilloJimenez/Practica-daw-1.2)

Vamos ha realizar un certificado autorizado de HTTPS para poder realizar conexiones seguras, este lo probaremos a nivel local y este al ser autorizado lo detectara como no seguro

En primer lugar tendremos la siguiente estructura de archivos

```
├── README.md
├── conf
│   ├── 000-default.conf
│   └── default-ssl.conf
└── scripts
    ├── .env
    ├── deploy.sh
    ├── install_lamp.sh
    ├── install_tools.sh
    └── setup_selfsigned_certificate.sh
```

En el archivo setup_selfsigned_certificate.sh comenzaremos realizando el script con nuestra estructura básica

```sh
#!/bin/bash

set -ex #-e rompe la ejecucion al haber un error y la x paso a paso

source .env # Importamos variables de entorno
```

Posteriormente debemos de actualizar la lista de instalación con el comando ``apt update`` y actualizaremos los paquetes que tengamos instalados en nuestro servidor mediante el comando ``apt upgrade -y``

Una vez echo esto, comenzaremos con la creación de nuestro certificado autorizado el cual nos permitirá la conexión mediante HTTPS, para esto añadiremos el siguiente comandó 

``` sh
openssl req \
  -x509 \
  -nodes \
  -days 365 \
  -newkey rsa:2048 \
  -keyout /etc/ssl/private/apache-selfsigned.key \
  -out /etc/ssl/certs/apache-selfsigned.crt \
  -subj "/C=$OPENSSL_COUNTRY/ST=$OPENSSL_PROVINCE/L=$OPENSSL_LOCALITY/O=$OPENSSL_ORGANIZATION/OU=$OPENSSL_ORGUNIT/CN=$OPENSSL_COMMON_NAME/emailAddress=$OPENSSL_EMAIL"
```

Y ademas añadiremos al archivo ``.env`` las siguientes variables necesarias para nuestro certificado

``` sh
OPENSSL_COUNTRY= # Variable de el país, por ejemplo para España pondremos ES
OPENSSL_PROVINCE= # Variable de provincia del certificado
OPENSSL_LOCALITY= # Variable de la localidad del certificado
OPENSSL_ORGANIZATION= # Organización que realiza la certificación 
OPENSSL_ORGUNIT= # Organismo o departamento que lo realiza
OPENSSL_COMMON_NAME= # Nombre de dominio al que se lo asignaremos
OPENSSL_EMAIL= # Email de la organización
```
Para que este función de forma correcta debemos de crear una archivo de configuración llamado ``default-ssl.conf`` el cual moveremos a apache mediante el comando ``cp ../conf/default-ssl.conf /etc/apache2/sites-available
``

Esta configuración tendrá el siguiente contenido, este nos permitirá poder conectarnos mediante HTTPS con el puerto __443__ y con el certificado anteriormente creado

``` sh
<VirtualHost *:443>
    #ServerName practica-https.local
    DocumentRoot /var/www/html
    DirectoryIndex index.php index.html

    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
    SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key
</VirtualHost>
```

Ademas modificaremos el archivo ``000-default.conf`` para que nos redirija siempre a el protocolo HTTPS y lo copiaremos con el siguiente comando ``cp ../conf/000-default.conf /etc/apache2/sites-available``

Este tiene el siguiente contenido
``` sh
<VirtualHost *:80>
    #ServerName practica-https.local
    DocumentRoot /var/www/html

    # Redirige al puerto 443 (HTTPS)
    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteRule ^ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
</VirtualHost>
```

Una vez copiados estos dos archivos de configuración en la ruta deseada, habilitamos el ``default-ssl.conf`` mediante el comando ``a2ensite default-ssl.conf``

Ademas debemos de habilitar el modulo SSL/TSL para poder conectarnos mediante HTTPS, esto se realiza con el siguiente comando ``a2enmod ssl``

Ya terminada la activación del certificado y el protocolo HTTPS comenzaremos con activar la redirection a este protocolo mediante le comando ``a2ensite 000-default.conf``

En caso de tener algún archivo de configuración mas activado deberemos de desactivarlo mediante ``a2dissite`` y seguido del nombre del archivo

Por ultimo para que se apliquen los cambios debemos de restaurar apache mediante el comando ``systemctl restart apache2``

Para poder realizar pruebas con un host local debemos de entrar en el archivo ``C:\Windows\System32\drivers\etc\hosts`` en caso de Windows y entrar con permiso de administrador y hay pondremos __al final la IP y el host que queremos que nos redirija__, este solo se podrá probar a nivel local, para poder probarlo en otras maquinas debemos de contratar un dominio