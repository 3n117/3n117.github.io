---
layout: post
title: GLPI Docker SSL
date: 27-11-2024
categories: [documentacion]
tag: [docker, ssl, glpi]
---


## 1. Descargar la imagen de Docker
Usa una imagen verificada para evitar problemas. En este caso, utilizaremos la imagen oficial de GLPI proporcionada por Elestio.
```bash
$ docker pull elestio/glpi
``` 

## 2. Crear un volumen para Apache
Docker, en ocasiones, sobrescribe la configuración de Apache. Para evitar esto, crearemos un volumen que será utilizado más adelante.
```bash
$ docker volume create apache-config
``` 

## 3. Crear y ejecutar el contenedor
Ejecuta el siguiente comando para crear y lanzar un contenedor configurado para usar HTTPS:
```bash
$ docker run -d --name glpi-https2 -v apache-config:/etc/apache2 -p 888:443 elestio/glpi
``` 
### Descripción de los parámetros del comando:
- `-d` Ejecuta el contenedor en segundo plano.
- `--name glpi-https2` Asigna el nombre glpi-https2 al contenedor.
- `-v apache-config:/etc/apache2` Usa un volumen llamado apache-config para personalizar y guardar la configuración de Apache.
- `-p 888:443` Expone el puerto HTTPS (443) del contenedor en el puerto 888 de la máquina anfitriona.
- `elestio/glpi` Especifica la imagen de GLPI.

## 4. Preparar el entorno dentro del contenedor
Accede al contenedor y actualiza los paquetes instalados:
```bash
$ sudo apt update
``` 

## 5. Instalar OpenSSL
Para habilitar HTTPS, instala OpenSSL con el siguiente comando:
```bash
$ sudo apt install openssl
``` 

## 6. Crear un certificado SSL autofirmado
Genera una clave privada y un certificado autofirmado válido por un año. Responde a las preguntas del asistente. Al preguntar por el "Common Name (CN)", escribe localhost para que el certificado sea válido para ese nombre.
```bash
$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout localhost.key -out localhost.crt
``` 

## 7. Configurar Apache para HTTPS
### Editar el archivo de configuración SSL
Abre y edita el archivo ubicado en /etc/apache2/sites-available/default-ssl.conf. Configúralo de la siguiente manera:
```
<IfModule mod_ssl.c>
    <VirtualHost _default_:443>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html/glpi/public
        <Directory /var/www/html/glpi/public>
            Require all granted
            RewriteEngine On
            RewriteCond %{REQUEST_FILENAME} !-f
            RewriteRule ^(.*)$ index.php [QSA,L]
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
        SSLEngine on
        SSLCertificateFile      /etc/apache2/certs/localhost.crt
        SSLCertificateKeyFile   /etc/apache2/certs/localhost.key
        <FilesMatch "\.(cgi|shtml|phtml|php)$">
            SSLOptions +StdEnvVars
        </FilesMatch>
        <Directory /usr/lib/cgi-bin>
            SSLOptions +StdEnvVars
        </Directory>
    </VirtualHost>
</IfModule>

``` 

## 2. Crear un volumen para Apache
Docker, en ocasiones, sobrescribe la configuración de Apache. Para evitar esto, crearemos un volumen que será utilizado más adelante.
```bash
$ docker volume create apache-config
``` 

## 8. Configurar Apache para escuchar solo en el puerto 443
Edita el archivo /etc/apache2/ports.conf para que Apache no escuche en el puerto 80. Cambia su contenido para que quede como sigue:
```
Listen 443
``` 
Guarda y cierra el archivo.

## 9. Habilitar el módulo SSL
Ejecuta el siguiente comando para habilitar el módulo SSL en Apache:
```bash
$ sudo a2enmod ssl
``` 

## 10. Habilitar el sitio SSL
Si el sitio default-ssl no está habilitado, actívalo con este comando:
```bash
$ sudo a2ensite default-ssl
``` 

## 11. Reiniciar Apache
Finalmente, reinicia el servicio de Apache para aplicar los cambios:
```bash
$ sudo service apache2 reload
``` 


Con este procedimiento, habrás configurado GLPI en Docker con soporte para SSL, garantizando una comunicación segura mediante HTTPS.



