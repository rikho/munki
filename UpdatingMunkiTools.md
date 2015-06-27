# Details #

The munki tools are distributed as a metapackage. This enables several things for the Munki admin:

  1. Admins can choose to install only a subset of the munki tools; for example, they can choose to _not_ install the munki admin tools (makecatalogs, makepkginfo, munkiimport) on most client machines.
  1. If you have customized or replaced the launchd jobs used by munki, you can upgrade to newer tools without having your changes overwritten.
  1. In many cases, you can update the munki tools silently in the background without needing a restart. As long as the launchd subpackage has not changed, no reboot is required.

All of these require installing only a subset of the packages included in the metapackage. There are a few approaches to accomplishing that:

  1. Using installer's ChoiceChangesXML files and munki's support for embedding them into pkginfo. You could, for example, use the following installer\_choices\_xml to install only the core tools and Managed Software Update.app; skipping the install of the admin tools and the launchd plists:
```
<key>installer_choices_xml</key>
<array>
    <dict>
        <key>attributeSetting</key>
        <integer>1</integer>
        <key>choiceAttribute</key>
        <string>selected</string>
        <key>choiceIdentifier</key>
        <string>core</string>
    </dict>
    <dict>
        <key>attributeSetting</key>
        <integer>0</integer>
        <key>choiceAttribute</key>
        <string>selected</string>
        <key>choiceIdentifier</key>
        <string>admin</string>
    </dict>
    <dict>
        <key>attributeSetting</key>
        <integer>1</integer>
        <key>choiceAttribute</key>
        <string>selected</string>
        <key>choiceIdentifier</key>
        <string>app</string>
    </dict>
    <dict>
        <key>attributeSetting</key>
        <integer>0</integer>
        <key>choiceAttribute</key>
        <string>selected</string>
        <key>choiceIdentifier</key>
        <string>launchd</string>
    </dict>
</array>
```
  1. Editing the distribution.dist file inside the munkitools.mpkg -- this would have a similar effect as the previous approach, but could be used outside of munki -- for example, if installing the tools via ARD or manually.
  1. Disassembling or ignoring the metapackage and installing only those sub packages you wish. With munki, you could create separate diskimages for each subpackage, or you can use a feature of makepkginfo to specify a specific package from a disk image. Here's an example of specifying just the core tools subpackage:

```
/usr/local/munki/makepkginfo -p munkitools-0.7.1.1173.0.mpkg/Contents/Packages/munkitools_core-0.7.1.1173.0.pkg /Users/Shared/pkgs/munkitools-0.7.1.1173.0.mpkg.dmg
```

Which results in:

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>autoremove</key>
	<false/>
	<key>catalogs</key>
	<array>
		<string>testing</string>
	</array>
	<key>description</key>
	<string>Core command-line tools used by munki.</string>
	<key>display_name</key>
	<string>Munki core tools</string>
	<key>installed_size</key>
	<integer>448</integer>
	<key>installer_item_hash</key>
	<string>06e3199b75061019525b9ef77ea8047efeb719dbbf0050479604d66e93b78d8c</string>
	<key>installer_item_location</key>
	<string>munkitools-0.7.1.1173.0.mpkg.dmg</string>
	<key>installer_item_size</key>
	<integer>555</integer>
	<key>minimum_os_version</key>
	<string>10.4.0</string>
	<key>name</key>
	<string>munkitools_core</string>
	<key>package_path</key>
	<string>munkitools-0.7.1.1173.0.mpkg/Contents/Packages/munkitools_core-0.7.1.1173.0.pkg</string>
	<key>receipts</key>
	<array>
		<dict>
			<key>filename</key>
			<string>munkitools_core-0.7.1.1173.0.pkg</string>
			<key>installed_size</key>
			<integer>448</integer>
			<key>packageid</key>
			<string>com.googlecode.munki.core</string>
			<key>version</key>
			<string>0.7.1.1173.0</string>
		</dict>
	</array>
	<key>uninstall_method</key>
	<string>removepackages</string>
	<key>uninstallable</key>
	<true/>
	<key>version</key>
	<string>0.7.1.1173.0</string>
</dict>
</plist>
```

You can create pkginfo items for munki-core, munki-admin, munki-app, munki-launchd and use munki's dependency relationships to ensure everything needed gets installed and that reboots are enforced only when necessary.

| munki-app | unattended\_install: True | requires: munki-core |
|:----------|:--------------------------|:---------------------|
| munki-admin | unattended\_install: True | requires: munki-core |
| munki-core | unattended\_install: True | requires: munki-launchd |
| munki-launchd | RestartAction: RequireRestart |                      |

If a managed\_installs in a manifest specified only "munki-app", munki-app, munki-core, and munki-launchd would be installed. For future updates, as long as munki-launchd hasn't changed, the updates would happen as unattended installs.  If a future update included an updated munki-launchd, the unattended\_install would be invalidated, and a reboot would be needed, meaning the update would either wait until all users were logged out, or for a user to initiate the update via Managed Software Update.

### Easiest approach ###

In my opinion, the easiest approach is to use "Show Contents" to dig into the munki-tools metapackage, and then import each sub package separately:

This might look like:

```
/usr/local/munki/munkiimport /Volumes/munkitools-0.8.2.1430.0/munkitools-0.8.2.1430.0.mpkg/Contents/Packages/munkitools_core-0.8.2.1430.0.pkg
/usr/local/munki/munkiimport /Volumes/munkitools-0.8.2.1430.0/munkitools-0.8.2.1430.0.mpkg/Contents/Packages/munkitools_admin-0.8.2.1430.0.pkg
/usr/local/munki/munkiimport /Volumes/munkitools-0.8.2.1430.0/munkitools-0.8.2.1430.0.mpkg/Contents/Packages/munkitools_launchd-0.8.0.1.pkg
/usr/local/munki/munkiimport /Volumes/munkitools-0.8.2.1430.0/munkitools-0.8.2.1430.0.mpkg/Contents/Packages/munkitools_app-3.3.1.1398.pkg
```

You'll still need either use requires/update\_for relationships to ensure all the sub packages are installed, or perhaps just all all of these packages as individual managed\_installs items to an included\_manifest.

This technique should allow for quicker and more seamless updates to munki tools for your client machines, as in many cases you can update the munki tools with no user action required.