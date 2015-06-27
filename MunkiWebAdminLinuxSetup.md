# Setting up MunkiWebAdmin on CentOS/RHEL 6 #

This is a guide walking you through a basic setup of MunkiWebAdmin on a Linux box.

This guide was most recently tested on 11 July 2013 on CentOS 6.4.

Basic assumption: you host a munki repo on a Linux box and want to add MunkiWebAdmin to an existing setup.

This setup demonstrates basic functionality only. It may not be optimal for your environment. See external documentation on implementing production Django applications for more information.

These directions are for configuring the MunkiWebAdmin Django app for use with Apache and mod\_wsgi. This process was tested on CentOS 6.

Original setup documentation by Timothy Sutton.

## Install Prerequisites ##

### Setup the Virtual Environment ###

Make sure git is installed:
```
  which git
```
If it isn't, install it:
```
 yum install git
```

We'll use git later to grab the MunkiWebAdmin files.

Install python setup tools:
```
  yum install python-setuptools
```
Make sure virtualenv is installed:
```
  virtualenv --version
```
If it isn't, install it:
```
 easy_install virtualenv
```

MunkiWebAdmin's only Python dependency is the Django web framework. It is best practice to use a virtual environment for MunkiWebAdmin, which will provide a complete self-contained directory for MunkiWebAdmin and Django, as well as any other packages that the admin would like to keep local to MunkiWebAdmin. A virtual environment isn't strictly required – the alternative is to install Django in the system's site-packages area, but this guide will assume you are using a virtualenv. It also assumes that you at least have Apache installed and tested working on your server.

### Create a non-admin service account and group ###

Create the munkiwebadmin user:
```
  useradd munkiwebadmin
```

Create the munki group:
```
  groupadd munki
```
Add munkiwebadmin to the munki group:
```
usermod -g munki munkiwebadmin
```
Make sure the munki group has the proper permissions to your munki repo:
```
chgrp -R munki /Path/To/munki_repo
chmod -R g+rw /Path/To/munki_repo
```

### Create the virtual environment ###

When a virtualenv is created, pip will also be installed to manage a virtualenv's local packages.
Create a virtualenv which will handle installing Django in a contained environment. In this example we'll create a virtualenv for MunkiWebAdmin at /usr/local. This should be run from Bash, as this is what the virtualenv activate script expects.

Go to where we want to install the virtualenv:
```
  cd /usr/local
```
Create the virtualenv that munkiwebadmin will live in:
```
  virtualenv munkiwebadmin_env
```
Make sure munkiwebadmin has the proper permissions to the new munkiwebadmin\_env folder:
```
  chown -R munkiwebadmin munkiwebadmin_env
```
Switch to the service account:
```
  su munkiwebadmin
```

Go into the newly minted virtualenv and activate it (Must be done from **bash**):
```
  cd munkiwebadmin_env
  source bin/activate
```

Install Django in the virtual environment:
```
  pip install django==1.5.1
```
**As of this writing, MunkiWebAdmin is compatible with Django 1.5.1. Your results with other Django versions may vary.**

"Activating" the virtualenv inserts its 'bin' path at the front of the shell user's path. If the virtualenv isn't activated, pip or easy\_install would attempt to install packages in the system's 'site-packages' directory. The deactivate command will call a function to reverse the path manipulation performed by activate. The virtualenv should remain activated until the app is up and tested, however.

### Copy and Configure MunkiWebAdmin ###

Still working inside the munkiwebadmin\_env virtual environment, use git to clone the current MunkiWebAdmin code:
```
  git clone https://code.google.com/p/munki.munkiwebadmin/ munkiwebadmin
```
This clones the current MunkiWebAdmin code into a directory named "munkiwebadmin".

Make a copy of the settings\_template.py -- this is a template file; you must copy it to the correct name and edit it.
```
  cd munkiwebadmin
  cp settings_template.py settings.py
```

Edit settings.py:

  * Set ADMINS to an administrative name and email
  * Set TIME\_ZONE to the appropriate timezone
  * Set MUNKI\_REPO\_DIR to the local filesystem path to your munki repo.

You can make more changes later (for example, attempting to configure LDAP authentication).

### More setup ###

We need to run Django's management command to initialize the app's database and create an admin user to log into the web interface.  Still in the 'munkiwebadmin' directory, run:
```
  python manage.py syncdb
```
This command will also prompt you to create a default admin user. Do this, and don't forget the username or password!

Stage the static files (Type yes when prompted):
```
  python manage.py collectstatic
```


### Testing the web service ###

We can now test running the app using Django's built-in webserver:
```
  python manage.py runserver 0.0.0.0:8000
```

In this example, the '0.0.0.0' specifies that it will respond to requests from any IP address, and run on port 8000. You can now connect to the instance's address at this port from a web browser. Verify that you can log in. You won't see any data in reports or inventory until you configure a test client with the scripts to report to the server.

You should be able to look at catalogs; look at and edit manifests; and access the site admin tools.

### Installing mod\_wsgi and configuring Apache ###

Django's built-in web server is fine for initial testing, but is unsuitable for use with more than a handful of clients. For production use, we'll configure Apache and mod\_wsgi to serve the web app.

To install mod\_wsgi, it's recommended to use the distribution's package manager for this. See the [mod\_wsgi installation instructions](http://code.google.com/p/modwsgi/wiki/InstallationInstructions) for more details.

Make sure you exit the munkiwebadmin user and go back to root before continuing.

Install mod\_wsgi using the following command:
```
yum install mod_wsgi
```
After that is in place, create the wsgi directory:
```
mkdir /var/run/wsgi
```
Make sure that the munkiwebadmin user has read/write access to it (along with root for good measure):
```
  chown munkiwebadmin:root /var/run/wsgi
```

_Note: It would be best to use a group for this (such as modwsgiusers), but you can add munkiwebadmin directly if desired._

### Set up an Apache VirtualHost ###

Now lets set up our VirtualHost to use mod\_wsgi to serve the app.  This is typically found in /etc/httpd/conf/httpd.conf.  Your VirtualHost details may vary, but the important directives are shown below.

Before we setup our VirtualHost file, put the following line into the general config for httpd.conf:
```
  WSGISocketPrefix /var/run/wsgi
```
Now enter the following virtual host into httpd.conf:
```
<VirtualHost *:80>
   WSGIScriptAlias / /usr/local/munkiwebadmin_env/munkiwebadmin/munkiwebadmin.wsgi
   WSGIDaemonProcess munkiwebadmin user=munkiwebadmin group=munki
   Alias /static/ /usr/local/munkiwebadmin_env/munkiwebadmin/static/
   <Directory /usr/local/munkiwebadmin_env/munkiwebadmin>
       WSGIProcessGroup munkiwebadmin
       WSGIApplicationGroup %{GLOBAL}
       Order deny,allow
       Allow from all
   </Directory>
</VirtualHost>
```

WSGIScriptAlias is similar to the Alias directive in Apache, except we are giving the path to a Python script (by convention ending in '.wsgi') that will be executed to load the app.

WSGIDaemonProcess lets us define the name of the daemon process, as well as extra arguments about the user and group running the process, as well as working directory, etc. In this case we care mainly about specifying a user who will have read access to the virtualenv, and write access to at least the SQLite database and optionally the Munki repo's manifest files, if editing manifests is desired.  It is strongly suggested that you use a non-admin service account to run munkiwebadmin.

The app still contains static files for CSS, JS and image assets, and we need to specify that the '/static' URI points to the same-named directory within the munkiwebadmin app folder.

The Directory directive defines access to the Django app directory. This is also an area where other WSGI-related directives can be specified. See the [mod\_wsgi Configuration Directives Wiki page](http://code.google.com/p/modwsgi/wiki/ConfigurationDirectives) for more information.

The above example uses port 80, the standard web port. Note that if your test clients are still using port 8000, you should either change the clients' MWA\_HOST setting in munkiwebadmin-config, or set the VirtualHost to listen on port 8000 instead. The default Apache configuration on Debian (and perhaps other OS's/distributions) also requires that other listening ports be added to the ports configuration at '/etc/apache2/ports.conf'.

Switch back over to the munkiwebadmin user and we will create our Python script that is specified in the above WSGIScriptAlias directive. This should live at the root of the Django app, alongside settings.py (/usr/local/munkiwebadmin\_env/munkiwebadmin/). In this example this file is named 'munkiwebadmin.wsgi'.

```
import os, sys
import site

MUNKIWEBADMIN_ENV_DIR = '/usr/local/munkiwebadmin_env'

# Use site to load the site-packages directory of our virtualenv
site.addsitedir(os.path.join(MUNKIWEBADMIN_ENV_DIR, 'lib/python2.6/site-packages'))

# Make sure we have the virtualenv and the Django app itself added to our path
sys.path.append(MUNKIWEBADMIN_ENV_DIR)
sys.path.append(os.path.join(MUNKIWEBADMIN_ENV_DIR, 'munkiwebadmin'))

os.environ['DJANGO_SETTINGS_MODULE'] = 'settings'

import django.core.handlers.wsgi
application = django.core.handlers.wsgi.WSGIHandler()
```

We're using the 'site' module to add the virtualenv's site-packages directory, as this provides some extra logic for handling .pth files created by Python Eggs. We're also adding the virtualenv root, as well as that of the Django app, to the Python path using sys.path.append(). The 'DJANGO\_SETTINGS\_MODULE' needs to be set, and lastly we use Django's WSGI handler to pass the application back to Apache.

There are different approaches one can take in writing the WSGI script, depending on the major version of mod\_wsgi, whether you would like to define a blank 'baseline' environment, and other implementation details. The [mod\_wsgi Wiki](http://code.google.com/p/modwsgi/wiki/VirtualEnvironments) has a good starting place to read more about this.

These links are worth reading for background, or if you'd rather write a different script (or suggest an improvement to the one provided here).

Restart the apache service:
```
  service httpd restart
```

Try to access munkiadmin via http://servername/

## Configuring clients ##

In the 'munkiwebadmin/scripts' folder, you'll find the scripts which will run on the clients.

You will need to modify the MWA\_HOST value in munkiwebadmin-config, which should be set to the address or hostname of the server running MunkiWebAdmin. If you'd like to run MunkiWebAdmin on a port other than port 80 (and you probably should), you would also specify that as part of the URL (as in this example, we'd specify ':8000' at the end of the URL.

Copy the four scripts to a test client at /usr/local/munki, and ensure they are at mode 755, owned by root:wheel. The next time you run Munki on this client, you should a see a new client entry, along with a report and inventory items in the webapp.

For deployment to all your Munki-managed machines, consider either packaging the scripts, or using a copy\_from\_dmg item. As of 2013/03/23, the create-mwa-scripts-installer.sh script, which produces a pkg installer for these scripts, was added.

Congratulations! You now have a basic MunkiWebAdmin setup using mod\_wsgi.  Feel free to tweak settings as needed.

## What's next? ##

### Further modifications in settings.py ###

Here are some additional, optional modifications to settings.py you might like to make:

**Manifest usernames**

MunkiWebAdmin supports adding a user name to manifests, and this actually gets written into the manifest .plist file. Enable this feature by setting the MANIFEST\_USERNAME\_IS\_EDITABLE to True. The MANIFEST\_USERNAME\_KEY setting defines the actual key name used in the .plist.

**Git auto-commits**

If your Munki repo is also a Git repository, MunkiWebAdmin will automatically commit to it whenever a manifest is edited in the web app. You can configure the path to the Git binary with the GIT\_PATH setting.

**LDAP**

Enable LDAP support for user authentication by setting USE\_LDAP to True. On CentOS systems, the 'ldap' Python module is provided by the 'python-ldap' package. Configuration instructions for LDAP use with MunkiWebAdmin aren't yet available.