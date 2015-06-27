# Introduction #

Munki can install software distributed in Apple package format, from "drag-n-drop" disk images, and software "packaged" with Adobe CS3/4/5/6 Enterprise Deployment tools.

But some software may have to be repackaged in order to be installed by munki. Some examples of types of software that may need to be packaged or repackaged:

  * Software distributed in InstallVISE or InstallAnywhere format.
  * Software distributed in Apple package format, but the package doesn't work silently or at the loginwindow.
  * Software you develop yourself that isn't packaged at all


# Details #

### Tools ###

**PackageMaker**

http://support.apple.com/kb/DL1071

This is Apple's utility for creating packages. It is available as part of the Xcode Tools, and also included with the Server Admin Tools. It can create every package format supported by Apple: old-style bundle packages, new-style flat packages, metapackages, distribution packages, and hybrid packages that work on multiple OS versions.

Pros: PackageMaker is a supported Apple tool and is free. Many of the other tools rely on PackageMaker for at least some of their functionality.

Cons: It's hard to use, and had a history of buggy releases.

**Iceberg**

http://s.sudre.free.fr/Software/Iceberg.html

Iceberg is freeware by Stéphan Sudre, licensed with a BSD-style license. Capable and well-documented, Iceberg is very popular among Mac OS X administrators. It can create packages and metapackages, but not the newer distribution packages and flat packages.

Pros: Easy to use and free. It supports creation of packages from filesystem snapshots, as well as manual assembly of package contents.

Cons: Iceberg's installer installs a StartupItem that launches an always-on background task. This makes some admins uncomfortable.

**Packages**

http://s.sudre.free.fr/Software/Packages.html

Newer tool from the creator of Iceberg -- this one creates ONLY flat packages.

**InstallEase**

http://www.lanrev.com/solutions/package-building.html

Absolute Manage (formerly LANrev), the maker of a cross-platform system management tool, has made their InstallEase package creation utility freely available.

Pros: Ease of use, the ability to export Iceberg project files, and the creation of "uninstall" packages - packages that will uninstall software installed by another package. Creation of packages from filesystem snapshots.

Cons: It does not work standalone. To actually create packages, you must have Apple's PackageMaker and/or Iceberg installed as well. You must register to download the software, which puts you on Absolute Manage's marketing lists.

**Casper Composer**

http://www.jamfsoftware.com/products/composer.php

Composer is a $100 utility from JAMF Software. Part of the Casper Suite of OS X client management tools, Composer is also available separately. Casper Composer creates packages based on filesystem snapshots. When used with the Casper suite, it can create installation packages with extra abilities such as installing default preferences into users' home directories.

Pros: Easy to use. Good documentation.

Cons: It's not free. Casper Composer requires Apple's PackageMaker to build standard Apple packages. Composer's special package features work only with other tools in the Casper Suite.

### Techniques ###

MacTech articles:

Sample workflow using InstallEase:

http://www.mactech.com/articles/mactech/Vol.25/25.01/2501MacEnterprise-PackagingforDistribution/index.html

Sample workflow using PackageMaker:

http://www.mactech.com/articles/mactech/Vol.25/25.03/2503MacEnterprise-PackagingforSystemAdministrators/index.html