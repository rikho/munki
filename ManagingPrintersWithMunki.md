# Introduction #

It is possible to add, remove and manage printers using Munki.

You might be asking: Why add and remove printers using Munki? Why not just use MCX?

You can use MCX to manage printer lists but the functionality is limited. One major issue with managing printer lists with MCX is if you add a printer to an MCX client's printer list, and the driver file for that the printer isn't installed on the client system, the printer will be added using the Generic Printer Driver. Even if the printer driver file is installed later the printer continues to use the Generic Printer Driver.

# Details #

**Note: I am using The Luggage to build the packages to install the printers. You can use whatever packager you prefer, but I think using The Luggage makes things easier and cleaner.**

So to install and remove a printer using Munki do the following:

**I. Install The Luggage**

1. Install XCode: http://developer.apple.com/technologies/xcode.html

2. Install The Luggage:
Download: https://github.com/unixorn/luggage/zipball/master

3. Open Terminal and type:
```
cd ~/Downloads/directory_you_extracted_the_luggage_to/
make pkg
```

4. Install the package that was created by the previous command. This installs The Luggage on your system.

**II. Prepare the Printer Install Script**

1. Create a postflight script for a package that will be installed using Munki. This is the simple shell
script I made that you can use or modify if you deem it necessary:
```
#!/bin/sh

# (c) 2010 Walter Meyer SUNY Purchase College

# Script to install and setup printers on a Mac OS X system in a "Munki-Friendly" way.
# Make sure to install the required drivers first!

# Variables. Edit these.
printername="SOME_PRINTER_NAME"
location="SOME LOCATION"
gui_display_name="HP Color LaserJet 9500N Example"
address="lpd://printserver.yourcompany.org/SOME_PRINTER_NAME"
driver_ppd="/Library/Printers/PPDs/Contents/Resources/hp color LaserJet 9500.gz"
# Populate these options if you want to set specific options for the printer. E.g. duplexing installed, etc.
option_1=""
option_2=""
option_3=""

### Printer Install ###
# In case we are making changes to a printer we need to remove an existing queue if it exists.
/usr/bin/lpstat -p $printername
if [ $? -eq 0 ]; then
	/usr/sbin/lpadmin -x $printername
fi

# Now we can install the printer.
/usr/sbin/lpadmin \
        -p "$printername" \
        -L "$location" \
        -D "$gui_display_name" \
        -v "$address" \
        -P "$driver_ppd" \
        -o "$option_1" \
        -o "$option_2" \
        -o "$option_3" \
        -o printer-is-shared=false \
        -E
# Enable and start the printers on the system (after adding the printer initially it is paused).
/usr/sbin/cupsenable $(lpstat -p | grep -w "printer" | awk '{print$2}')

# Create an uninstall script for the printer.
uninstall_script="/private/etc/cups/printers_deployment/uninstalls/$printername.sh"
mkdir -p /private/etc/cups/printers_deployment/uninstalls
echo "#!/bin/sh" > "$uninstall_script"
echo "/usr/sbin/lpadmin -x $printername" >> "$uninstall_script"
echo "/usr/bin/srm /private/etc/cups/printers_deployment/uninstalls/$printername.sh" >> "$uninstall_script"

# Permission the directories properly.
chown -R root:_lp /private/etc/cups/printers_deployment
chmod -R 700 /private/etc/cups/printers_deployment

exit 0
```

The comments in the script should explain what is going on. But basically
the script installs a printer based on the variables set at the
beginning of the script.

**III. Build the Package**

Now we are going to build a package that contains the postflight script we made using The Luggage.

1. Create a directory where you want the printer installer package to be output to and stored in on your system.

2. Create a file called 'Makefile' in the directory you created. You can use the example Makefile below. Edit the TITLE and the REVERSE\_DOMAIN at a minimum.

```
#
#   Copyright 2009 Joe Block <jpb@ApesSeekingKnowledge.net>
#
#   Licensed under the Apache License, Version 2.0 (the "License");
#   you may not use this file except in compliance with the License.
#       You may obtain a copy of the License at
#
#       http://www.apache.org/licenses/LICENSE-2.0
#
#   Unless required by applicable law or agreed to in writing, software
#   distributed under the License is distributed on an "AS IS" BASIS,
#   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#   See the License for the specific language governing permissions and
#   limitations under the License.
#
# 
#  Luggage makefile for building a package that installs a printer.

include /usr/local/share/luggage/luggage.make

TITLE=SOME_UNIQUE_PRINTER_INSTALLER_NAME
REVERSE_DOMAIN=com.yourcompany
PAYLOAD=pack-script-postflight
PACKAGE_VERSION=1.0
```

3. Drop the postflight script we made earlier in the directory with the Makefile.

4. Open terminal and cd to the directory with your Makefile and postflight script.

5. Run this command: make dmg

6. Once the command finishes you will have a dmg file with your package in it. Almost there!

**Update Note:** Initially I had the postflight script generate a unique file on each client system. That file would then be used by Munki to determine wether the printer was installed by checking the file's MD5 checksum. I've since abandoned this method in favor of using The Luggage to build the printer installer packages. If I need to change the options on an existing printer that is installed I just iterate the PACKAGE\_VERSION line in the luggage makefile from 1.0 to 1.1, build another package, and add it to my repo. This way Munki knows that an updated package needs to be installed by checking the Receipt information on the client system. Doing it this way eliminates the extra work needed to checksum the file and manually add that info to the pkginfo file.

**IV. Add the dmg to your munki repo**

1. Run makepkginfo on your printer install dmg/package located on your repo.
E.g:
```
makepkginfo /Volumes/munki/repo/path_to_your_pkg.dmg > /Volumes/munki/repo/path_to_your_pkg.dmg.pkginfo
```

2. Next change the uninstall\_method key in the pkginfo file to look like so (edit this):
```
        <key>uninstall_method</key>
        <string>/etc/cups/printers_deployment/uninstalls/your_printername_variable_from_the_postflight_script.sh</string>
```

3. Finally add a requires key to the pkginfo file and reference the required driver installation package(s) for the printer (if you haven't added the printer driver installer(s) to your repo yet do it now). E.g.:
```
        <key>requires</key>
        <array>
                <string>Lexmark Printer Drivers</string>
        </array>
```

Here is one of my printer install pkginfo files: http://pastebin.com/uwakXxVH

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>autoremove</key>
	<false/>
	<key>catalogs</key>
	<array>
		<string>printer_deployments</string>
	</array>
	<key>description</key>
	<string></string>
	<key>display_name</key>
	<string>printserver01_PRINTER01</string>
	<key>installer_item_hash</key>
	<string>5b27f8f3df91798b33b74d858adc293b6cbcd1e5d728e0f48c17c19a42bc4592</string>
	<key>installer_item_location</key>
	<string>printer_deployments/printers/printserver01_PRINTER01-1.1.dmg</string>
	<key>installer_item_size</key>
	<integer>13</integer>
	<key>minimum_os_version</key>
	<string>10.4.0</string>
	<key>name</key>
	<string>printserver01_PRINTER01</string>
	<key>receipts</key>
	<array>
		<dict>
			<key>filename</key>
			<string>printserver01_PRINTER01-1.1.pkg</string>
			<key>installed_size</key>
			<integer>0</integer>
			<key>packageid</key>
			<string>edu.purchase.printserver01_PRINTER01</string>
			<key>version</key>
			<string>1.1.0.0.0</string>
		</dict>
	</array>
	<key>uninstall_method</key>
	<string>/etc/cups/printers_deployment/uninstalls/printserver01_PRINTER01.sh</string>
	<key>uninstallable</key>
	<true/>
	<key>version</key>
	<string>1.1.0.0.0</string>
</dict>
</plist>
```

**How do I find out what options are available to configure the printer with (Duplex, etc.)?**

Install the printer in question on your system first.

Then in terminal:
```
lpoptions -p YOUR_CUPS_PRINTER_QUEUE_NAME -l
```
This command will output a list of configurable options for your printer.
So if the output is this:
```
BRMonoColor/Color/Grayscale: *Color Mono
BRSlowDrying/Slow Drying Paper: *OFF ON
```
I could set the options variables in the script to look like this:
```
option_1="BRMonoColor=Mono"
option_2="BRSlowDrying=ON"
option_3=""
```

**Potential Problems:**

There are some potential problems with using this printer installation method. If a
privileged user removes the printer manually, Munki would have no way
of knowing that the printer has been removed. Munki is only aware of the
printer being installed based on the fact that your package was installed. You could argue this isn't really a problem because if your user is an admin they can do anything they want anyway. It is something to be aware of though.

Remember that if you change the 'printername' variable in the script with the intention of changing the CUPS name of a printer that is already installed, this will not work. A new printer will be installed if you try this. If you want to change the CUPS printer name for a printer that is already installed you have to remove the existing printer with Munki first.

Also remember that Munki determines whether a printer package is installed based on the information set in your Makefile you built the package with. If you change the TITLE or REVERSE\_DOMAIN and build a package with the intention of modifying an existing install it won't work! Just iterate the PACKAGE\_VERSION from 1.0 to 1.1 or 1.1 to 1.2, etc.

# Alternate Method Using nopkg #

As of Munki 0.8.3.1634, there is now a "nopkg" type for package-free installation.  This allows us to run scripts directly in Munki without having to create packages to install.  We can use this "nopkg" type to run all of our printer installs from Munki pkginfos, which allows for easy editing in the future.

There are some pros and cons to this approach, but here's the method.

## Concepts Behind Installing Printers Using nopkg ##

The basic install script above works great for installing the printers. Normally, when installing packages with Munki, we check receipts or installs arrays to determine whether an install/update is necessary.  Payload-free packages don't leave receipts, so we'd use an installs array.  With nopkg, we can't use receipts because nothing is being installed, so we have to have the logic take place in an installcheck\_script instead.

We can use the installcheck\_script to determine:
  1. Does the printer currently exist on the system?
  1. Do we need to update the printer settings?

From there, the postinstall\_script can actually make the necessary changes.

The postinstall\_script contains the same check that the installcheck\_script does.  If the installcheck\_script succeeds, then doing the check again won't matter.  We have to re-declare variables anyway, since we can't have them persist between the two scripts reliably, so I took the easy way out and just copied and pasted the whole thing.

In order to check to make sure our printer settings are explicitly configured the way we want to, I'm actually leaving behind a special receipt file that lists a version number.  The pros and cons of this will be discussed after the method.

### Uninstalling ###
We can use the uninstall\_script to remove the printer from the pkginfo directly, rather than creating a separate script file as in the above example.  This offers a tiny bit more flexibility and doesn't leave files behind.

## Installing Printers Using nopkg ##

1) Create your installcheck\_script file locally:
```
#!/bin/sh

# Based on 2010 Walter Meyer SUNY Purchase College (c)
# Modified by Nick McSpadden, 2013

# Script to install and setup printers on a Mac OS X system in a "Munki-Friendly" way.
# Make sure to install the required drivers first!

# Variables. Edit these.
printername="queue_name".example.com
location="cosmetic_location_name"
gui_display_name="cosmetic_queue_name"
address=$printername
driver_ppd="/Library/Printers/PPDs/Contents/Resources/HP Color LaserJet 4700.gz"
# Populate these options if you want to set specific options for the printer. E.g. duplexing installed, etc.
option_1="HPOption_Duplexer=True"
option_2=""
option_3=""
currentVersion="2.1"

### Determine if receipt is installed ###
if [ -e /private/etc/cups/deployment/receipts/$printername.plist ]; then
	storedVersion=`/usr/libexec/PlistBuddy -c "Print :version" /private/etc/cups/deployment/receipts/$printername.plist`
	echo "Stored version: $storedVersion"
else
	storedVersion="0"
fi

versionComparison=`echo "$storedVersion < $currentVersion" | bc -l`
# This will be 0 if the current receipt is greater than or equal to current version of the script

### Printer Install ###
# If the queue already exists (returns 0), we don't need to reinstall it.
/usr/bin/lpstat -p $printername
if [ $? -eq 0 ]; then
	if [ $versionComparison == 0 ]; then
		# We are at the current or greater version
		exit 1
	fi
    # We are of lesser version, and therefore we should delete the printer and reinstall.
    exit 0
fi
```

1.5) Create the postinstall\_script as well:
```
#!/bin/sh

# Based on 2010 Walter Meyer SUNY Purchase College (c)
# Modified by Nick McSpadden, 2013

# Script to install and setup printers on a Mac OS X system in a "Munki-Friendly" way.
# Make sure to install the required drivers first!

# Variables. Edit these.
printername="queue_name".example.com
location="cosmetic_location_name"
gui_display_name="cosmetic_queue_name"
address=$printername
driver_ppd="/Library/Printers/PPDs/Contents/Resources/HP Color LaserJet 4700.gz"
# Populate these options if you want to set specific options for the printer. E.g. duplexing installed, etc.
option_1="HPOption_Duplexer=True"
option_2=""
option_3=""
currentVersion="2.1"

### Determine if receipt is installed ###
if [ -e /private/etc/cups/deployment/receipts/$printername.plist ]; then
	storedVersion=`/usr/libexec/PlistBuddy -c "Print :version" /private/etc/cups/deployment/receipts/$printername.plist`
	echo "Stored version: $storedVersion"
else
	storedVersion="0"
fi

versionComparison=`echo "$storedVersion &lt; $currentVersion" | bc -l`
# This will be 0 if the current receipt is greater than or equal to current version of the script

### Printer Install ###
# If the queue already exists (returns 0), we don't need to reinstall it.
/usr/bin/lpstat -p $printername
if [ $? -eq 0 ]; then
	if [ $versionComparison == 0 ]; then
		# We are at the current or greater version
		exit 1
	fi
    # We are of lesser version, and therefore we should delete the printer and reinstall.
    /usr/sbin/lpadmin -x $printername
fi

# Now we can install the printer.
/usr/sbin/lpadmin \
        -p "$printername" \
        -L "$location" \
        -D "$gui_display_name" \
        -v lpd://"${address}" \
        -P "$driver_ppd" \
        -o "$option_1" \
        -o "$option_2" \
        -o "$option_3" \
        -o printer-is-shared=false \
        -o printer-error-policy=abort-job \
        -E
# Enable and start the printers on the system (after adding the printer initially it is paused).
/usr/sbin/cupsenable $(lpstat -p | grep -w "printer" | awk '{print$2}')

# Create a receipt for the printer
mkdir -p /private/etc/cups/deployment/receipts
/usr/libexec/PlistBuddy -c "Add :version string" /private/etc/cups/deployment/receipts/$printername.plist
/usr/libexec/PlistBuddy -c "Set :version $currentVersion" /private/etc/cups/deployment/receipts/$printername.plist

# Permission the directories properly.
chown -R root:_lp /private/etc/cups/deployment
chmod -R 700 /private/etc/cups/deployment

exit 0
```

2) We do the same for the uninstall script:
```
#!/bin/sh
printerName="queue_name".example.com
/usr/sbin/lpadmin -x $printerName
rm -f /private/etc/cups/deployment/receipts/$printerName.plist
```

3) Now that the scripts exist, we can create a pkginfo out of it that is properly escaped and formatted for XML:
```
makepkginfo --installcheck_script=installcheck_script.sh --uninstall_script=uninstall_script.sh > printer.pkginfo
```

4) Fill in the rest of the important things into the pkginfo:
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>autoremove</key>
	<false/>
	<key>catalogs</key>
	<array>
		<string>release</string>
	</array>
	<key>description</key>
	<string></string>
	<key>display_name</key>
	<string>AddPrinter</string>
	<key>installcheck_script</key>
	<string>#!/bin/sh

# Based on 2010 Walter Meyer SUNY Purchase College (c)
# Modified by Nick McSpadden, 2013

# Script to install and setup printers on a Mac OS X system in a "Munki-Friendly" way.
# Make sure to install the required drivers first!

# Variables. Edit these.
printername="queue_name".example.com
location="cosmetic_location_name"
gui_display_name="cosmetic_queue_name"
address=$printername
driver_ppd="/Library/Printers/PPDs/Contents/Resources/HP Color LaserJet 4700.gz"
# Populate these options if you want to set specific options for the printer. E.g. duplexing installed, etc.
option_1="HPOption_Duplexer=True"
option_2=""
option_3=""
currentVersion="2.1"

### Determine if receipt is installed ###
if [ -e /private/etc/cups/deployment/receipts/$printername.plist ]; then
	storedVersion=`/usr/libexec/PlistBuddy -c "Print :version" /private/etc/cups/deployment/receipts/$printername.plist`
	echo "Stored version: $storedVersion"
else
	storedVersion="0"
fi

versionComparison=`echo "$storedVersion &lt; $currentVersion" | bc -l`
# This will be 0 if the current receipt is greater than or equal to current version of the script

### Printer Install ###
# If the queue already exists (returns 0), we don't need to reinstall it.
/usr/bin/lpstat -p $printername
if [ $? -eq 0 ]; then
	if [ $versionComparison == 0 ]; then
		# We are at the current or greater version
		exit 1
	fi
    # We are of lesser version, and therefore we should delete the printer and reinstall.
    exit 0
fi

</string>
	<key>installer_type</key>
	<string>nopkg</string>
	<key>minimum_os_version</key>
	<string>10.7.0</string>
	<key>name</key>
	<string>AddPrinter</string>
	<key>postinstall_script</key>
	<string>#!/bin/sh

# Based on 2010 Walter Meyer SUNY Purchase College (c)
# Modified by Nick McSpadden, 2013

# Script to install and setup printers on a Mac OS X system in a "Munki-Friendly" way.
# Make sure to install the required drivers first!

# Variables. Edit these.
printername="queue_name".example.com
location="cosmetic_location_name"
gui_display_name="cosmetic_queue_name"
address=$printername
driver_ppd="/Library/Printers/PPDs/Contents/Resources/HP Color LaserJet 4700.gz"
# Populate these options if you want to set specific options for the printer. E.g. duplexing installed, etc.
option_1="HPOption_Duplexer=True"
option_2=""
option_3=""
currentVersion="2.1"

### Determine if receipt is installed ###
if [ -e /private/etc/cups/deployment/receipts/$printername.plist ]; then
	storedVersion=`/usr/libexec/PlistBuddy -c "Print :version" /private/etc/cups/deployment/receipts/$printername.plist`
	echo "Stored version: $storedVersion"
else
	storedVersion="0"
fi

versionComparison=`echo "$storedVersion &lt; $currentVersion" | bc -l`
# This will be 0 if the current receipt is greater than or equal to current version of the script

### Printer Install ###
# If the queue already exists (returns 0), we don't need to reinstall it.
/usr/bin/lpstat -p $printername
if [ $? -eq 0 ]; then
	if [ $versionComparison == 0 ]; then
		# We are at the current or greater version
		exit 1
	fi
    # We are of lesser version, and therefore we should delete the printer and reinstall.
    /usr/sbin/lpadmin -x $printername
fi

# Now we can install the printer.
/usr/sbin/lpadmin \
        -p "$printername" \
        -L "$location" \
        -D "$gui_display_name" \
        -v lpd://"${address}" \
        -P "$driver_ppd" \
        -o "$option_1" \
        -o "$option_2" \
        -o "$option_3" \
        -o printer-is-shared=false \
        -o printer-error-policy=abort-job \
        -E
# Enable and start the printers on the system (after adding the printer initially it is paused).
/usr/sbin/cupsenable $(lpstat -p | grep -w "printer" | awk '{print$2}')

# Create a receipt for the printer
mkdir -p /private/etc/cups/deployment/receipts
/usr/libexec/PlistBuddy -c "Add :version string" /private/etc/cups/deployment/receipts/$printername.plist
/usr/libexec/PlistBuddy -c "Set :version $currentVersion" /private/etc/cups/deployment/receipts/$printername.plist

# Permission the directories properly.
chown -R root:_lp /private/etc/cups/deployment
chmod -R 700 /private/etc/cups/deployment

exit 0</string>
	<key>unattended_install</key>
	<true/>
	<key>uninstall_method</key>
	<string>uninstall_script</string>
	<key>uninstall_script</key>
	<string>#!/bin/sh
printerName="queue_name".sacredsf.org
/usr/sbin/lpadmin -x $printerName
rm -f /private/etc/cups/deployment/receipts/$printerName.plist</string>
	<key>uninstallable</key>
	<true/>
	<key>version</key>
	<string>2.1</string>
</dict>
</plist>

```

This pkginfo makes it an unattended\_install, which means it'll trigger even if a user is logged in, without requiring the GUI to activate.

The important parts to change:
  * **description** - This is the cosmetic description.
  * **display\_name** - The name that users see.
  * **name** - This is what goes into the manifest.
  * **version** - This should match the version you list in your script.  When updating this pkginfo, make sure you update the version in all places accordingly, for your own sanity.  Since we're duplicating parts of the scripts, it's critically important that all pieces are updated properly.

5) Throw your pkginfo into your Munki repo and makecatalogs.  Add it to a manifest, and test it out!


## Pros & Cons to the nopkg Approach ##

The advantage to using the nopkg method is that all of your logic is being done in the pkginfo directly.  To make any changes to the process simply involves editing the pkginfo.  You don't have to rebuild packages when you make these changes.

On the other hand, we are leaving a receipt behind, and using script logic to determine versioning.  This is very close to how packages natively behave, so what we're really doing is recreating the package except without the "build" step.  Since there's no payload, though, the use of a package doesn't really confer any natural advantage in this particular context.

If you wanted to bundle print drivers along with your printer, then you'd definitely want a package because then you'd have a payload to deploy.  I avoid this problem by installing the HP Printer Driver package from Apple as part of the standard deployment process, so all machines should have it already.

This solution offers a bit more flexibility than rebuilding packages, but it depends on how often you think you may need to make changes to the process, and how many printers.