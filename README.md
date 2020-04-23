# Installation magento 2

### I. Intro
![](https://img.shields.io/github/license/HeidiSQL/HeidiSQL.svg?style=flat)
#### 1.1- Requeierement:

1. OS: Linux Ubuntu v18
2. web server: Nginx v1.17
3. Mysql: mysql server v14
4. Magento v2.3.2
5. generator password: https://www.lastpass.com/password-generator


 

### II. Preparation

1. connect as root
```
sudo -i
```

2. mettre a jour system
```
apt update && apt upgrade
```

3. Creation user et le dossier html
3.1 Creation user `magento`
```
adduser magento
```
3.2 Creation dossier `html`
```
mkdir /home/magento/html
```
3.3. modifier les droits
```
chown -R magento:magento /home/magento/html
```

4. Creation Access Keys<br>
4.1 Connecter (Inscrit) sur <a href="https://marketplace.magento.com" target="_blank"><b>Magento Marketplace</b></a>.<br>
4.2 Allez sur <b>votre compte</b> (Haut droit) et choisi <b>My Profile</b>.<br>
4.3 Cliqur sur <b>Access Keys</b> (tab Marketplace).<br>
![Image description](docs/static/1.png)<br>
4.4 Cliqur sur <b>Create A New Access Key</b> (Verrifier que vous etes toujours sur tab magento 2).<br>
4.5 Ecrivez un nom de votre cle puis OK.<br>
4.6 Gardez les <b>Public Key</b> et <b>Private Key</b>

5. installation zip
```
apt-get install zip unzip
```

### III- Installation FTP Server

##### Installation
installer vsftp
```
apt install vsftpd -y
```


##### Configuration
prendre une copie de fichier de config
```
cp /etc/vsftpd.conf /etc/vsftpd.conf.orig
```
ouvrer le fichier de config sur nano
```
nano /etc/vsftpd.conf
```


##### Tester
```
ftp -p localhost
```
> *output*<br>root@ubuntu-s-4vcpu-8gb-fra1-01:/# ftp -p localhost<br>
Connected to localhost.<br>
220 (vsFTPd 3.0.3)<br>
Name (localhost:root): magento<br>
331 Please specify the password.<br>
Password:<br>
230 Login successful.<br>
Remote system type is UNIX.<br>
Using binary mode to transfer files.<br>
ftp>

quiter la session
```
bye
```
##### securite


Now, we need to restart the server for the changes to take effect:
```
systemctl restart vsftpd
```

### IV- Installation PHP & extensions




Installez le package <b>software-properties-common</b> pour nous fournir le package <b>add-apt-repository</b>
```
apt-get install software-properties-common
```

mettre à jour votre système avec des packages non pris en charge à partir de <b>ppa</b>
```
add-apt-repository ppa:ondrej/php
```

Mettre à jour les packages
```
apt-get update
```


`
[Optionnel] La commande apt-cache search peut effectuer des recherches de packages (si vous avez d'autres packages a installer).
`
```
apt-cache search php7.2
```

install les packages
```
apt-get install php7.2 libapache2-mod-php7.2 php7.2-common php7.2-gd php7.2-mysql php7.2-curl php7.2-intl php7.2-xsl php7.2-mbstring php7.2-zip php7.2-bcmath php7.2-soap php-xdebug php-imagick
```

afficher la versin du PHP installer
```
php -v
```

a la fin de l'installation, on trouves que apache est installe<br>pour le supprimer

```
apt-get remove apache2
```

netoyage total
```
apt-get purge apache2
```

### V- Installation et configuration du PHP-FPM (<a href="https://www.php.net/manual/en/install.fpm.php ">FastCGI Process Manager</a>)

##### Installation

Install PHP 7.2 FPM:
```
apt-get install php7.2-fpm
```

##### Configuration

prendre une copie du fichier de config
```
cp /etc/php/7.2/fpm/php.ini /etc/php/7.2/fpm/php.ini.backup
```

Ouvrir le fichier de config sur nano<br>
```
nano /etc/php/7.2/fpm/php.ini
```

avec la commande <b>Ctrl + W</b> on fait des recherches sur le fichier, et modifier les valeurs:
- memory_limit = 2G<br>
- max_execution_time = 3600<br>
- max_input_time = 1800<br>
- upload_max_filesize = 10M<br>
- zlib.output_compression = On<br>

`Ctrl + W: recherche`<br>
`Ctrl + O: sauvegarde`<br>
`Ctrl + X: quit`<br>

Redemarrer le service
```	
service php7.2-fpm start
```	
	

### VI- Installation Composer

installation
```
curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
```

> All settings correct for using Composer<br>Downloading...<br><br>Composer (version 1.10.5) successfully installed to: /usr/local/bin/composer<br>Use it: php /usr/local/bin/composer

```
composer --version
```

> Do not run Composer as root/super user! See https://getcomposer.org/root for details<br>Composer version 1.10.5 2020-04-10 11:44:22

### VII- Installation Firewall

UFW is installed by default on Ubuntu. If it has been uninstalled for some reason, you can install it with:
```
apt install ufw
```

7.1. Regle 1: *refuser* tout connexion entrant, *accepter* tout connexion sortant 

```
ufw default deny incoming
ufw default allow outgoing
```

7.2. Regle 2: autorise la connexion SSH
```
ufw allow 22/tcp
```

7.3. Regle 3: autorise la connexion FTP
```
ufw allow 21/tcp
```

7.4. Regle 4: autorise la connexion HTTP
```
ufw allow 80/tcp
```

7.5. Regle 5: autorise la connexion HTTPS
```
ufw allow 443/tcp
```

status du firewall
```
ufw status
```

si la resultat:
> Status: inactive

demarrer firewall
```
ufw enable
```

### VIII- Installation web server: `nginx`

Importer la clé de signature du package et ajoutez-la à `apt`:
```
cd /tmp/ && wget http://nginx.org/keys/nginx_signing.key
sudo apt-key add nginx_signing.key
sudo sh -c "echo 'deb http://nginx.org/packages/mainline/ubuntu/ '$(lsb_release -cs)' nginx' > /etc/apt/sources.list.d/Nginx.list"
```
installation:
```
sudo apt-get update
sudo apt-get install nginx
```

afficher la version:
```
nginx -v
```
> *Output*<br>nginx version: nginx/1.17.10

Assurez-vous que NGINX est en cours d'exécution et activé pour démarrer automatiquement lors des redémarrages:
```
sudo systemctl start nginx
sudo systemctl enable nginx
```

### IX- Installation MySql Server


#### installation

installer mysql server
```
apt install mysql-server
```

Améliorez la sécurité de votre mysql server
```
mysql_secure_installation
```

redemarrer service mysql
```
systemctl restart mysql.service
```

#### Configuration


#### Compte 

ouvrer service mysql
```
mysql
```
> *output* <br>mysql>

creation nouveau compte mysql
```
CREATE USER 'magento'@'localhost' IDENTIFIED BY 'your-password';
GRANT ALL PRIVILEGES ON magento.* TO 'magento'@'localhost';
FLUSH PRIVILEGES;
```

creation de la  base de donnees `magento`
```
CREATE DATABASE magento;
```
> *output* <br>Query OK, 1 row affected (0.00 sec)

### X- Installation Magento

#### permission

positionner sur le dossier
```
cd /home/magento/html
```
modifier les permissions
```
find var generated vendor pub/static pub/media app/etc -type f -exec chmod g+w {} +
find var generated vendor pub/static pub/media app/etc -type d -exec chmod g+ws {} +
chown -R :www-data .
chmod u+x bin/magento
```

#### configuration nginx

user  www-data;

backup des fichier de config
```
cp /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.back
cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf-back
```

```
nano /etc/nginx/nginx.conf
```
remplacer la premiere ligne
```
user  nginx;
```
par
```
user  www-data;
```
`sauvegarder` et `quiter`


vider le contenu de `default.conf`
```
echo "" > /etc/nginx/conf.d/default.conf
```

ouvrer `default.conf`  pour modification
```
nano /etc/nginx/conf.d/default.conf
```

> *put*<br>upstream fastcgi_backend {<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;server unix:/run/php/php7.2-fpm.sock;<br>}<br><br>server {<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;listen 80;<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;server_name localhost;<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;set $MAGE_ROOT /home/magento/html;<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;include /home/magento/html/nginx.conf.sample;<br>}<br>

tester la configuration
```
nginx -t
```

redemarrer `nginx`
```
service nginx reload
```




### XI- Installation PhpMyAdmin

### XII- Installation Java

### XIII- Installation ElasticEearch

### XIX- Installation Certbot


9.1. Add Certbot PPA
You'll need to add the `Certbot PPA` to your list of repositories. To do so, run the following commands on the command line on the machine:
```
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository universe
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
```

9.2. Install Certbot
Run this command on the command line on the machine to install Certbot.
```
sudo apt-get install certbot python-certbot-nginx
```

9.3. Choose how you'd like to run Certbot

Run this command to get a certificate and have Certbot edit your Nginx configuration automatically to serve it, turning on HTTPS access in a single step.
```
sudo certbot --nginx
```
