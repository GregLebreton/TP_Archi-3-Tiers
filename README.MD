                                    ### TP_Archi-3-Tiers ###
 
 
 
# prérequis:

  - Vagrant
                                          
# l'infrastructure

      $ sudo git clone https://GregLebreton/TP_Arch-3-Tiers.git

      $ cd vagrant 

      $ sudo vagrant up
    
# connection SSH aux machines:

      $ sudo vagrant ssh nginxserver

      $ sudo vagrant ssh wordpress

      $ sudo vagrant ssh database
      
# installation sur les machines:
      
  A) installation sur la machine nginxserver:
  
   1) installation du package nginx:

          $ sudo apt update && sudo apt upgrade

          $ sudo apt install nginx
      
   2) configuration du nginx:
      
          $ sudo nano /etc/sites-available/reverse_proxy_nginx.conf

              # Le reverse proxy en simple http nécessite seulement d'écouter sur le port 80 et de rediriger par le proxy_pass
              # vers la machine contenant le Wordpress
              server {
                  listen 80;
                  location / {
                    proxy_pass http://192.168.60.3;
                    }
                  }

          $ sudo systemctl restart nginx
      
      
  B) installations sur la machine wordpress:
  
   1) installation du package wordpress:

          $ sudo apt update && sudo apt upgrade

          $ sudo apt install nginx

          $ cd /var/www/html/

          $ wget https://wordpress.org/latest.tar.gz

          $ tar -xvzf latest.tar.gz
      
   2) configuration du Wordpress:
 
          $ cd wordpress

          $ cp wp-config-sample.php wp-config.php

          $ sudo chown -R www-data:www-data /var/www/html/wordpress

          $ sudo chmod 755 /var/www
      
   3) configuration du nginx de Wordpress:
 
          $ sudo nano /etc/sites-available/wordpress_nginx.conf

              # Ce nginx va recevoir les requête du nginx reverse proxy et va servir le wordpress
              # contenu dans le repertoire racine définit dans "root"
              server {
                  listen 80;
                  root /var/www/html/wordpress;
                  index index.php index.html index.htm;
                  server_name _;

                  access_log /var/log/nginx/wordpress_access.log;
                  error_log /var/log/nginx/wordpress_error.log;

                  client_max_body_size 64M;

                  location / {
                      try_files $uri $uri/ /index.php?$args;
                      }

                  location ~ \.php$ {
                      try_files $uri =404;
                      include /etc/nginx/fastcgi_params;
                      fastcgi_read_timeout 3600s;
                      fastcgi_buffer_size 128k;
                      fastcgi_buffers 4 128k;
                      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                      fastcgi_pass unix:/run/php/php7.3-fpm.sock;
                      fastcgi_index index.php;
                          }

                      }
      
  C) installation sur la machine database:
  
   1) installation de MariaDB
    
          $ sudo apt-get install software-properties-common dirmngr -y

          $ sudo apt-key adv --fetch-keys 'https://mariadb.org/mariadb_release_signing_key.asc'

          $ sudo add-apt-repository 'deb [arch=amd64,arm64,ppc64el] http://mariadb.mirrors.ovh.net/MariaDB/repo/10.5/debian buster main'

          $ sudo apt update

          $ sudo apt install mariadb-server mariadb-client -y
          
   2) configuration Mariadb

          $ sudo mysql
    
              CREATE DATABASE wpdb;
              CREATE USER "wpuser"@"localhost" identified by "dbpassword";
              GRANT ALL PRIVILEGES ON wpdb.* TO "wpuser"@"localhost";
              CREATE USER "wpuser"@"192.168.60.3" identified by "dbpassword";
              GRANT ALL PRIVILEGES ON wpdb.* TO "wpuser"@"192.168.60.3";
              FLUSH PRIVILEGES;
              exit
  ```
  ```
              
           $ sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
           
   * trouver la ligne avec la directive bind-address = 127.0.0.1 et remplacer la ligne par: #bind-address = 127.0.0.1     

           $  mysql_secure_installation
           
   * choisir yes aux questions en ajoutant un mot de passe root    

     
           
           
           
    
      
      

       
       
       
    
