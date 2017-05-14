Playbook ansibleplaybook-wordpress-vhosts-dns
=============================================

This playbook will create a virtual host for your wordpress site, perform basic configuration and set permissions to the virtual host directory correctly.  It will also setup a dns entry for your local dns service.

Requirements
------------
  - DDNS server setup, it is beyond the scope of this document to explain how to do this. Do some google searches on [bind setup dynamic dns](https://wiki.debian.org/DDNS) and it should give you what you need.
  - Apache: the script assumes that the apache config files are:
     apache config: `/etc/apache2`
     apache log files: `/var/log/vhosts`
  - Mysql: is local to the server the script is being run from.

all.yml Variables
-----------------
This file will hold all of your systems settings for your system. Make sure you modify it to suite your needs, otherwise the script will fail.

Dependencies
------------
  - Configured local dns server with dynamic updates activated.
  - DDNS key is setup for the zone.
  - User has write access to `/var/www/`
  - User has sudo access to restart apache.
  - [nsupdate](https://docs.ansible.com/ansible/nsupdate_module.html)
  - [mysqldb](http://docs.ansible.com/ansible/mysql_db_module.html)
  - [mysql_user](http://docs.ansible.com/ansible/mysql_user_module.html)

Setup the playbook
------------------
You will need to copy the distribution version of the `all.yml.dist` file to `all.yml`, then configure it to suite your system.

    $ cp group_vars/all.yml.dist group_vars/all.yml
    $ nano group_vars/all.yml

Running the  Playbook
---------------------
The playbook is designed to be run from the command line with only one variable for the site you want to create.
Example normal run:

    $ ansible-playbook playbook.yml -i hosts -e "site_name=peter"

If you find that something is not right with the variables you can dump all the variables out with.

   $ ansible-playbook playbook.yml -i hosts -e "site_name=peter dump=true"

Directories and Files Created
-----------------------------
apache files/directories:

    file: /etc/apache2/sites-available/{{ site_name }}.web-dev.conf
    directory: "/var/www/{{ site_name }}.web-dev/public_html"
    directory: "/var/log/vhosts/{{ site_name }}.web-dev"

vhost:

    <VirtualHost *:80>
         ServerAdmin webmaster@{{ site_name }}.web-dev
         ServerName {{ site_name }}.web-dev.skynet
         ServerAlias www.{{ site_name }}.web-dev.skynet
         DocumentRoot /var/www/{{ site_name }}.web-dev/public_html/
         <Directory "/var/www/{{ site_name }}.web-dev/public_html/">
                Options FollowSymLinks
            AllowOverride All
              Require all granted
         </Directory>
         ErrorLog /var/log/vhosts/{{ site_name }}.web-dev/error.log
         CustomLog /var/log/vhosts/{{ site_name }}.web-dev/access.log combined
    </VirtualHost>


Special Note about nsupdate & bind:
-----------------------------------
This ansible module is not very helpful with errors, if its not working correctly, tail your system log file and re-run the script and see what bind is saying about the error.

    $ tail -f /var/log/syslog

Also make sure the bind has write permissions to the /etc/bind directory else DDNS will not work correctly and drive you close to insanity :)

License
-------

BSD

Author Information
------------------

[Peter Tsiampas](https://peter.tsiampas.com) - All rounded IT guy.
