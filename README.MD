                                 ### TP_Archi-3-Tiers ###
 
 
 
# prérequis:

  - Vagrant
                                          
# création de l'infrastructure

      $ sudo git clone https://GregLebreton/TP_Archi-3-Tiers.git

      $ cd TP_Arch-3-Tiers 

      $ sudo vagrant up
    
# connection SSH aux machines:

      $ sudo vagrant ssh nginxserver

      $ sudo vagrant ssh wordpress

      $ sudo vagrant ssh database
      
# sur votre machine locale, renseigner dans le fichier /etc/hosts le nom de domaine avec l'IP de la machine correpondant au server d'entrée Nginx:

          $ sudo nano /etc/hosts

             192.168.60.2  monsite.com www.monsite.com


# installation sur les machines:
      
  A) installation sur la machine nginxserver:
  
   1) installation du package nginx:

          $ sudo apt update && sudo apt upgrade -y

          $ sudo apt install nginx -y
      
   2) configuration du nginx:

          $ sudo touch /etc/nginx/sites-available/reverse_proxy_nginx.conf
      
          $ sudo nano /etc/nginx/sites-available/reverse_proxy_nginx.conf

              # Le reverse proxy en simple http nécessite seulement d'écouter sur le port 80 et de rediriger par le proxy_pass
              # vers la machine contenant le Wordpress
              server {
                  listen 80;
                  location / {
                    proxy_pass http://www.monsite.com;
                    }
                  }
                  
   3) creation du lien symbolique pour activer la config dans les sites activés:

          $ sudo ln -s /etc/nginx/sites-available/reverse_proxy_nginx.conf /etc/nginx/sites-enabled/reverse_proxy_nginx.conf
                  
   4) on ajoute la variable d'envirronnement dans le fichier /etc/hosts pour y ajouter la redirection vers la machine wordpress

          $ sudo nano /etc/hosts
          
              192.168.60.3  monsite.com www.monsite.com

   5) suppréssion de la configuration par défaut

          $ sudo unlink /etc/nginx/site-enabled/default
                  
   6) on redémarre Nginx

          $ sudo systemctl restart nginx
      
      
  B) installations sur la machine wordpress:
  
   1) installation du package wordpressb et du serveur nginx:

          $ sudo apt update && sudo apt upgrade -y

          $ sudo apt install nginx -y

          $ cd /var/www/html/

          $ sudo wget https://wordpress.org/latest.tar.gz

          $ sudo tar -xvzf latest.tar.gz && sudo rm -rf latest.tar.gz
          
          $ sudo apt install php7.3-fpm php7.3-mysql -y
      
   2) configuration du Wordpress:
 
          $ cd wordpress

          $ sudo cp wp-config-sample.php wp-config.php

          $ sudo chown -R www-data:www-data /var/www/html/wordpress

          $ sudo chmod 755 /var/www
          
          $ sudo nano /var/www/html/wordpress/wp-config.php

              // ** MySQL settings - You can get this info from your web host ** //
              /** The name of the database for WordPress */
              define( 'DB_NAME', 'wpdb' );

              /** MySQL database username */
              define( 'DB_USER', 'wpuser' );

              /** MySQL database password */
              define( 'DB_PASSWORD', 'dbpassword' );

              /** MySQL hostname */
              define( 'DB_HOST', '192.168.60.4' );

                 
   3) configuration du nginx de Wordpress:
 
          $ sudo nano /etc/nginx/sites-available/wordpress_nginx.conf

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
                      
   4) creation du lien symbolique pour activer la config dans les sites activés:

          $ sudo ln -s /etc/nginx/sites-available/wordpress_nginx.conf /etc/nginx/sites-enabled/wordpress_nginx.conf
          
   5) suppréssion de la config par défaut

          $ sudo unlink /etc/nginx/site-enabled/default
                      
   6) redémarrage de Nginx
                      
          $ sudo systemctl restart nginx
          
      
  C) installation sur la machine database:
  
   1) installation de MariaDB

          $ sudo apt update && sudo apt upgrade -y
    
          $ sudo apt-get install software-properties-common dirmngr -y

          $ sudo apt-key adv --fetch-keys 'https://mariadb.org/mariadb_release_signing_key.asc'

          $ sudo add-apt-repository 'deb [arch=amd64,arm64,ppc64el] http://mariadb.mirrors.ovh.net/MariaDB/repo/10.5/debian buster main'

          $ sudo apt update

          $ sudo apt install mariadb-server mariadb-client -y
          
   2) configuration Mariadb

          
          # ici on créer des droits pour un utilisateur en local et venant de la machine Wordpress
          $ sudo mysql
    
              CREATE DATABASE wpdb;
              CREATE USER "wpuser"@"localhost" identified by "dbpassword";
              GRANT ALL PRIVILEGES ON wpdb.* TO "wpuser"@"localhost";
              CREATE USER "wpuser"@"192.168.60.3" identified by "dbpassword";
              GRANT ALL PRIVILEGES ON wpdb.* TO "wpuser"@"192.168.60.3";
              FLUSH PRIVILEGES;
              exit
              
   * modifier la configuration de MariaDB pour qu'elle accepte les connections venant de la machine Wordpress
              
          $ sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
           
   * trouver la ligne avec la directive bind-address = 127.0.0.1 et remplacer la ligne par: 
   
          #bind-address = 127.0.0.1  
          
   * sécuriser MariaDB en supprimant les failles de sécurité

          $  sudo mysql_secure_installation
        note: mot de passe root machine database: vagrant
           
   * répondre yes aux questions en renseignant un mot de passe root

   3) redémarrage de MariaDB

          $ sudo systemctl restart mysql
 
# rendez vous à l'adresse www.monsite.com sur votre navigateur de la machine locale    
       
      
