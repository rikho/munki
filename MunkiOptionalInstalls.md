# Introduction #

Optional installs allow administrators to specify packages as available for optional installation, allowing end-users to choose to install and/or remove these items without needing admin privileges themselves.


# Details #

You can now add a "optional\_installs" key to any manifest (or included manifest):

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

managed\_installs and managed\_uninstalls have higher precedence than optional\_installs, so if an item is in either a managed\_installs or managed\_uninstallslist, it will not be displayed in Managed Software Update.app as an available optional install.

If a user chooses items to install or remove from the list of optional software, the choices are recorded in a locally-generated manifest: "/Library/Managed Installs/manifests/SelfServeManifest". This manifest inherits its catalogs from the primary manifest (the one pointed to by the ClientIdentifier) and is processed like any other manifest after all of the server-provided manifests have been processed.