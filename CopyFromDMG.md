# Introduction #

Starting with munki tools 0.6.0.633.0, munki supports a new "copy\_from\_dmg" installer type. This is a more generic mechanism for copying items from a disk image to the startup disk than the appdmg installer type.

copy\_from\_dmg supports copying arbitrary items from a disk image to arbitrary locations on the startup disk. You can also specify owner, group, and mode for the copied items.


# Details #

To use the copy\_from\_dmg method, use makepkginfo to create a pkginfo item like so:

```
makepkginfo /Volumes/repo/pkgs/apps/GoogleEarthMacNoUpdate-Intel-5.2.dmg --owner=root --group=admin --mode=go-w --item="Google Earth Web Plug-in.plugin" --destinationpath="/Library/Internet Plug-ins"
```

This creates a pkginfo item instructing munki to copy "Google Earth Web Plug-in.plugin" from the root of the mounted disk image to "/Library/Internet Plug-ins"; to set its owner to "root," group to "admin," and to remove the write bit for "other". The resulting plist looks like this:

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
	<key>installer_item_location</key>
	<string>apps/GoogleEarthMacNoUpdate-Intel-5.2.dmg</string>
	<key>installer_item_size</key>
	<integer>23168</integer>
	<key>installer_type</key>
	<string>copy_from_dmg</string>
	<key>installs</key>
	<array>
		<dict>
			<key>CFBundleShortVersionString</key>
			<string>5.2</string>
			<key>path</key>
			<string>/Library/Internet Plug-ins/Google Earth Web Plug-in.plugin</string>
			<key>type</key>
			<string>bundle</string>
		</dict>
	</array>
	<key>items_to_copy</key>
	<array>
		<dict>
			<key>destination_path</key>
			<string>/Library/Internet Plug-ins</string>
			<key>group</key>
			<string>admin</string>
			<key>mode</key>
			<string>go-w</string>
			<key>source_item</key>
			<string>Google Earth Web Plug-in.plugin</string>
			<key>user</key>
			<string>root</string>
		</dict>
	</array>
	<key>minimum_os_version</key>
	<string>10.4.0</string>
	<key>name</key>
	<string>Google Earth Web Plug-in</string>
	<key>uninstall_method</key>
	<string>remove_copied_items</string>
	<key>uninstallable</key>
	<true/>
	<key>version</key>
	<string>5.2.0.0.0</string>
</dict>
</plist>
```

The Google Earth disk image actually contains two items you might want to install -- the Google Earth application itself, and the Google Earth Web Plug-in. You have two options here:

  1. Create two separate pkginfo items, one for Google Earth.app and one for Google Earth Web Plug-in.plugin. You could then add an "update\_for" key to the Google Earth Web Plug-in item, marking it as an update for Google Earth. When munki is instructed to install Google Earth, it will install the Google Earth Web Plug-in as well.<br><br>
<ol><li>Combine the "installs" and "items_to_copy" sections from the two pkginfo items into a single pkginfo item.  makepkginfo currently only supports a single installation item when creating a copy_from_dmg pkginfo item, but the plist itself can support multiple items. Such a combined pkginfo item would look like this:</li></ol>

<pre><code>&lt;?xml version="1.0" encoding="UTF-8"?&gt;<br>
&lt;!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd"&gt;<br>
&lt;plist version="1.0"&gt;<br>
&lt;dict&gt;<br>
	&lt;key&gt;autoremove&lt;/key&gt;<br>
	&lt;false/&gt;<br>
	&lt;key&gt;catalogs&lt;/key&gt;<br>
	&lt;array&gt;<br>
		&lt;string&gt;testing&lt;/string&gt;<br>
	&lt;/array&gt;<br>
	&lt;key&gt;installer_item_location&lt;/key&gt;<br>
	&lt;string&gt;apps/GoogleEarthMacNoUpdate-Intel-5.2.dmg&lt;/string&gt;<br>
	&lt;key&gt;installer_item_size&lt;/key&gt;<br>
	&lt;integer&gt;23168&lt;/integer&gt;<br>
	&lt;key&gt;installer_type&lt;/key&gt;<br>
	&lt;string&gt;copy_from_dmg&lt;/string&gt;<br>
	&lt;key&gt;installs&lt;/key&gt;<br>
	&lt;array&gt;<br>
		&lt;dict&gt;<br>
			&lt;key&gt;CFBundleIdentifier&lt;/key&gt;<br>
			&lt;string&gt;com.Google.GoogleEarthPlus&lt;/string&gt;<br>
			&lt;key&gt;CFBundleName&lt;/key&gt;<br>
			&lt;string&gt;Google Earth&lt;/string&gt;<br>
			&lt;key&gt;CFBundleShortVersionString&lt;/key&gt;<br>
			&lt;string&gt;5.2&lt;/string&gt;<br>
			&lt;key&gt;path&lt;/key&gt;<br>
			&lt;string&gt;/Applications/Google Earth.app&lt;/string&gt;<br>
			&lt;key&gt;type&lt;/key&gt;<br>
			&lt;string&gt;application&lt;/string&gt;<br>
		&lt;/dict&gt;<br>
		&lt;dict&gt;<br>
			&lt;key&gt;CFBundleShortVersionString&lt;/key&gt;<br>
			&lt;string&gt;5.2&lt;/string&gt;<br>
			&lt;key&gt;path&lt;/key&gt;<br>
			&lt;string&gt;/Library/Internet Plug-ins/Google Earth Web Plug-in.plugin&lt;/string&gt;<br>
			&lt;key&gt;type&lt;/key&gt;<br>
			&lt;string&gt;bundle&lt;/string&gt;<br>
		&lt;/dict&gt;<br>
	&lt;/array&gt;<br>
	&lt;key&gt;items_to_copy&lt;/key&gt;<br>
	&lt;array&gt;<br>
		&lt;dict&gt;<br>
			&lt;key&gt;destination_path&lt;/key&gt;<br>
			&lt;string&gt;/Applications&lt;/string&gt;<br>
			&lt;key&gt;source_item&lt;/key&gt;<br>
			&lt;string&gt;Google Earth.app&lt;/string&gt;<br>
		&lt;/dict&gt;<br>
                &lt;dict&gt;<br>
			&lt;key&gt;destination_path&lt;/key&gt;<br>
			&lt;string&gt;/Library/Internet Plug-ins&lt;/string&gt;<br>
			&lt;key&gt;group&lt;/key&gt;<br>
			&lt;string&gt;admin&lt;/string&gt;<br>
			&lt;key&gt;mode&lt;/key&gt;<br>
			&lt;string&gt;go-w&lt;/string&gt;<br>
			&lt;key&gt;source_item&lt;/key&gt;<br>
			&lt;string&gt;Google Earth Web Plug-in.plugin&lt;/string&gt;<br>
			&lt;key&gt;user&lt;/key&gt;<br>
			&lt;string&gt;root&lt;/string&gt;<br>
		&lt;/dict&gt;<br>
	&lt;/array&gt;<br>
	&lt;key&gt;minimum_os_version&lt;/key&gt;<br>
	&lt;string&gt;10.4.0&lt;/string&gt;<br>
	&lt;key&gt;name&lt;/key&gt;<br>
	&lt;string&gt;Google Earth&lt;/string&gt;<br>
	&lt;key&gt;uninstall_method&lt;/key&gt;<br>
	&lt;string&gt;remove_copied_items&lt;/string&gt;<br>
	&lt;key&gt;uninstallable&lt;/key&gt;<br>
	&lt;true/&gt;<br>
	&lt;key&gt;version&lt;/key&gt;<br>
	&lt;string&gt;5.2.0.0.0&lt;/string&gt;<br>
&lt;/dict&gt;<br>
&lt;/plist&gt;<br>
</code></pre>