# Introduction #

munki stores configuration info in the "ManagedInstalls" preferences domain. By default, this info is stored in /Library/Preferences/ManagedInstalls.plist, but you can also use MCX (as of the 0.7.0 release) or /private/var/root/Library/Preferences/ManagedInstalls.plist, or a combination of these locations with the normal defaults precedence:

  * MCX
  * /private/var/root/Library/Preferences/ManagedInstalls.plist
  * /Library/Preferences/ManagedInstalls.plist

# Details #

Here's a sample /Library/Preferences/ManagedInstalls.plist:

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>ClientIdentifier</key>
  <string>arbitrary_name</string>
  <key>SoftwareRepoURL</key>
  <string>http://munkiwebserver/repo</string>
  <key>LoggingLevel</key>
  <integer>1</integer>
  <key>DaysBetweenNotifications</key>
  <integer>1</integer>
</dict>
</plist>
```

# Supported ManagedInstalls Keys #

| **Key** | **Type** | **Default** | **Description** |
|:--------|:---------|:------------|:----------------|
| AppleSoftwareUpdatesOnly | boolean  | false       | If true, only install updates from an Apple Software Update server. No munki repository is needed or used.|
| InstallAppleSoftwareUpdates | boolean  | false       | If true, install updates from an Apple Software Update server, in addition to "regular" munki updates.|
| SoftwareUpdateServerURL | string   |             | Catalog URL for Apple Software Updates. If undefined or empty, Munki will use the same catalog that the OS uses when you run Apple's Software Update application or call /usr/sbin/softwareupdate. |
| SoftwareRepoURL | string   | http://munki/repo | Base URL for munki repository|
| PackageURL | string   | `<SoftwareRepoURL>`/pkgs | Base URL for munki pkgs. Useful if your packages are served from a different server than your catalogs or manifests.|
| CatalogURL | string   | `<SoftwareRepoURL>`/catalogs | Base URL for munki catalogs. Useful if your catalogs are served from a different server than your packages or manifests.|
| ManifestURL | string   | `<SoftwareRepoURL>`/manifests | Base URL for munki manifests. Useful if your manifests are served from a different server than your catalogs or manifests.|
| IconURL | string   | `<SoftwareRepoURL>`/icons | (**New for Munki 2**) Base URL for product icons. Useful if your icons are served from a different server or different directory than the default.|
| ClientResourceURL | string   | `<SoftwareRepoURL>`/client\_resources | (**New for Munki 2**) Base URL for custom client resources for Managed Software Update. Useful if your resources are served from a different server or different directory than the default.|
| ClientResourcesFilename | string   | manifest name.zip or site\_default.zip | (**New for Munki 2**) Specific filename to use when requesting custom client resources.|
| HelpURL | string   | none        | (**New for Munki 2**) If defined, a URL to open/display when the user selects "Managed Software Center Help" from Managed Software Center's Help menu.|
| ClientIdentifier | string   |             | Identifier for munki client. Usually is the same as a manifest name on the munki server. If this is empty or undefined, Munki will attempt the following identifiers, in order: fully-qualified hostname, "short" hostname, serial number and finally, "site\_default" |
| ManagedInstallDir | string   | /Library/Managed Installs | Folder where munki keeps its data on the client.|
| LogFile | string   | /Library/Managed Installs/Logs/ManagedSoftwareUpdate.log | Primary log is written to this file. Other logs are written into the same directory as this file.|
| LogToSyslog | boolean  | false       | If true, log to `/var/log/system.log` in addition to ManagedSoftwareUpdate.log.|
| LoggingLevel | integer  | 1           | Higher values cause more detail to be written to the primary log.|
| DaysBetweenNotifications | integer  | 1           | Number of days between user notifications from Managed Software Update. Set to 0 to have Managed Software Update notify every time a background check runs if there are available updates.|
| UseClientCertificate | boolean  | false       | If true, use an SSL client certificate when communicating with the munki server. Requires an https:// URL for the munki repo. See ClientCertificatePath for details.|
| UseClientCertificateCNAsClientIdentifier | boolean  | false       | If true, use the CN of the client certificate as the Client Identifier.Used in combination with the UseClientCertificate key.|
| SoftwareRepoCAPath | string   | (empty)     | Path to the directory that stores your CA certificate(s). See the curl man page for more details on this parameter.|
| SoftwareRepoCACertificate | string   | /Library/Managed Installs/certs/ca.pem | Absolute path to your CA Certificate.|
| ClientCertificatePath | string   | /Library/Managed Installs/certs/[munki.pem|client.pem|cert.pem] | Absolute path to a client certificate. There are 3 defaults for this key. Concatenated cert/key PEM file accepted.|
| ClientKeyPath | string   | (empty)     | Absolute path to a client private key.|
| AdditionalHttpHeaders | array    | (empty)     | This key provides the ability to specify custom HTTP headers to be sent with all curl() HTTP requests. AdditionalHttpHeaders must be an array of strings with valid HTTP header format.|
| PackageVerificationMode | string   | hash        | Controls how munki verifies the integrity of downloaded packages. Possible values are: **none**: No integrity check is performed. **hash**: Integrity check is performed if package info contains checksum information. **hash\_strict**: Integrity check is performed, and fails if package info does not contain checksum information.|
| SuppressUserNotification | boolean  | false       | If true, Managed Software Update will never notify the user of available updates. Managed Software Update can still be manually invoked to discover and install updates.|
| SuppressAutoInstall | boolean  | false       | If true, munki will not automatically install or remove items.|
| SuppressLoginwindowInstall | boolean  | false       | Added in version 0.8.4.1696.0. If true, Munki will not install items while idle at the loginwindow except for those marked for unattended\_install or unattended\_uninstall. |
| SuppressStopButtonOnInstall | boolean  | false       | If true, Managed Software Update will hide the stop button while installing or removing software, preventing users from interrupting the install.|
| InstallRequiresLogout | boolean  | false       | If true, Managed Software Update will require a logout for all installs or removals.|
| ShowRemovalDetail | boolean  | false       | If true, Managed Software Update will display detail for scheduled removals.|


# Additional Notes #

## LogFile ##

munki normally writes its logs to /Library/Managed Installs/Logs/, with the main log written to ManagedSoftwareUpdate.log in that directory. Other logs are named "Install.log", "errors.log", and "warnings.log".  If you'd like the logs to be written somewhere else (for example /var/log or /Library/Logs), set LogFile to the desired pathname of the main log:

`sudo defaults write /Library/Preferences/ManagedInstalls LogFile "/var/log/munki/managedsoftwareupdate.log"`

The other logs will be written to the same directory.

## InstallAppleSoftwareUpdates ##

If this key is present and set to True, munki will call softwareupdate and attempt to install Apple Software Updates.

## SoftwareUpdateServerURL ##

This key can be used to point to an internal Apple Software Update server.

## SuppressUserNotification ##

This key (when present and value is set to True) causes munki to never notify users of available updates. This might be useful in a lab environment, where you'd like updates to be applied only when no-one is logged in and the machine is at the login window.

## SuppressAutoInstall ##

Normally, munki automatically installs and removes software if there are changes needed and that machine is at the loginwindow with no users logged in. If you have a need to do updates always and only with the consent of the user, setting SuppressAutoInstall to True prevents munki from automatically installing updates (and processing removals).

## SuppressLoginwindowInstall ##

(Added in version 0.8.4.1696.0) If this preference is set to true, Munki will not install updates when idle at the loginwindow, with the exception of updates marked for unattended\_install or unattended\_uninstall.

## ShowRemovalDetail ##

By default, Managed Software Update.app suppresses detail on what will be removed, instead showing a simple "Software removals" entry in the list. If you'd like Managed Software Update.app to show specific detail about what will be removed, set ShowRemovalDetail to True. This key has no effect on /usr/local/munki/managedsoftwareupdate, which always shows all detail.

## InstallRequiresLogout ##

Managed Software Update.app enforces a logout before it installs or removes software only if one or more items to be installed/removed requires a logout or restart. You can force a logout for all updates by setting InstallRequiresLogout to True. This key has no effect on running /usr/local/munki/managedsoftwareupdate from the command-line.

## AdditionalHttpHeaders ##

This key provides the ability to specify custom HTTP headers to be sent with all curl() HTTP requests. AdditionalHttpHeaders must be an array of strings with valid HTTP header format. For example:
```
    <key>AdditionalHttpHeaders</key>
    <array>
            <string>Key-With-Optional-Dashes: Foo Value</string>
            <string>another-custom-header: bar value</string>
    </array>
```

One could use this to obtain a cookie in a preflight script and update ManagedInstalls.plist with the appropriate header.  However, it is recommended that you use Secure Config for sensitive data (i.e. cookie) since ManagedInstalls.plist is world-readable.

# Secure Config #

This feature is completely optional, and is enabled purely by the presence of the config.

Munki will check for the existence of a configuration plist in the following secure location:
`/private/var/root/Library/Preferences/ManagedInstalls.plist`

This feature is necessary since `/Library/Preferences` is not meant for secure data, and `defaults write` sets plists to world-readable.  The secure config is protected by permissions of the `/private/var/root` directory, so file permissions are not a concern.

**Note**: configs are loaded from the secure ManagedInstalls.plist **after** the regular ManagedInstalls.plist, so any configs set here will overwrite configs defined in ManagedInstalls.plist!

**Note**: the following configs are **required** in /Library/Preferences/ManagedInstalls.plist (or set via MCX), as the GUI portion of Munki runs as the logged in user, not root. **Do not** place them in the secure plist, or you may encounter unexpected behavior from Managed Software Update.app:
  * ManagedInstallDir
  * InstallAppleSoftwareUpdates
  * AppleUpdatesOnly
  * ShowRemovalDetail
  * InstallRequiresLogout
  * HelpURL

# Using the `defaults` command #

If you use the `/usr/bin/defaults` command to set values for keys in ManagedInstalls.plist, remember that values default to the "string" type. If you are writing a boolean, integer, or array value, be sure to add the appropriate type flag. For example:

`defaults write /Library/Preferences/ManagedInstalls SuppressAutoInstall -bool false`

See `man defaults` for a complete list of type flags.