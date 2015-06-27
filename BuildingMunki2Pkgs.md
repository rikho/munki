# Introduction #

Building a Munki 2 Installer Package

Automated builds of the Munki 2 installer packages are available at http://munkibuilds.org. These are not official builds, but are made available by a friendly third party.

# Details #

You can also build a Munki installer package yourself.

### Requirements ###

  * Xcode 5, which then requires either OS X Mountain Lion or OS X Mavericks.
  * git (installing Xcode 5 on Mavericks should make this available; on Mountain Lion you may need to also install the Xcode Command Line Tools)

### Procedure ###

In a directory you can write to, run:

`git clone https://code.google.com/p/munki/`

This will create a new "munki' directory containing the latest git repo. Change into the directory:

`cd munki`

Check out the Munki2 branch:

`git checkout -b Munki2 origin/Munki2`

Run the build script:

`./code/tools/make_munki_mpkg.sh`

Provide your password when requested (certain steps require `sudo`).

If the script runs successfully, it will tell you where to find the Installer package:

`Distribution package created at /Users/Shared/munki/munkitools-2.0.0.2088.pkg.`