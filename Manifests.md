# Introduction #

Format of munki manifest files and supported keys


# Details #

Manifests are essentially just a simple list of the items to install or verify their installation or to remove or verify their removal.

### catalogs ###

The key 'catalogs' defines which catalogs to search for the given items. The catalogs are searched in order, but any valid match stops the search, so in the below example, a matching item in the testing catalog would stop the search - the production catalog would not be searched.

```
<dict>
	<key>catalogs</key>
        <array>
	    <string>testing</string>
	    <string>production</string>
	</array>
	<key>managed_installs</key>
	<array>
		<string>ServerAdminTools</string>
		<string>TextWrangler</string>
		<string>Silverlight</string>
	</array>
</dict>
```


### included\_manifests ###

Manifests support references to other manifests (nested manifests):

```
<dict>
	<key>catalogs</key>
        <array>
	    <string>testing</string>
	    <string>production</string>
	</array>
	<key>included_manifests</key>
	<array>
		<string>leopard_standard_apps</string>
	</array>
	<key>managed_installs</key>
	<array>
		<string>TextWrangler</string>
	</array>
</dict>
```

Where there is a manifest leopard\_standard\_apps that contains some standard apps for your users:

```
<dict>
	<key>managed_installs</key>
	<array>
		<string>MicrosoftOffice2008</string>
		<string>Firefox</string>
		<string>Thunderbird</string>
	</array>
</dict>
```

Included manifests do not need to have a "catalogs" entry (and in most cases, should _not_ include a "catalogs" list) -- if this is missing, the catalogs of the parent manifest will be searched.

### managed\_installs ###

managed\_installs contains a list of items you would like to ensure are installed and kept up-to-date:

```
	<key>managed_installs</key>
	<array>
		<string>MicrosoftOffice2008</string>
		<string>Firefox</string>
		<string>Thunderbird</string>
	</array>
```

### managed\_uninstalls ###

You may include a "managed\_uninstalls" key:

```
	<key>managed_uninstalls</key>
	<array>
                <string>ServerAdminTools</string>
                <string>TextWrangler</string>
                <string>Silverlight</string>
        </array>
```

These items will be checked to see if they're installed, and removed if possible.

### optional\_installs ###

Manifests may also have an "optional\_installs" key:

```
<key>optional_installs</key>
<array>
    <string>GoogleChrome</string>
    <string>GoogleEarth</string>
    <string>GoogleSketchUp</string>
    <string>TextWrangler</string>
</array>
```

These items are checked against the available catalogs and added to a list of optional installs displayed in Managed Software Update.app. Users can then select any of these items for install, or if they are installed, for removal.

managed\_installs and managed\_uninstalls have higher precedence than optional\_installs, so if an item is in either a managed\_installs or managed\_uninstalls list, it will not be displayed in Managed Software Update.app as an available optional install.

If a user chooses items to install or remove from the list of optional software, the choices are recorded in a locally-generated manifest: "/Library/Managed Installs/manifests/SelfServeManifest". This manifest inherits its catalogs from the primary manifest (the one pointed to by the ClientIdentifier) and is processed like any other manifest after all of the server-provided manifests have been processed.

### managed\_updates ###

The value for this key is in the same format as managed\_installs. Items in managed\_updates are each checked to see if **some** version of the item is already installed; if so, the item is processed just as if it was in the managed\_installs list. You could use managed\_updates to tell munki to update software that is not in a managed\_installs list; for example, if you have machines that may or may not have Adobe Photoshop CS5 and you do not want to add Adobe Photoshop CS5 to a managed\_installs list, but would still like munki to apply any applicable updates, you could add Adobe Photoshop CS5 to the managed\_updates list.

An item in a managed\_updates list that is scheduled for removal (because it appears in a managed\_uninstalls list) will not be processed for updates.


### item names ###

For managed\_installs, managed\_uninstalls and optional\_installs, items are typically referred to by name only, but if you need to specify the install of a specific version, you can append the version number:

```
	<key>managed_installs</key>
	<array>
		<string>MicrosoftOffice2008</string>
		<string>Firefox-3.0.9</string>
		<string>Thunderbird</string>
```

In the above example, the catalog(s) will be searched for the latest version of MicrosoftOffice2008, version 3.0.9 of Firefox, and the latest version of Thunderbird.

Note that munki generally will not downgrade an existing install from a newer version to an older version, so specifying an older version in the managed\_installs list will not downgrade existing installations.

managed\_updates should list the item only without a version number.

For managed\_installs, managed\_uninstalls, optional\_installs, and managed\_updates, the name used in the manifest will be matched against the "name" field in each item in each catalog listed in the catalogs attribute of the manifest.