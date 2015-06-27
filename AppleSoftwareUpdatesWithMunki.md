# Introduction #

Munki can be configured to managed Apple Software Updates instead of the regular Software Update application, with the benefit that users don't need to be administrators to install updates.


# Details #

To manage Apple Software Updates with Munki, set InstallAppleSoftwareUpdates to True in [/Library/Preferences/ManagedInstalls.plist](configuration.md):
```
    <key>InstallAppleSoftwareUpdates</key>
    <true/>
```
Or using `defaults`:
```
defaults write /Library/Preferences/ManagedInstalls InstallAppleSoftwareUpdates -bool True
```

You may also direct Munki to use a different Apple Software Update server (for example, one you host internally) by settings the XXX key in /Library/Preferences/ManagedInstalls.plist to the appropriate CatalogURL:

```
    <key>SoftwareUpdateServerURL</key>
    <string>http://applesus.myorg.org:8088/index-leopard-snowleopard.merged-1.sucatalog</string>
```

or again with the defaults command:

```
defaults write /Library/Preferences/ManagedInstalls SoftwareUpdateServerURL "http://applesus.myorg.org:8088/index-leopard-snowleopard.merged-1.sucatalog"`
```

### Apple update behavior notes ###

Before release 0.8.4.1713.0 of the Munki tools, Apple Software Updates were only checked if there were no available Munki updates. If there were Munki packages to install or remove, they would be handled first. A given update session handled only updates from the Munki server or from an Apple Software Update server, not both.

Build 0.8.4.1713.0 changes the above behavior. Munki updates and Apple Software updates can appear and be installed in the same Munki session. To prevent possible conflicts, if any item to be installed or removed from the Munki server is an Apple item, Apple updates will not be processed in the same session.

### Apple Update Metadata ###

Build 0.8.4.1749.0 adds support for admin-provided additional metadata for Apple updates, which allows admins to better control the timing and conditions of Apple updates.  Adding metadata information for an Apple update cannot cause Munki to offer (or not offer) any update Apple Software Update will not offer. It can only change _how_ the update, if available from Apple Software Update, is installed. It can allow Munki to install an Apple update in an unattended manner; it can require a logout or restart even if the update normally would not require these, or it can be force installed after a certain date.

More info on Apple Update Metadata is [here](PkginfoForAppleSoftwareUpdates.md).


### Using the Munki tools only to install Apple Software Updates ###

With the 0.7.0 release (or later) of the Munki tools, you can use the Munki tools without any Munki repository to install Apple updates. In /Library/Preferences/ManagedPreferences.plist, set `AppleSoftwareUpdatesOnly` to True.

```
defaults write /Library/Preferences/ManagedInstalls AppleSoftwareUpdatesOnly -bool True
```

managedsoftwareupdate will then not request a manifest and catalog(s) from a Munki server, but will proceed directly to checking for (and possibly installing) Apple Software Updates.

Apple Update Metadata functionality is not available with this configuration, since the additional metadata is stored in a Munki repo.