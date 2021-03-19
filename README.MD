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

      $ sudo apt update && sudo apt upgrade
      
      $ sudo apt instal nginx
      
  B) installations sur la machine wordpress:

      $ sudo apt update && sudo apt upgrade
      
      $ sudo apt install nginx
      
      $ cd /var/www/html/

      $ wget https://wordpress.org/latest.tar.gz

      $ tar -xvzf latest.tar.gz
      
 # configuration de Wordpress:
 
      $ cd wordpress

      $ cp wp-config-sample.php wp-config.php
      
 # configuration du nginx:
    
