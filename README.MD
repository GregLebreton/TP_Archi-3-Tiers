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
      
      
      ...
      
      
      $ sudo systemctl restart nginx
      
      
  B) installations sur la machine wordpress:
  
    * installation du package wordpress:

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
      
    3) configuration du nginx:
 
      $ sudo nano /etc/sites-available/wordpress_nginx.conf
      
      ...
      
  C) installation sur la machine database:
  
      $ sudo apt-get install software-properties-common dirmngr

      $ sudo apt-key adv --fetch-keys 'https://mariadb.org/mariadb_release_signing_key.asc'

      $ sudo add-apt-repository 'deb [arch=amd64,arm64,ppc64el] http://mariadb.mirrors.ovh.net/MariaDB/repo/10.5/debian buster main'

      $ sudo apt update

      $ sudo apt install mariadb-server mariadb-client -y
    
      
      

       
       
       
    
