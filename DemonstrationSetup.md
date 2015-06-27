# Introduction #

Since Munki can use virtually any web server as its server, and since Mac OS X ships with Apache 2,  it’s very easy to set up a demonstration Munki server on any available Mac. You can even set up a Munki server on a single machine that is also a Munki client, and that is what we'll do here.

> This wiki page is adapted and updated from the MacTech column here: http://www.mactech.com/articles/mactech/Vol.26/26.11/2611MacEnterprise-ManagingSoftwareInstallswithMunki-Part2/index.html

# Details #

## Building a server repository ##

To set up a Munki server, we’re going to create a directory structure in `/Users/Shared`, and then configure Apache2 to serve it via HTTP. You can do the next few steps via the Finder or via the Terminal, but it’s easier to write them out as Terminal commands:

```
cd /Users/Shared/
mkdir munki_repo
mkdir munki_repo/catalogs
mkdir munki_repo/manifests
mkdir munki_repo/pkgs
mkdir munki_repo/pkgsinfo
```

You might be wondering what that last directory is. The `pkgsinfo` directory holds data that is not used directly by Munki clients, but is used by other Munki tools to create the catalogs. One more thing: let’s make sure the Apache 2 can read and traverse all of these directories:

```
chmod -R a+rX munki_repo
```

Next, we need to tell Apache2 to serve the `munki_repo` directory via HTTP. You could edit the `/etc/apache2/http.conf` file, or one of the other .conf files used by Apache2, but there’s a much easier method for this demonstration.

```
sudo ln -s /Users/Shared/munki_repo /Library/WebServer/Documents/
```

This creates a symlink inside `/Library/WebServer/Documents/` that points to our new `munki_repo` directory. By default on Mac OS X, `/Library/WebServer/Documents/` is Apache 2 ‘s DocumentRoot, so it will serve anything in that directory via HTTP.  (For Mountain Lion Server, make the symlink in whatever directory is Apache 2's WebRoot, which may depend on how you've configured Apache 2.)

## Activate Apache ##

If you are running Lion or earlier, turn on Web Sharing in the Sharing preferences pane.

If you are running Mountain Lion (and not Mountain Lion Server), you can turn on Apache 2 like so:

```
   sudo apachectl start
```

> This activates the Apache web server, and also activates the launchd job so that Apache will be active across restarts.
> To revert this change:

```
   sudo apachectl stop
```

Now that Apache is running, you can test your work so far. Using your favorite web browser, navigate to `http://localhost/munki_repo`. If you’ve done things correctly to this point, you should see a directory listing, showing the catalogs, manifests, pkgsinfo, and pkgs directories. If you don't see a directory listing, and you are using a web server other than OS X client's Web Sharing, see the following note.

> NOTE: For this demonstration, we're taking advantage of the fact that on OS X Client, Apache 2 serves directory listings by default to make it easier to see what is going on. More commonly, Apache is configured to NOT list directory contents. In production use, you will almost certainly want to disable directory listings, as they'd allow people to easily discover the contents of your pkgs directory and perhaps then "help themselves" to software they are not entitled to. Munki does not require directory listings to operate.

## Populating the repo ##

We now have a working Munki repo – but it’s completely empty and not useful at all. So let’s start to populate the repo.

We’re going to use some tools distributed with munki to import packages into our new Munki repo. Download the current munki installation package at http://code.google.com/p/munki/downloads/list.

Install the Munki tools by mounting the disk image and double-clicking the Installer package and installing like any other package. A restart is required after installation.

The tools you’ll use as an administrator are available from the command-line, and are installed in `/usr/local/munki`. This location is not in the standard search path, so you’ll need to either add this directory to your search paths, or be sure to type the full path when invoking these tools.

The tool we will use to import packages into the munki repo is called `munkiimport`. We need to configure it before we can use it – telling it where to find our repo, among other things.

```
bash-3.2$ /usr/local/munki/munkiimport --configure
Path to munki repo (example: /Volumes/repo): /Users/Shared/munki_repo   
Repo fileshare URL (example: afp://munki.example.com/repo): <just hit return>
pkginfo extension (Example: .plist): <just hit return>
pkginfo editor (examples: /usr/bin/vi or TextMate.app): TextMate.app <substitute your favorite text editor>
Default catalog to use (example: testing): testing
```

We are first asked for the path to the Munki repo, and since we set one up at `/Users/Shared/munki_repo`, that’s what we enter.

Next, we are asked for a repo fileshare URL. This is used when the repo is hosted on a remote file server, and this would typically be an afp:// or smb:// URL specifying the share. Since we’re hosting the repo on the local machine, we’ll leave this blank.

We are then asked to specify an extension to append to the name of pkginfo files. Some admins prefer “.plist”, some prefer “.pkginfo”. Personally, I just leave it blank – Munki doesn’t care.

Next, you are asked for an editor to use for the pkginfo files. If you like command-line editors, you can specify `/usr/bin/vi` or `/usr/bin/emacs` for example. If you, like me, prefer GUI text editors, you can specify a GUI text editor application by name (but be sure to include the “.app” extension). I picked TextMate.app, but you could choose TextWrangler.app, BBEdit.app, or even TextEdit.app.

Finally, you are asked for the default catalog new packages/pkginfo should be added to. We'll use a "testing" catalog for this.

Next, let’s get a package to import. Firefox is a good example package, and you can download it from http://www.mozilla.com/. As of this writing, the current version is 10.0, and when I download it using Safari, a disk image named “Firefox 10.0.dmg” is downloaded to my Downloads folder.

We’ll return to the command line to import the Firefox package.

### Importing Firefox ###

```
bash-3.2$ /usr/local/munki/munkiimport ~/Downloads/Firefox\ 10.0.dmg 
      Item name [Firefox]: 
   Display name []: Mozilla Firefox
    Description []: Web browser from Mozilla
        Version [10.0]: 
       Catalogs [testing]: 

      Item name: Firefox
   Display name: Mozilla Firefox
    Description: Web browser from Mozilla
        Version: 10.0
       Catalogs: testing

Import this item? [y/n] y
Upload item to subdirectory path []: apps/mozilla
Path /Users/Shared/munki_repo/pkgs/apps/mozilla doesn't exist. Create it? [y/n] y
Copying Firefox 10.0.dmg to /Users/Shared/munki_repo/pkgs/apps/mozilla/Firefox 10.0.dmg...
Saving pkginfo to /Users/Shared/munki_repo/pkgsinfo/apps/mozilla/Firefox-10.0...
```

We run the `munkiimport` tool and provide it a path to our downloaded disk image.

`munkiimport` then asks us to confirm or override some basic information about the package. We accept the item name by simply hitting return, but provide a new “Display name” and “Description”. We accept the version and the catalogs, again, by simply hitting return.

`munkiimport` then prints back our choices and asks if we want to import the item. (If we made any mistakes, this would be a good time to say “no”!) We agree, and `munkiimport` asks us if we’d like to upload the package to a subdirectory path. We could just skip this, and upload everything to the top level of the pkgs directory in the munki repo, but as our number of packages grows, that might get hard to navigate. So we’re going to upload this into a directory named “Mozilla” inside a directory named “apps”. As a sanity check, `munkiimport` warns us that the subdirectory path we’ve chosen doesn’t yet exist. Since this is a brand new repo, we knew in advance that the directory didn’t exist, so we want `munkiimport` to create it for us.

Finally, `munkiimport` copies the Firefox package to `/Users/Shared/munki_repo/pkgs/apps/mozilla/` and saves the pkginfo to `/Users/Shared/munki_repo/pkgsinfo/apps/mozilla/Firefox-10.0`.

Since I chose TextMate.app as my editor when I configured `munkiimport` earlier, `munkiimport` next opens the newly created pkginfo file in TextMate. No matter which editor you choose, you'll see a standard Apple property list file (plist) describing the package you just imported.

This gives you another opportunity to edit the pkginfo using your favorite text editor. We don’t need to make any changes, though, so we can just close it. If we return our attention to the terminal window we used to run `munkiimport`, we’ll see it’s prompting us for one more bit of information:

```
Rebuild catalogs? [y/n] 
```

Remember that munki clients don’t use the individual pkginfo files; instead they download and consult munki catalogs to find available software. So to actually make use of the pkginfo we just generated, we need to build new versions of all the defined catalogs. Answering “y” to this prompt causes munkiimport to rebuild the munki catalogs.

```
Rebuild catalogs? [y/n] y
Adding apps/mozilla/Firefox-10.0 to testing...
```

Since we only have one package (and its corresponding pkginfo) in our munki repo, we see a single item has been added to the testing catalog.
Again we can check our work so far. In your web browser, navigate to` http://localhost/munki_repo/catalogs/testing`. You should see a property list which contains the pkginfo for Firefox.

### Creating a client manifest ###

We now have one package in our Munki repo. Our next step is to create a client manifest so that Munki knows what to install on a given machine.

We'll use the `manifestutil` tool to create our manifest.

```
% /usr/local/munki/manifestutil 
Entering interactive mode... (type "help" for commands)
> new-manifest test_munki_client
> add-catalog testing --manifest test_munki_client
Added testing to catalogs of manifest test_munki_client.
> add-pkg Firefox --manifest test_munki_client
Added Firefox to section managed_installs of manifest test_munki_client.
> exit
```

We've created a new manifest named "test\_munki\_client".
We added "testing" to the list of catalogs to consult, and "Firefox" to the list of packages to install.

If you examine the file at `/Users/Shared/munki_repo/manifests/test_munki_client`, it should look like this:

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>catalogs</key>
	<array>
		<string>testing</string>
	</array>
	<key>included_manifests</key>
	<array>
	</array>
	<key>managed_installs</key>
	<array>
		<string>Firefox</string>
	</array>
	<key>managed_uninstalls</key>
	<array>
	</array>
</dict>
</plist>
```

Again, you can check your work in your web browser by navigating to `http://localhost/munki_repo/manifests/test_munki_client`. You should see the file you just created displayed in your web browser.

## Munki Client Configuration ##

We’re done (for now) with the server. Next, we need to configure the Munki client so it knows about our server. The Munki client stores its configuration in `/Library/Preferences/ManagedInstalls.plist`. Unless you’ve run the Munki client before, this file won’t yet exist. We’ll use the `defaults` command to create it with the data we need.

```
sudo defaults write /Library/Preferences/ManagedInstalls SoftwareRepoURL "http://localhost/munki_repo"
sudo defaults write /Library/Preferences/ManagedInstalls ClientIdentifier "test_munki_client"
```

We’ve told the client tools the top-level URL for the munki repo -- `http://localhost/munki_repo`, and the name of the client manifest we’d like to use -- "test\_munki\_client". That’s it for the client configuration. If you'd like, check your work:

```
defaults read /Library/Preferences/ManagedInstalls
```

Make sure the values of "SoftwareRepoURL" and "ClientIdentifier" are as expected.

## Testing the Munki client software ##

Now the moment of truth: let’s run the Munki client from the command line.

```
sudo /usr/local/munki/managedsoftwareupdate 
Managed Software Update Tool
Copyright 2010-2012 The Munki Project
http://code.google.com/p/munki

Downloading Firefox 10.0.dmg...
	0..20..40..60..80..100
Verifying package integrity...
The following items will be installed or upgraded:
    + Firefox-10.0
        Web browser from Mozilla

Run managedsoftwareupdate --installonly to install the downloaded updates.
```

Success! Munki saw that we needed Firefox 10.0 and downloaded it. (It did not yet install it – we’ll get to that in a bit.)

But what if instead when you run `managedsoftwareupdate` you see this:

```
sudo /usr/local/munki/managedsoftwareupdate 
Managed Software Update Tool
Copyright 2010-2012 The Munki Project
http://code.google.com/p/munki

No changes to managed software are available.
```

The most likely reason you see this is because you already have Firefox 10.0 (or later) installed. If you really want to test Munki, delete your copy of Firefox:

```
sudo rm –r /Applications/Firefox.app
```

Then run `/usr/local/munki/managedsoftwareupdate` again – you should see it being downloaded as in the example above.

### Demonstrating Managed Software Update.app ###

We ran `managedsoftwareupdate` from the command line and verified that the munki client tools could talk to our munki server and download the Firefox package. But, as we’ve noted, `managedsoftwareupdate` did not actually install Firefox. We could call `managedsoftwareupdate` again, this time passing it the `-–installonly` flag to make it install what it just downloaded. But instead, we’re going to introduce another tool – the one “regular” users would interact with – Managed Software Update.app. You’ll find it in the `/Applications/Utilities` folder. Double-click it to launch it.

Managed Software Update will check for updates with the Munki server, and should shortly display a window (closely resembling Apple's Software Update application's main window) displaying Firefox 10.0.

If you click on **Update now**, you’ll be asked if you want to install without logging out, or to log out and install. Choose one and Firefox will be installed.

## Conclusion ##

You've set up a demonstration of Munki's server and client components.

For a more in-depth introduction to Munki, see these MacTech articles:

http://www.mactech.com/articles/mactech/Vol.26/26.10/2610MacEnterprise-ManagingSoftwareInstallswithMunki/index.html<br>
<a href='http://www.mactech.com/articles/mactech/Vol.26/26.11/2611MacEnterprise-ManagingSoftwareInstallswithMunki-Part2/index.html'>http://www.mactech.com/articles/mactech/Vol.26/26.11/2611MacEnterprise-ManagingSoftwareInstallswithMunki-Part2/index.html</a><br>
<a href='http://www.mactech.com/articles/mactech/Vol.26/26.12/2612MacEnterprise-ManagingSoftwareInstallswithMunki-Part3/index.html'>http://www.mactech.com/articles/mactech/Vol.26/26.12/2612MacEnterprise-ManagingSoftwareInstallswithMunki-Part3/index.html</a><br>
<a href='http://www.mactech.com/articles/mactech/Vol.27/27.01/2701MacEnterprise-ManagingSoftwareInstallswithMunki-Part4/index.html'>http://www.mactech.com/articles/mactech/Vol.27/27.01/2701MacEnterprise-ManagingSoftwareInstallswithMunki-Part4/index.html</a>