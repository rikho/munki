# Introduction #

Due to an issue with the Adobe Reader X installer packages, installation will fail at the loginwindow. Here's how to fix it without repackaging.

# Details #

See this discussion:
http://groups.google.com/group/munki-dev/browse_thread/thread/9bb8bd205620b3be/74e1bd25f4b6d04c?lnk=gst&q=Adobe+Reader+X#74e1bd25f4b6d04c

To avoid the issue, add a preinstall\_script to the package info:

```
	<key>preinstall_script</key>
	<string>#!/bin/sh
if [ -e "/Applications/Adobe Reader.app" ]; then
	rm -r "/Applications/Adobe Reader.app"
fi
exit 0
	</string>
```

This has been tested up through the 10.1.2 update.

# Current Recommendation #

Use [AutoPkg](http://autopkg.github.io/autopkg/).

The AdobeReader.munki recipe finds the latest Reader X/XI, downloads it, imports it into your Munki repo, and adds the appropriate preinstall\_script to the package info.