Servidor Web Apache
Docker Compose
version: "2.2"
services:

Creamos el dns, con el bind9 como hemos hecho siempre
  asir_bind9:
    image: internetsystemsconsortium/bind9:9.16
    ports:
      -53:53
    volumes:
      -dns_conf:/etc/bind
    networks:
      n1:
        ipv4_address: 10.10.1.254
        
Creamos el cliente Kasm, para poder acceder a el desde el navegador
  asir_cliente:
    image: kasmweb/desktop:1.10.0-rolling
    ports: Mapeamos los puertos
      -6901:6901
Para poder usar la contraseña del contenedor
    environment:
      VNC_PW: password
    networks:
      -n1
    dns:
      -10.10.1.254
    stdin_open: true  # docker run -i
    tty: true         # docker run -t
    
Creamos servidor web 
  asir_webb:
    image: httpd:latest
    ports:
      -"80:80"
    networks:
      n1:
        ipv4_address: 10.10.1.5
    volumes: mapeamos los volúmenes
      -apache_index:/usr/local/apache2/htdocs
      -apache_conf:/usr/local/apache2/conf
Creamos contenedor del wireshark
asir_wireshark:
    image: linuxserver/wireshark
    cap_add:  
      - NET_ADMIN
    network_mode: host  
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
    ports:
      - 3000:3000

El wireshark  es accesible desde el navegador a través de localhost:3000. 
Volumenes y redes:
networks:
 n1:
  external: true
volumes:
  dns_conf:
    external: true
  apache_conf:
    external: true
  apache_index:
    external: true

Configuración maquinas
En el servidor web
En htdocs, dos carpetas con index distintos para las dos webs

En el volumen del apache, descomentamos: Include conf/extra/httpd-vhosts.conf

En extra, en el archivo httpd-vhosts.conf, creamos dos virtual host para cada web.
<VirtualHost *:80>
    ServerAdmin webmaster@dummy-host.example.com
    DocumentRoot "/usr/local/apache2/htdocs/pete"
    ServerName adios.ejemplo.com
    ErrorLog "logs/dummy-host.example.com-error_log"
    CustomLog "logs/dummy-host.example.com-access_log" common
</VirtualHost>

<VirtualHost *:80>
    ServerAdmin webmaster@dummy-host2.example.com
    DocumentRoot "/usr/local/apache2/htdocs/lucifer"
    ServerName hola.ejemplo.com
    ErrorLog "logs/dummy-host2.example.com-error_log"
    CustomLog "logs/dummy-host2.example.com-access_log" common
</VirtualHost>

Servidor dns
Abrimos la zona dns con dos cnames:
pete IN CNAME ejemplo
lucifer IN CNAME ejemplo
Reinicio 

En el cliente
Buscamos pete.ejemplo.com y nos aparece la pagina pete.

Buscamos lucifer.ejemplo.com y nos aparece la otra pagina.

Protocolo HTTPS
Comando para crear las claves:
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.cr

Comando cp .key o .crt contenedor:ruta volumen
Ejemplo:

sudo docker cp apache-selfsigned.key asir_webb:/usr/local/apache2/conf

docker cp apache-selfsigned.crt asir_webb:/usr/local/apache2/conf

En el volumen, en el archivo httpd-ssl.conf y en required modules, y en httpd.conf, descomentar las lineas para que funcione el htpps.

Ahora configuramos el virtual host asi:

<VirtualHost _default_:443>
DocumentRoot "/usr/local/apache2/htdocs/lucifer"
ServerName lucifer.prueba.com
ServerAdmin you@example.com
ErrorLog /proc/self/fd/2
TransferLog /proc/self/fd/1
SSLEngine on
SSLCertificateFile "/usr/local/apache2/conf/server.crt"
SSLCertificateKeyFile "/usr/local/apache2/conf/server.key"

Wireshark
Creamos el contenedor de wireshark, accesible a través de localhost:3000.

asir_wireshark:
    image: linuxserver/wireshark
    cap_add:  
      - NET_ADMIN
    network_mode: host  
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
    ports:
      - 3000:3000 #optional


