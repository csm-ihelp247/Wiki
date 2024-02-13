https://www.digitalocean.com/community/tutorials/how-to-install-mariadb-on-debian-11

https://adoptium.net/de/installation/linux/#_deb_installation_on_debian_or_ubuntu


# Pre-install
`hostname suite`

`nano /etc/hosts`
+   127.0.0.1     suite

`nano /etc/hostname`
+   suite



# Importing the Open-Xchange Buildkey

- `wget https://software.open-xchange.com/0xDFD4BCF6-oxbuildkey.pub -O - | apt-key add -`
- `wget https://software.open-xchange.com/0xDFD4BCF6-oxbuildkey.pub`
- `apt-key add - < 0xDFD4BCF6-oxbuildkey.pub`



# Add Open-Xchange Repository

`cat << EOF >> /etc/apt/sources.list.d/open-xchange.list
deb https://software.open-xchange.com/products/appsuite/stable/appsuiteui/DebianBuster/ /
deb https://software.open-xchange.com/products/appsuite/stable/backend/DebianBuster/ / 
EOF`



# Updating repositories and install packages

`apt update && apt-get install open-xchange open-xchange-authentication-database open-xchange-grizzly \
  open-xchange-admin open-xchange-appsuite \
  open-xchange-appsuite-backend open-xchange-appsuite-manifest`



## Add the Open-Xchange binaries to PATH

`echo PATH=$PATH:/opt/open-xchange/sbin/ >> ~/.bashrc && . ~/.bashrc`



# Init ConfigDB

`/opt/open-xchange/sbin/initconfigdb --configdb-pass=(PASSWORD_REPLACE) -a --mysql-root-passwd=(PASSWORD_REPLACE)`



## Installer

`/opt/open-xchange/sbin/oxinstaller --no-license \
--servername=suite --configdb-pass=(PASSWORD_REPLACE) \
--master-pass=(PASSWORD_REPLACE) --network-listener-host=localhost --servermemory 4096`

`systemctl restart open-xchange`



# Register Server

`/opt/open-xchange/sbin/registerserver -n suite -A oxadminmaster -P (PASSWORD_REPLACE)`



# Create and Register FileStore

`mkdir /var/opt/filestore
chown open-xchange:open-xchange /var/opt/filestore`

`/opt/open-xchange/sbin/registerfilestore -A oxadminmaster -P (PASSWORD_REPLACE) \
-t file:/var/opt/filestore -s 1000000`



# Register Database

`/opt/open-xchange/sbin/registerdatabase -A oxadminmaster -P (PASSWORD_REPLACE) \
-n oxdatabase -p (PASSWORD_REPLACE) -m true`



# Configure Apache2 

`a2enmod proxy proxy_http proxy_balancer expires deflate headers rewrite mime setenvif lbmethod_byrequests`

`nano /etc/apache2/conf-available/proxy_http.conf`
- refer to Guide
 https://oxpedia.org/wiki/index.php?title=AppSuite:Open-Xchange_Installation_Guide_for_Debian_11.0

`nano /etc/apache2/sites-enabled/000-default.conf`
- refer to Guide
https://oxpedia.org/wiki/index.php?title=AppSuite:Open-Xchange_Installation_Guide_for_Debian_11.0



# Create Context and User

`/opt/open-xchange/sbin/createcontext -A oxadminmaster -P (PASSWORD_REPLACE) -c 1 \
-u oxadmin -d "Context Admin" -g Admin -s User -p (PASSWORD_REPLACE) -L defaultcontext \
-e oxadmin@example.com -q 1024 --access-combination-name=groupware_standard`


`/opt/open-xchange/sbin/createuser -c 1 -A oxadmin -P (PASSWORD_REPLACE) -u testuser \
-d "Test User" -g Test -s User -p secret -e testuser@example.com \
--imaplogin testuser --imapserver 127.0.0.1 --smtpserver 127.0.0.1`



# Add SSL - 
### https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-ubuntu-20-04

- `apt install certbot python3-certbot-apache`
- `certbot --apache -d (URL_REPLACE)`

