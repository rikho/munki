# Introduction #

When checking to see what needs to be installed, Munki uses information in the pkginfo items to decide whether or not a given item is installed. A Munki admin must understand how this works in order to create functional pkginfo items.

# Details #

Each time `managedsoftwareupdate` runs (except when in --installonly mode), it checks the lists of managed\_installs, managed\_updates, optional\_installs, and managed\_uninstalls items to see if they are installed.

Listed below, in order of precedence, are the methods used by munki to determine if a given item should be installed (or removed).
The order is as follows:
  * installcheck\_script
  * installs items
  * receipts

When combining these methods, **only** the highest priority method is used.  For example, if a given pkginfo item has both an "installs" list and a "receipts" list, the receipts will be ignored for purposes of determining installation status. Even in this case, though, receipts may be used when removing an item, as they help Munki determine exactly which files were installed.

### Install Check Script ###
As of munkitools 0.8.3 Build 1581, support has been added for including an "installcheck\_script" within a pkginfo file.  The purpose is to provide a method for determining if an item needs to be installed where providing "installs/receipts" is inadequate.  Command-line tools typically installed via `port` (macports) or Python modules installed using `easy_install` are prime examples as they provide no easy method for determining their installed version.

An "installcheck\_script" should be crafted such that an exit code of 0 indicates that the item is currently **not** installed and should therefore be installed.  All non-zero exit codes indicate that the item in question is installed.

Here's an example "installcheck\_script" illustrating a check to determine if the current version of the `argparse` Python module is installed:

```
#!/bin/sh

# Grab current version of installed python module
version="$(python -c 'import argparse;print argparse.__version__' 2>/dev/null)"

# Compare with the version we want to install
if [ ${version:-0} \< 1.2.1 ]; then
	exit 0
else
	exit 1
fi
```

_Same script, embedded in a pkginfo file..._

```
<key>installcheck_script</key>
	<string>#!/bin/sh

# Grab current version of installed python module
version="$(python -c 'import argparse;print argparse.__version__' 2&gt;/dev/null)"

# Compare with the version we want to install
if [ ${version:-0} \&lt; 1.2.1 ]; then
	exit 0
else
	exit 1
fi
</string>
```

Inversely, if an item that included an "installcheck\_script" were to be a "managed\_uninstall", this same script would be used to determine if the item is installed so that a removal can be processed.  This is similar to how removals are processed, based on "installs" items.

#### Uninstall Check Script ####
Optionally, an explicit "uninstallcheck\_script" can be provided to determine whether or not an item should be removed.  In this case, the script would provide an exit code of 0 to indicate that the item is currently installed and that removal should occur.  All non-zero exit codes indicate that the item in question is **not** installed.

### Installs ###
This list is generated for you by `makepkginfo` or `munkiimport` for some types of installation items ("drag-n-drop" disk images; Adobe installers), but not for Apple packages. You can generate (or modify) this list yourself.

This is the most flexible mechanism for determining installation status. The "installs" list can contain any number of items. These can be applications, Preference Panes, Frameworks, or other bundle-style items, Info.plists, or simple directories or files. The Munki admin can use any combination of items to help Munki determine if an item is installed or not.

Here's an auto-generated "installs" list for Firefox 6.0:

```
	<key>installs</key>
	<array>
		<dict>
			<key>CFBundleIdentifier</key>
			<string>org.mozilla.firefox</string>
			<key>CFBundleName</key>
			<string>Firefox</string>
			<key>CFBundleShortVersionString</key>
			<string>6.0</string>
			<key>minosversion</key>
			<string>10.5</string>
			<key>path</key>
			<string>/Applications/Firefox.app</string>
			<key>type</key>
			<string>application</string>
		</dict>
	</array>
```

To determine if Firefox 6 is installed, Munki looks for an application with a CFBundleIdentifier of "org.mozilla.firefox" and if found, verifies that its version ("CFBundleShortVersionString") is at least 6.0.

If it can't find the application or its version is lower than 6.0, Munki considers Firefox-6.0 as _not_ installed.

Installs lists can contain multiple items. If any item is missing or has an older version, the item will be considered not installed.

You can manually generate items to add to an "installs" list using `makepkginfo`:

```
/usr/local/munki/makepkginfo -f /Library/Internet\ Plug-Ins/Flash\ Player.plugin
```
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>installs</key>
	<array>
		<dict>
			<key>CFBundleShortVersionString</key>
			<string>10.3.183.5</string>
			<key>path</key>
			<string>/Library/Internet Plug-Ins/Flash Player.plugin</string>
			<key>type</key>
			<string>bundle</string>
		</dict>
	</array>
</dict>
</plist>
```

You could then copy and paste the entire "installs" key and value, or copy just the dict value and add it to and existing installs list.

In this example, Munki would check for the existence of "/Library/Internet Plug-Ins/Flash Player.plugin" and if found, check its version. If the version was lower than "10.3.183.5", this item would be considered not installed.

You can generate "installs" items for any filesystem item, but Munki only knows how to determine versions for bundle-style items that contain an Info.plist or version.plist with version information. For other filesystem items, Munki can only determine existence (in the case of a non-bundle directory), or can calculate a checksum (for files). For files with checksums, the test will fail (and therefore the item will be considered not installed) if the checksum for the file on disk does not match the checksum in the pkginfo.

```
	<key>installs</key>
	<array>
		<dict>
			<key>md5checksum</key>
			<string>087fe4805b63412ec3ed559b0cd9be71</string>
			<key>path</key>
			<string>/private/var/db/dslocal/nodes/MCX/computergroups/loginwindow.plist</string>
			<key>type</key>
			<string>file</string>
		</dict>
	</array>
```

If you'd like Munki to only check for the existence of a file and do not care about its contents, remove the generated md5checksum information in the installs item info. Be sure to leave the path intact!

```
	<key>installs</key>
	<array>
		<dict>
			<key>path</key>
			<string>/private/var/db/dslocal/nodes/MCX/computergroups/loginwindow.plist</string>
			<key>type</key>
			<string>file</string>
		</dict>
	</array>
```


### Receipts ###
When an Apple-style package is installed, it leaves a receipt on the machine. Metapackages leave multiple receipts. `makepkginfo` and `munkiimport` add the names and versions of those receipts to a "receipts" array in the pkginfo for a package.

Here's is a receipts array for the Avid LE QuickTime codecs, version 2.3.4:
```
	<key>receipts</key>
	<array>
		<dict>
			<key>filename</key>
			<string>AvidCodecsLE.pkg</string>
			<key>installed_size</key>
			<integer>1188</integer>
			<key>name</key>
			<string>AvidCodecsLE</string>
			<key>packageid</key>
			<string>com.avid.avidcodecsle</string>
			<key>version</key>
			<string>2.3.4</string>
		</dict>
	</array>
```

If Munki is using the "receipts" array to determine installation status, it checks for the existence and the version of each receipt in the array. If any receipt is missing or has a lower version number than the version specified for that receipt in the "receipts" array, the item is considered not installed. Only if every receipt is present and all versions are the same as the ones in the pkginfo (or higher) is the item considered installed.

If you are troubleshooting, you can use the `pkgutil` tool on Snow Leopard and Lion to examine the installed receipts:

```
# pkgutil --pkg-info com.avid.avidcodecsle
No receipt for 'com.avid.avidcodecsle' found at '/'.
```

In this case, the receipt for the Avid LE QuickTime codecs was not found on this machine.

A common complication with receipts is this: with many metapackages, the installation logic results in only a subset of the subpackages being installed. Generally, the "receipts" list contains a receipt for every subpackage in a metapackage (and needs this info if Munki is asked to remove the software based on package receipts). But if it is normal and expected that not every subpackage will actually be installed, Munki will continually mark the item as not currently installed and offer to install it again and again.

One solution for this issue is to add an "optional" key with the value of "true" to the receipts that are optionally installed. Munki will then not consider these receipts when determining installation status.

```
	<key>receipts</key>
	<array>
		<dict>
			<key>filename</key>
			<string>mandatory.pkg</string>
			<key>installed_size</key>
			<integer>1188</integer>
			<key>name</key>
			<string>Mandatory</string>
                        <key>packageid</key>
			<string>com.foo.mandatory</string>
			<key>version</key>
			<string>1.0</string>
		</dict>
		<dict>
			<key>filename</key>
			<string>optional.pkg</string>
			<key>installed_size</key>
			<integer>1188</integer>
			<key>name</key>
			<string>Optional</string>
                        <key>optional</key>
                        <true/>
                        <key>packageid</key>
			<string>com.foo.optional</string>
			<key>version</key>
			<string>1.0</string>
		</dict>
	</array>
```

Another solution for this situation is to provide an "installs" array that lists items that are installed by the package. Munki can use that information instead of the receipts to determine installation status.