# Setting up MunkiWebAdmin on OS X #

This is a guide walking you through a basic setup of MunkiWebAdmin on an OS X machine.

This guide was most recently tested on 10 July 2013 on a Mac running OS X 10.8.4.

Basic assumption: you host a munki repo on an OS X machine and want to add MunkiWebAdmin to an existing setup.

This setup demonstrates basic functionality only. It may not be optimal for your environment. See external documentation on implementing production Django applications for more information.

Original setup documentation by Nate Walck.

## Install Prerequisites and Stage files ##

### Setup the Virtual Environment ###

Install Xcode, or at least the Command Line Tools for Xcode if they are available.  Make sure to get the right one for your version of OS X.  These can be found on developer.apple.com. Xcode can be found on the Mac App Store. (Specifically you need git, so you might be able to get by with just a git install...)

Install virtualenv.
```
  sudo easy_install virtualenv
```

Create a new virtual environment.
```
  cd /Users/Shared
  virtualenv munkiwebadmin_env
```
This will create a virtual Python environment inside a newly created munkiwebadmin\_env folder.

Note that we created this directory inside /Users/Shared; you can use another location if you'd like.

Go into the newly created directory and activate the virtual environment:
```
  cd munkiwebadmin_env
  source bin/activate
```
**If for some reason the default shell for your munkiadmin user is not bash or /bin/sh, switch to bash before you do the above. (Hint: type "bash"...)**

Your prompt should now look like this:
```
(munkiwebadmin_env)bash-3.2$
```
### Install munkiwebadmin and django prerequisites ###

We must now install some munkiwebadmin prereqs inside of the virtual environment.

Install Django. If your test machine is running Lion or later:
```
  pip install django==1.5.1
```

If your test machine is running Snow Leopard you need to use Django 1.4.1 instead:
```
  pip install django==1.4.1
```

**Note: This guide was tested with Django 1.5.1. MunkiWebAdmin may or may not run successfully with other versions of Django, though Django 1.4.1 should be OK.**

Install the django-wsgiserver:
```
  pip install django-wsgiserver==0.8.0beta
```
**Note: This guide was tested with django-wsgiserver 0.8.0beta. MunkiWebAdmin may or may not run successfully with other versions.**

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
  * Under INSTALLED\_APPS uncomment django\_wsgiserver
  * Set MUNKI\_REPO\_DIR to the local filesystem path to your munki repo. (In this case, /Users/Shared/munki\_repo)
  * Make other edits as you feel comfortable

Save and close the file.

### Initialize the Database, Create an admin account ###

Back in Terminal, initialize the database and setup the admin account by following the prompts:
```
  python manage.py syncdb
```

You will see it create a bunch of tables in a local Sqlite3 db file:

```
(munkiwebadmin_env)bash-3.2$ python manage.py syncdb
Creating tables ...
Creating table auth_permission
Creating table auth_group_permissions
Creating table auth_group
Creating table auth_user_groups
Creating table auth_user_user_permissions
Creating table auth_user
Creating table django_content_type
Creating table django_session
Creating table django_admin_log
Creating table reports_machine
Creating table reports_munkireport
Creating table inventory_inventory
Creating table inventory_inventoryitem

You just installed Django's auth system, which means you don't have any superusers defined.
Would you like to create one now? (yes/no): 
```

You'll see it also prompt you to create a default admin user. Do this, and don't forget the username or password!

```
You just installed Django's auth system, which means you don't have any superusers defined.
Would you like to create one now? (yes/no): yes
Username (leave blank to use 'gneagle'): root
Email address: root@example.com
Password: 
Password (again): 
Superuser created successfully.
Installing custom SQL ...
Installing indexes ...
Installed 0 object(s) from 0 fixture(s)
```

Remember the superuser name and password. You'll need it for initial login to the web app.

### Create a non-admin service account and group ###

It's not a good idea to run the MunkiWebAdmin webapp as root, and you probably don't want to run it as an existing user (like yourself). So we will create a dedicated service account for this application.

Create the munkiwebadmin account:

  * Open System Preferences and go to Users & Groups.
  * Create a Standard user named munkiwebadmin.

**(We're using the Users & Groups or Accounts preference pane to create the user so that an unused UID and useful defaults will be assigned automatically. If you want to create the user via command-line, feel free, but make sure all the required attributes are populated! The details are not documented here.)**

To make sure no one can log in with this account interactively, we will change a few attributes in the user record. Run the following commands in Terminal:
```
  sudo dscl . create /Users/munkiwebadmin home /var/empty
  sudo dscl . create /Users/munkiwebadmin passwd *
```
Now create a munki group and add munkiwebadmin to it.
```
  sudo dseditgroup -o create -n . munki
  sudo dseditgroup -o edit -a munkiwebadmin -t user munki
```
Make sure the munki group (which includes munkiwebadmin) can read and write in our munkiwebadmin\_env:
```
  sudo chgrp -R munki /Users/Shared/munkiwebadmin_env
  sudo chmod -R g+rw /Users/Shared/munkiwebadmin_env
```
While we are at it, lets give this group read/write access to the munki repo (MunkiWebAdmin will need permissions to modify manifests) (Substitute the path for your Munki repo)
```
  sudo chgrp -R munki /Users/Shared/munki_repo
  sudo chmod -R g+rw /Users/Shared/munki_repo
```


### Start the web application and test ###

Switch to the service account we just created:
```
  sudo su
  su munkiwebadmin
```

Make sure you are still in the correct directory and cd there if you aren't:

```
  pwd
  /Users/Shared/munkiwebadmin-env/munkiwebadmin
```

Start MunkiWebAdmin:
```
  python manage.py runwsgiserver port=8000 host=0.0.0.0
```

You'll see something like:

```
Validating models..
0 errors found
July 10, 2013 - 15:02:11
Django version 1.5.1, using settings 'munkiwebadmin.settings'
cherrypy django_wsgiserver is running at http://0.0.0.0:8000/
Quit the server with CONTROL-C.
launching with the following options:
{'adminserve': 'Deprecated',
 'autoreload': True,
 'daemonize': False,
 'host': '0.0.0.0',
 'pidfile': None,
 'port': '8000',
 'server_group': 'www-data',
 'server_name': 'localhost',
 'server_user': 'www-data',
 'servestaticdirs': True,
 'ssl_certificate': None,
 'ssl_private_key': None,
 'staticserve': True,
 'threads': 10,
 'workdir': None}
```

Open your browser of choice and go to http://127.0.0.1:8000

If all is configured properly, you should be greeted with the MunkiWebAdmin login page.

Login with the username and password you chose earlier.

You should be able to look at catalogs; look at and edit manifests; and access the site admin tools.

Reports and inventory will be empty until your Munki clients are configured to report to the MunkiWebAdmin server process.

Hitting Control-C in the terminal session will end the server process. See later in this document for how to run it persistently.

## Configuring clients ##

In the 'munkiwebadmin/scripts' folder, you'll find the scripts which will run on the clients.

You will need to modify the MWA\_HOST value in munkiwebadmin-config, which should be set to the address or hostname of the server running MunkiWebAdmin. If you'd like to run MunkiWebAdmin on a port other than port 80 (and you probably should), you would also specify that as part of the URL (as in this example, we'd specify ':8000' at the end of the URL.

Copy the four scripts to a test client at /usr/local/munki, and ensure they are at mode 755, owned by root:wheel. The next time you run Munki on this client, you should a see a new client entry, along with a report and inventory items in the webapp.

For deployment to all your Munki-managed machines, consider either packaging the scripts, or using a copy\_from\_dmg item. As of 2013/03/23, the create-mwa-scripts-installer.sh script, which produces a pkg installer for these scripts, was added.

## What next? ##

## Using launchd to start MunkiWebAdmin at startup ##

You will want a way to run MunkiWebAdmin that doesn’t require a Terminal session to be constantly open.  You could run it via the screen command, but a better option is to write a launchd item that runs the server as the munkiadmin service account.  Here's an example LaunchDaemon plist:
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>KeepAlive</key>
	<true/>
	<key>Label</key>
	<string>com.googlecode.munki.munkiwebadmin</string>
	<key>ProgramArguments</key>
	<array>
		<string>/Users/Shared/munkiwebadmin_env/bin/python</string>
		<string>/Users/Shared/munkiwebadmin_env/munkiwebadmin/manage.py</string>
		<string>runwsgiserver</string>
		<string>port=8000</string>
		<string>host=0.0.0.0</string>
	</array>
	<key>RunAtLoad</key>
	<true/>
	<key>StandardErrorPath</key>
	<string>/Users/Shared/munkiwebadmin_env/munkiwebadmin.log</string>
	<key>StandardOutPath</key>
	<string>/Users/Shared/munkiwebadmin_env/munkiwebadmin.log</string>
	<key>UserName</key>
	<string>munkiwebadmin</string>
</dict>
</plist>
```

The paths in the first two array entries under ProgramArguments may need to be changed to reflect the location of your munkiwebadmin\_env; the same goes for StandardErrorPath and StandardOutPath.

Make sure this plist has owner root, group wheel, and mode 644. Save it to `/Library/LaunchDaemons/com.googlecode.munki.munkiwebadmin.plist`.

You can manually start it (instead of waiting for a restart) like so:
```
  sudo launchctl load /Library/LaunchDaemons/com.googlecode.munki.munkiwebadmin.plist
```

### Further modifications in settings.py ###

Here are some additional, optional modifications to settings.py you might like to make:

**Manifest usernames**

MunkiWebAdmin supports adding a user name to manifests, and this actually gets written into the manifest .plist file. Enable this feature by setting the MANIFEST\_USERNAME\_IS\_EDITABLE to True. The MANIFEST\_USERNAME\_KEY setting defines the actual key name used in the .plist.

**Git auto-commits**

If your Munki repo is also a Git repository, MunkiWebAdmin will automatically commit to it whenever a manifest is edited in the web app. You can configure the path to the Git binary with the GIT\_PATH setting.

**LDAP**

Enable LDAP support for user authentication by setting USE\_LDAP to True. You'll need to install additional Python and Django modules for LDAP support. Configuration instructions for LDAP use with MunkiWebAdmin aren't yet available.

**Database**

This sample setup makes use of a Sqlite3 database. With a large number of clients, this might not perform well. Django supports several databases, but the only other I've tested (and currently use in production) is MySQL. To get Django (and MunkiWebAdmin) working with MySQL, you'll need to install mysql-python in the virtualenv:

```
 pip install mysql-python
```

and edit the DATABASES section of settings.py -- comment out the "#using sqlite3" part; uncomment the "# mysql example" part, and edit it. Point it to your database server and provide a database name, user, and password.

### Other deployment options ###

This setup guide makes use of [django-wsgiserver](https://bitbucket.org/cleemesser/django-wsgiserver), which uses the [CherryPy](http://www.cherrypy.org) webserver to serve the Django app and static files. It has reasonable performance for sites with low to medium traffic. However, it may not scale for very high numbers of clients.

There are other deployment web servers. which may provide better performance, at the cost of being more difficult to configure. The Linux setup guides use Apache2 with mod-wsgi. This module is not available by default on most versions of OS X, but is available with OS X Server (Mountain Lion), but you may be able to install it yourself.

See https://docs.djangoproject.com/en/1.5/howto/deployment/ for more discussion of other deployment options.