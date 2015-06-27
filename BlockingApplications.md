# Introduction #

Munki supports the concept of 'blocking\_applications'. This causes Munki to skip unattended\_installs or uninstalls if certain applications are running, and causes Managed Software Update.app to warn the user to quit conflicting applications if they are running when the user chooses to update without logging out.

# Details #

Munki will skip any unattended\_installs or unattended\_uninstalls if:

  * There is a blocking\_applications key in the pkginfo, and any application listed is running, or
  * There is no blocking\_applications key in the pkginfo, and any application in the **installs** list (if it exists) is running.

Additionally, using similar logic as above, Managed Software Update.app will display an alert asking the user to quit any blocking apps that are running if the user chooses to update without logging out.

The blocking\_applications list may be added to the pkginfo for any package, and takes this form:

```
<key>blocking_applications</key>
<array>
	<string>Firefox</string>
	<string>Safari</string>
	<string>Opera</string>
</array>
```

or

```
<key>blocking_applications</key>
<array>
	<string>Firefox.app</string>
	<string>Safari.app</string>
	<string>Opera.app</string>
</array>
```

This sample blocking\_applications list would be suitable for use with the Adobe Flash Player installation package.

The string used to identify the application should be the name of the application bundle -- the ".app" extension is optional. Apps do not have to be installed or managed by munki to be included in a blocking\_applications list.

Even though it might be useful to block on faceless background applications (like iTunesHelper) or other processes, since normal users do not have the ability to easily quit such processes, don't use them as blocking\_applications; instead specify the item needs a logout or restart as applicable. It would be confusing and frustrating for a user to be notified that installation cannot continue because "Microsoft Database Daemon" is running.