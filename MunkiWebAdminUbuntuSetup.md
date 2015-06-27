# Introduction #

A complete walkthrough for setting up a Munki repo and Munki Web Admin on an Ubuntu Server environment using encrypted connections.

# Assumptions #

This documentation was written using a fresh installation of Ubuntu Server 12.10 with no additional packages or services installed or configured. [VMware Fusion](http://www.vmware.com/products/fusion/overview.html) was used to host the Ubuntu Server and an OS X 10.8.3 test client, but the instructions should work on physical machines as well.

## Installing Required Tools On The Server ##

The major components for this project are

  * Apache - _web server and webdav server_
  * MySQL - _database server for Munki Web Admin, although you can use any Python-supported backend_
    * phpMyAdmin - _optional, but provides a more feature-rich interface for the database_
  * Git - _easiest way to install Munki Web Admin and keep it up to date_
  * SSH Server - _optional, a secure method of accessing your server remotely_

### Update Your Operating System ###

```
sudo apt-get update
sudo apt-get upgrade
sudo reboot
```

### Install Required Software ###

The following command will install two optional packages, 'vim' and 'openssh-server'. While they are optional, it is assumed you will need a command line text editor (although Ubuntu does ship with nano) and presumably a secure way to access the system remotely.

```
sudo apt-get install \
	build-essential \
	git \
	libapache2-mod-wsgi \
	libmysqlclient-dev \
	mysql-server \
	openssh-server \
	phpmyadmin \
	python-dev \
	python-setuptools \
	vim
```

  * **NOTE:** You will be prompted for a root password for MySQL - DO NOT FORGET THIS PASSWORD!!
  * **NOTE:** You want to choose `apache2` when prompted to automatically configure a web server for phpMyAdmin.
  * **NOTE:** Choose "Yes" when asked if you want to configure a database for phpMyAdmin.
    * Next, type in your MySQL root password.
    * Next, leave the password field blank to automatically generate a password for phpMyAdmin.


### Enable mod\_rewrite, mod\_dav, mod\_ssl and virtualenv ###

```
sudo a2enmod rewrite ssl dav_fs
sudo a2ensite default-ssl
sudo apachectl restart
sudo easy_install virtualenv
```

### Create a Database for Munki Web Admin ###
We need to create a user and an empty database for Munki Web Admin to use. You can do so from the command line, but it's easier in phpMyAdmin. Open a browser to **https://example.com/phpmyadmin** and login with `root` and the password that you set during the installation process.

  * Click the `Users` tab _(some versions use `Privileges` instead of `Users`)_
  * Click `Add a new user`
  * Type `munkiwebadmin` for the username (this will also be the database name)
  * Set `Host` to `Local` _which should provide the value `localhost`_
  * Click `Generate` to generate a password. _you should copy this password since you'll need it in the next step_
  * Select `Create a database with the same name and grant all privileges`
  * Click `Create User` and close the dialog window

Next we need to set the default collation for the database:

  * click `munkiwebadmin` from the database list on the left _(you may need to use the reload button just above that list)_
  * click the `Operations` tab
  * select `utf8_general_ci` under the Collation section
  * click the `Go` button

## Munki Web Admin Installation ##

```
cd /usr/local
sudo virtualenv munkiwebadmin_env
sudo chown -R `whoami` munkiwebadmin_env
cd munkiwebadmin_env
source bin/activate
pip install django==1.5.1
pip install django-wsgiserver==0.8.0beta
pip install MySQL-python
git clone https://code.google.com/p/munki.munkiwebadmin/ munkiwebadmin
cd munkiwebadmin
cp settings_template.py settings.py
```

Edit settings.py:

  * Set ADMINS to an administrative name and email
  * Set TIME\_ZONE to the appropriate timezone
  * Under INSTALLED\_APPS uncomment django\_wsgiserver
  * Set MUNKI\_REPO\_DIR to the local filesystem path to your munki repo. (In this case, `/var/www`)
  * Under DATABASES, make the following changes:
    * ENGINE: django.db.backends.mysql
    * NAME: munkiwebadmin
    * USER: munkiwebadmin
    * PASSWORD: _did you copy the password from the previous step?_
  * Make other edits as you feel comfortable


Next, we need to initialize the database:

```
python manage.py syncdb
```
This command will also prompt you to create a default admin user. Do this, and don't forget the username or password!

Stage the static files (Type yes when prompted):

```
python manage.py collectstatic
```

Next we need to generate a .wsgi file to help Apache bootstrap munki web admin. _The following script was slightly modified from its original, which can be found [here](http://www.sensibledevelopment.com/2011/01/a-generic-wsgi-file-for-deploying-django-with-virtualenv-and-mod_wsgi/)._

**/usr/local/munkiwebadmin\_env/munkiwebadmin.wsgi:**

```
import os
import site
import sys

# Remember original sys.path.
prev_sys_path = list(sys.path)

# we add currently directory to path and change to it
pwd = os.path.dirname(os.path.abspath(__file__))
os.chdir(pwd)
sys.path = [pwd] + sys.path
sys.path = [os.path.join(pwd, 'munkiwebadmin')] + sys.path

# find the site-packages within the local virtualenv
for python_dir in os.listdir('lib'):
    site_packages_dir = os.path.join('lib', python_dir, 'site-packages')
    if os.path.exists(site_packages_dir):
        site.addsitedir(os.path.abspath(site_packages_dir))

# Reorder sys.path so new directories at the front.
new_sys_path = []
for item in list(sys.path):
    if item not in prev_sys_path:
        new_sys_path.append(item)
        sys.path.remove(item)
sys.path[:0] = new_sys_path

# now start django
from django.core.handlers.wsgi import WSGIHandler
os.environ['DJANGO_SETTINGS_MODULE'] = 'settings'
application = WSGIHandler()
```


# Apache Configurations #

## Redirect All HTTP Traffic to HTTPS ##

Edit the default site's config file...

**/etc/apache2/sites-enabled/000-default**

```
<VirtualHost *:80>
        RewriteEngine On
        RewriteCond %{SERVER_PORT} !^443$
        RewriteRule ^/(.*) https://%{HTTP_HOST}/$1 [NC,R,L]
</VirtualHost>
```

## Configure The Munki Repo & WebDAV ##
Parts of the Munki repo need to be publicly accessible via your web server, so let's use `/var/www` since Apache is already looking there. The following commands will clean out the existing /var/www directory and create the scaffolding needed by the Munki admin tools.

**WARNING:** _If you have anything in /var/www, make sure it's backed up first!_

```
sudo rm /var/www/*
sudo mkdir /var/www/pkgsinfo \
	/var/www/catalogs \
	/var/www/manifests \
	/var/www/pkgs
sudo chown -R www-data /var/www/
sudo chmod -R a+rX /var/www/
```

The munki admin tools only work on OS X systems. Therefore, we need a way to access the /var/www directory from an OS X client. We'll use WebDAV for this. Since Munki Web Admin and the WebDAV share will both be served by Apache, all of the files in your Munki repo will be owned by www-data. This isn't a necessity, but it certainly helps to keep things running smoothly when it comes to permissions and ownership issues.

**/etc/apache2/mods-enabled/dav\_fs.conf**
```
DAVLockDB ${APACHE_LOCK_DIR}/DAVLock
Alias /munki_repo /var/www/
<Location /munki_repo>
        DAV On
        SSLRequireSSL
        Options None
        AuthType Basic
        AuthName "Munki Repo"
        AuthUserFile /etc/apache2/htpasswd
        <LimitExcept GET OPTIONS>
                Order allow,deny
                Allow from all
                Require valid-user
        </LimitExcept>
</Location>
```

Add a local 'admin' user. If you have multiple Munki administrators, you should probably [setup LDAP authentication](http://blogger.ziesemer.com/2010/12/ldap-authentication-apache-http-server.html) for your WebDAV share by editing the above example instead.

```
sudo htpasswd -c /etc/apache2/htpasswd admin
```

_Technically, WebDAV is completely configured at this point. However, I would not recommend testing it until we generate a custom SSL certificate._


## Custom SSL Settings ##

You might get lucky and your snakeoil certificates are already setup with the proper Common Name value, but many times that's not the case. Therefore, we'll need to generate a new certificate:

```
sudo openssl req \
	-x509 \
	-nodes \
	-days 3652 \
	-newkey rsa:2048 \
	-keyout /etc/ssl/private/munkiwebadmin.key \
	-out /etc/ssl/certs/munkiwebadmin.crt
```

**IMPORTANT:** _When specifying the Common Name for the certificate, be absolutely sure to use the same FQDN or IP address that you'll be setting in your munki clients' SoftwareRepoURL key. Failing to do so will prevent munki (curl, really) from trusting your repo._


All that's left now is to edit `/etc/apache2/sites-enabled/default-ssl`. It should look something like this:

**/etc/apache2/sites-enabled/default-ssl**

```
<IfModule mod_ssl.c>
<VirtualHost _default_:443>
	ServerAdmin webmaster@localhost
	DocumentRoot /var/www

	# Base configuration
	Alias /catalogs/ /var/www/catalogs/
	Alias /manifests/ /var/www/manifests/
	Alias /pkgs/ /var/www/pkgs/
	<Directory />
		Options FollowSymLinks
		AllowOverride None
	</Directory>

	# Munki Web Admin
	Alias /static/ /usr/local/munkiwebadmin_env/munkiwebadmin/static/
	<Directory /usr/local/munkiwebadmin_env/munkiwebadmin/static>
		Order deny,allow
		Allow from all
	</Directory>
	WSGIScriptAlias / /usr/local/munkiwebadmin_env/munkiwebadmin.wsgi
	<Directory /usr/local/munkiwebadmin_env>
		<Files munkiwebadmin.wsgi>
			Order allow,deny
			Allow from all
		</Files>
	</Directory>


	# Munki Repo
	<Directory /var/www/>
		Options FollowSymLinks
		AllowOverride None
		Order allow,deny
		allow from all
	</Directory>

	# Logging
	ErrorLog ${APACHE_LOG_DIR}/error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/ssl_access.log combined
	SSLEngine on
	SSLCertificateFile    /etc/ssl/certs/munkiwebadmin.crt
	SSLCertificateKeyFile /etc/ssl/private/munkiwebadmin.key
</VirtualHost>
</IfModule>
```

…and of course, `sudo apachectl restart` should bring it all together.


# Setting Up Munki Admin Tools #

### Mount The Munki Repo Over WebDAV ###
On the OS X machine that has the [latest munki tools](http://munkibuilds.org/) installed, click "Go" -> "Connect To Server…" from Finder's menu. Type the FQDN of your server, or your server's IP address followed by "/munki\_repo". e.g. **https://example.com/munki_repo**.

When prompted, authenticate using the credentials you setup with the htpasswd command.

**IMPORTANT:** You'll need to mount this volume anytime you want to use the munkitools to administer the contents of your munki repo.

### Configure the munki admin tools ###
Still on your OS X client, run `/usr/local/munkiimport --configure` and make sure the "Path to munki repo" is set to `/Volumes/munki_repo`. Once you've done this, you can test that your munki admin tools are happy by running `sudo /usr/local/munki/makecatalogs`.


# Configuring Clients #

As of 2013/03/23, Munki Web Admin comes with a tool that produces a package installer suitable for your Munki Web Admin installation. There are no files to edit, just a few questions to answer. But first, you need a copy of those scripts on a machine running OS X 10.6 or higher:

```
scp -r mwa-admin@example.com:/usr/local/munkiwebadmin_env/munkiwebadmin/scripts munkiwebadmin-scripts
cd munkiwebadmin-scripts
```

Now you can create the package installer:

```
./create-mwa-scripts-installer.sh
```

You will be asked a few questions before the script will create the installer for you. Here are some examples of what you might see:

  * **Enter the FQDN of your MWA server:** https://example.com
  * **Specify allowed network prefixes:** 192.168 172.16

Only the first question requires an answer. The second provides information that the preflight script can use to prevent Munki from running when the client's IP address doesn't start with one of the values specified. In this example case, Munki will only run when the client IP is 192.168.x.x or 172.16.x.x. If you leave this question blank, Munki will run regardless of the client's IP address(es).

In this example setup, the script has automatically grabbed the SSL certificate from https://example.com and embedded it into the resulting package installer. All you need to do now is deploy that package to all of your clients.

## Configuring Clients, Part 2 ##

The above steps will make sure the clients can communicate with Munki Web Admin, but they will not be able to access the Munki repo! To fix this, you'll need to execute the following commands on each client, probably best to do this in a postinstall\_script:

```
#!/bin/bash
source /usr/local/munki/munkiwebadmin-config
defaults write /Library/Preferences/ManagedInstalls SoftwareRepoURL "${MWA_HOST}"
defaults write /Library/Preferences/ManagedInstalls SoftwareRepoCACertificate "${MWA_SSL_CERTIFICATE}"
exit 0
```

_The line beginning with "source" will allow you to use the $MWA\_HOST and $MWA\_SSL\_CERTIFICATE variables as defined in that file. You can hard code the values, but using variables reduces the likelihood of typos and ensures you can use the same postinstall\_script should your server's FQDN or certificate name ever change._


# Summary #
At this point you should have Munki Web Admin installed along with a repository for Munki and a means to administer that repository, all over SSL. We went through a lot though, so here are some important paths and URLs for you to remember:

  * **https://example.com** - _your munki web admin user interface_
  * **https://example.com/munki** - _your WebDAV access to munki's repo_
  * **/var/www** - _path to munki's repo on the server_
  * **/usr/local/munkiwebadmin\_env** - _The munki web admin application_
  * **https://example.com/phpmyadmin** - _A robust interface for the MySQL database. Also a very easy way to export munki web admin's data to other formats._