# Introduction #

managedsoftwareupdate is the main client tool on Munki. It runs automatically approximately every hour, or in response to events from Managed Software Update.app, but can also be manually invoked.


# Details #

Usage: `/usr/local/munki/managedsoftwareupdate [options]`

This is the main client tool. It functions in a similar manner to Apple's `softwareupdate` command-line tool.

### Options ###

| -h | --help  | Show a help message and exit |
|:---|:--------|:-----------------------------|
| -a | --auto  | Used by launchd LaunchAgent for scheduled runs. No user feedback or intervention. All other options ignored. |
| -l | --logoutinstall | Used by launchd LaunchAgent when running at the loginwindow. |
|    | --installwithnologout | Used by Managed Software Update.app when user triggers an install without logging out. |
|    | --manualcheck  | Used by launchd LaunchAgent when checking manually. |
| -m | --munkistatusoutput | Provides GUI progress feedback when installing. |
|    | --id=ID | Alternate identifier for catalog retreival |
| -q | --quiet | Quiet mode. Logs messages, but nothing to stdout. --verbose is ignored if --quiet is used. |
| -v | --verbose | More verbose output. May be specified multiple times. |
|    | --checkonly | Check for updates, but don't install them. This is the default behavior. |
|    | --installonly | Skip checking and install any pending updates. |
|    | --applesuspkgsonly  |  When checking for updates, check only for Apple SUS packages, skip Munki packages. |
|    | --munkipkgsonly | When checking for updates, check only for Munki packages, skip Apple SUS. |
| -V | --version | Print the version of the munki tools and exit. |

### Commonly used options ###

--installonly

--version

### Operation overview ###

Client asks for a main manifest.  It's retrieved.  Main manifest contains some metadata and a list of managed installs. On the client, it's named client\_manifest.plist, though on the server it may have any name.
`managedsoftwareupdate` requests a manifest via one of three values:

1) if you pass --id=arbitrarystring at the command line, it uses 'arbitrarystring' as the request:

http://webserver/repo/manifests/arbitrarystring

2) If no --id option is passed, it looks for a ClientIdentifier value in /Library/Preferences/ManagedInstalls.plist and uses that.

3) If no ClientIdentifier is available, it uses the fully-qualified hostname of the machine:

http://webserver/repo/manifests/hostname.mycompany.com

If that fails, it tries the short hostname:

http://webserver/repo/manifests/hostname

If that fails, it tries the machine serial number:

http://webserver/repo/manifests/AABBCCDDEEFF

If that fails, it tries to retrieve a manifest named "site\_default":

http://webserver/repo/manifests/site_default

Note that if the ManifestURL takes the form of a CGI invocation (http://webserver/cgi-bin/getmanifest?), the final URLs look like:

http://webserver/cgi-bin/getmanifest?arbitrarystring

The CGI approach opens the possibility for pattern matching and more logic in the client-to-catalog mapping. You can do server-side logic to dynamically return the appropriate manifest for a given client.

Next, the client asks for more detail on each managed install.

As we get more detail on each managed install, we may discover some dependancies, so we request more info on these as well.

We then check the items to see if they are already installed.

For items that are not installed, or have an older version, we download installer items and put them in a staging folder.

(any items previously staged that have no corresponding catalog item are removed from the staging folder - this covers the case where an item is erroneously added to a manifest and then removed before it is actually installed)
Using the dependency info and the catalog items, we build a list of items to be installed and their order. (InstallInfo.plist)

Unless the --auto option is passed, after checking for and downloading updates, managedsoftwareupdate will quit. You'll need to run it a second time with --installonly to get it to install the updates.  This is a safety measure to make it less likely you'll accidently install something that requires a restart underneath a logged-in user.