# Introduction #

MunkiWebAdmin is a Django web application that incorporates the functionality of the MunkiReport project and also provides the ability to browse catalogs, and browse and edit manifests of an existing Munki repo.

## Components and Credits ##

MunkiWebAdmin was developed using the [Django web framework](https://www.djangoproject.com).

It also makes use of the following projects:

  * [jQuery](http://jquery.com)
  * [DataTables](http://datatables.net)
  * [Twitter Bootstrap](http://twitter.github.com/bootstrap/)

The demonstration setup for OS X also makes use of [django-wsgiserver](https://bitbucket.org/cleemesser/django-wsgiserver), which itself makes use of the web server component of [CherryPy](http://www.cherrypy.org).

Report functionality ported from Per Olofsson's [MunkiReport](http://code.google.com/p/munkireport/).

Inventory code contributed back to Arjen van Bochoven's [MunkiReport-PHP](http://code.google.com/p/munkireport-php/).

## Getting the code ##

If you have experience with setting up Django applications, the Django project is available here:
```
  git clone https://code.google.com/p/munki.munkiwebadmin/
```
You can also browse the code repo here:

> http://code.google.com/p/munki/source/browse/?repo=munkiwebadmin

For a more step by step approach to setting up MunkiWebAdmin, see one of the setup guides below.

## Setup Guides ##

  * [Sample setup on OS X](MunkiWebAdminOSXSetup.md)
  * [Sample setup on CentOS/RHEL](MunkiWebAdminLinuxSetup.md)
  * [Sample setup on Ubuntu Server](MunkiWebAdminUbuntuSetup.md)

## User-contributed guides and documentation ##

  * [Brian Mickelson's notes on deploying MunkiWebAdmin os OS X Server (10.8)](http://fluffyquickness.com/2013/01/mwa-10-8-server/)

## Support and Discussion ##

Please direct discussion of MunkiWebAdmin to the munki-web-admin Google Group: http://groups.google.com/group/munki-web-admin

## Screenshots ##

**Machine report:**

![http://wiki.munki.googlecode.com/git/images/mwa_report.jpg](http://wiki.munki.googlecode.com/git/images/mwa_report.jpg)

**Manifest editor:**

![http://wiki.munki.googlecode.com/git/images/mwa_manifest.jpg](http://wiki.munki.googlecode.com/git/images/mwa_manifest.jpg)

**Machine application inventory:**

![http://wiki.munki.googlecode.com/git/images/mwa_inventory.jpg](http://wiki.munki.googlecode.com/git/images/mwa_inventory.jpg)