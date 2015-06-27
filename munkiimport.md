# Introduction #

munkiimport is a command-line-based assistant for importing pkgs, disk images and apps into your munki repo. It creates a pkginfo file for the installer item, wraps the pkg or app into a disk image if needed, and uploads the disk image or flat package to the repo, optionally under a subdirectory of your choosing. If the pkg/dmg upload is successful, munkiimport then uploads the pkginfo and opens it in your preferred editor.


# Details #

Basic usage:

Run munkiimport --configure to tell the utility about your repo and preferred editor:

```
munkiimport --configure
Path to munki repo (example: /Volumes/repo): 
Repo URL (example: afp://munki.pretendco.com/repo): 
pkginfo extension (Example: .plist): 
pkginfo editor (examples: /usr/bin/vi or TextMate.app):
```

Now import a pkg, or disk image:

```
munkiimport /Users/gneagle/Downloads/tl-3.1.1-client-osx.dmg
     Item name [ThinLinc Client]: 
  Display name []: 
   Description []: Remote workstation client.  
       Version [3.1.1.0.0]: 
      Catalogs [testing]: production

     Item name: ThinLinc Client
  Display name: 
   Description: Remote workstation client.
       Version: 3.1.1.0.0
    Catalogs: : production

Import this item? [y/n] y
Upload installer item to subdirectory path [None]: apps
Copying tl-3.1.1-client-osx.dmg to /Volumes/repo/pkgs/apps/tl-3.1.1-client-osx.dmg...
Saving pkginfo to /Volumes/repo/pkgsinfo/apps/ThinLinc Client-3.1.1.0.0...
```

...and then /Volumes/repo/pkgsinfo/apps/ThinLinc Client-3.1.1.0.0 opens in TextMate for additional editing. Once you're done with any editing, the command line shows:

```
Rebuild catalogs? [y/n] 
```

Most of the time you'll answer 'y' to rebuild the catalogs after you've made changes. You might answer 'n' if you are going to be importing more items and want to wait until all new items are imported before rebuilding the catalogs.