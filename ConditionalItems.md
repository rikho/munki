# Introduction #

This mechanism allows Munki to conditionally install or remove items based on certain conditions.

# Details #

Manifests can contain a "conditional\_items" key/value pair:

```
	<key>conditional_items</key>
	<array>
	  <dict>
	    <key>condition</key>
	    <string>machine_type == "laptop" AND os_vers BEGINSWITH "10.7"</string>
	    <key>managed_installs</key>
	    <array>
	      <string>LionVPNprofile</string>
	    </array>
	    <key>managed_uninstalls</key>
	    <array>
	      <string>CiscoVPNclient</string>
	    </array>
	  </dict>
	  <dict>
	    <key>condition</key>
	    <string>machine_type == "laptop" AND os_vers BEGINSWITH "10.6"</string>
	    <key>managed_installs</key>
	    <array>
	      <string>CiscoVPNclient</string>
	    </array>
	  </dict>
	</array>
```

The above conditional\_items section will cause the CiscoVPNclient to be installed on laptops running Snow Leopard. If such a laptop is later upgraded to Lion, the CiscoVPNclient will be removed and a Lion profile that configures VPN will be installed.

conditional\_items contains an array of dictionaries. Each dictionary contains a "condition" key, whose values is an NSPredicate-style string. Additional keys and values are the same as for the "main" part of a manifest: included\_manifests, managed\_installs, managed\_uninstalls, managed\_updates, and optional\_installs.


### Built-in Conditions ###

Currently available built-in attributes for conditional comparison:

| **Attribute** | **Type** | **Description** | **Example Comparison** |
|:--------------|:---------|:----------------|:-----------------------|
| hostname      | string   | Hostname        | hostname == "LobbyiMac" |
| arch          | string   | Processor architecture. e.g. 'powerpc',  'i386',  'x86\_64'. | arch == "x86\_64"      |
| os\_vers      | string   | Full OS Version e.g. "10.7.2" | os\_vers BEGINSWITH "10.7" |
| os\_vers\_major | integer  | Major OS Version e.g. '10' | os\_vers\_major == 10  |
| os\_vers\_minor | integer  | Minor OS Version e.g. '7' | os\_vers\_minor == 7   |
| os\_vers\_patch | integer  | Point release version e.g. '2' | os\_vers\_patch >= 2   |
| machine\_model | string   | 'Macmini1,1', 'iMac4,1', 'MacBookPro8,2' | machine\_model == "iMac4,1" |
| machine\_type | string   | 'laptop' or 'desktop' | machine\_type == "laptop" |
| catalogs      | array of strings |  This contains the current catalog list for the manifest being processed. | catalogs CONTAINS "testing" |
| ipv4\_address | array of strings | This contains current IPv4 addresses for all interfaces. | ANY ipv4\_address CONTAINS '192.168.161.' |
| munki\_version | string   | Full version of the installed munkitools | munki\_version LIKE '`*`0.8.3`*`' |
| serial\_number | string   | Machine serial number | serial\_number == "W9999999U2P" |
| date          | UTC date string | Date and time. Note the special syntax required to cast a string into an NSDate object. | date > CAST("2013-01-02T00:00:00Z", "NSDate") |

**Notes**

The date attribute was added in 0.8.4.1691.0. The date string must be written in UTC format, but is interpreted as a local date/time.

The condition `date > CAST("2013-01-02T00:00:00Z", "NSDate")` is True if the local time is after midnight local time on 02 Jan 2013.

Comparisons are implemented via Cocoa's NSPredicate objects.
Predicate syntax documented here:
http://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/Predicates/Articles/pSyntax.html

### Admin Provided Conditions ###

Administrators may create their own custom conditions (key/value pairs) by way of authoring condition scripts.  Condition scripts should be placed in `/usr/local/munki/conditions` and can be written in any language of your choosing (shell, python, perl, ruby, etc.) and are evaluated each time `managedsoftwareupdate` is executed (command-line or GUI).

**Requirements**

To be able to provide a stable framework for admin provided conditions, the following guidelines should be followed:

  * Permissions for the conditions directory and its scripts should follow the same standards as preflight/postflight scripts (i.e. same permissions as `/usr/local/munki/managedsoftwareupdate`).
  * Each condition script must output to `ConditionalItems.plist` in the Managed Install directory, which is `/Library/Managed Installs` by default.  Note that no one script should be permitted to completely overwrite this plist as any conditions gathered by previously run script(s) would be lost.
  * Each "write" to `ConditionalItems.plist` must contain a key/value pair.  A value may also be an 'array' of values, similar to the built-in conditions 'ipv4\_address' or 'catalogs'.
  * Items in `/usr/local/munki/conditions` that begin with a period or are directories will be ignored.

**Notes and Considerations** (will be updated as seen necessary)

Authoring condition scripts is a great way to extend your munki administrative capabilities in your environment.  There are however a few items to consider:

  * A condition script's key/value output **will** overwrite another's previous output if the keys are the same.  This applies to both built-in conditions and admin provided conditions.  Example: "machine\_type" is a built-in key, but a condition script has been instructed to use the same key but with a potentially _different_ value or array of values.
  * There's more than one way to write out to `ConditionalItems.plist`, please see the examples for a jumping-off point
  * A condition script may output more than one key/value pair.  Perhaps you'd like to collate all of your custom conditions into one script or keep each condition organized in its own separate script - the choice is yours.
  * Build in an "exit strategy" so that your condition script doesn't hang indefinitely or stall for a lengthy period of time.  Since scripts are executed on each `managedsoftwareupdate` run, the potential exists to bog down overall execution.  Those familiar with preflight/postflight script usage should understand this.
  * If you're troubleshooting and looking for `ConditionalItems.plist` after a `managedsoftwareupdate` run, you won't find it.  Look for the 'ConditionalItems' dictionary ('Conditions' as of 0.8.3 Build 1355) in `/Library/Managed Installs/ManagedInstallReport.plist` for a full list of any conditions your scripts generated.

**Examples** (condition scripts)

This first example (albeit not terribly useful), creates a key/value pair of active hardware ports.  Note that outputted key "hardware\_ports" contains an array of values.  Also note that since we're using the `defaults` command, we must ensure that the `ConditionalItems.plist` is converted to xml and does NOT remain in binary format.

```
#!/bin/sh
# This is an example of a bash conditional script which outputs a key with an array of values
# Please note how the array is created and used within the defaults statement for proper output

# Read the location of the ManagedInstallDir from ManagedInstall.plist
managedinstalldir="$(defaults read /Library/Preferences/ManagedInstalls ManagedInstallDir)"
# Make sure we're outputting our information to "ConditionalItems.plist" (plist is left off since defaults requires this)
plist_loc="$managedinstalldir/ConditionalItems"

IFS=$'\n' # Set our field spearator to newline since each unique item is on its own line
for hardware_port in `networksetup -listallhardwareports | awk -F ": " '/Hardware Port/{print $2}'`; do
	# build array of values from output of above command
	hardware_ports+=( $hardware_port )
done

# Note the key "hardware_ports" which becomes the condition that you would use in a predicate statement
defaults write "$plist_loc" "hardware_ports" -array "${hardware_ports[@]}"

# CRITICAL! Since 'defaults' outputs a binary plist, we need to ensure that munki can read it by converting it to xml
plutil -convert xml1 "$plist_loc".plist

exit 0
```

This second example, while similar to the built-in 'ipv4\_address', generates two key/value pairs; the first is the name of the primary network interface, and the second is its IP address.  Note that they're represented by keys "primary\_interface\_name" and "primary\_ip\_address".  Additionally, we're ensuring that we read the `ConditionalItems.plist` and adding our additional key/value pairs prior to writing the plist back out.  Python's 'plistlib' will completely overwrite an existing plist if this is not performed.

```
#!/usr/bin/python
'''This is a basic example of a conditional script which outputs 2 key/value pairs:
Examples:
primary_interface_name: en0
primary_ip_address: 192.168.1.128

NOTE: Information gathered is ONLY for the primary interface'''

from SystemConfiguration import *    # from pyObjC
import socket
import collections
import os
import plistlib

from Foundation import CFPreferencesCopyAppValue

# Read the location of the ManagedInstallDir from ManagedInstall.plist
BUNDLE_ID = 'ManagedInstalls'
pref_name = 'ManagedInstallDir'
managedinstalldir = CFPreferencesCopyAppValue(pref_name, BUNDLE_ID)
# Make sure we're outputting our information to "ConditionalItems.plist"
conditionalitemspath = os.path.join(managedinstalldir, 'ConditionalItems.plist')

NETWORK_INFO = {}
def getIPAddress(service_uuid):
    ds = SCDynamicStoreCreate(None, 'GetIPv4Addresses', None, None)
    newpattern = SCDynamicStoreKeyCreateNetworkServiceEntity(None,
                                                          kSCDynamicStoreDomainState,
                                                          service_uuid,
                                                          kSCEntNetIPv4)

    newpatterns = CFArrayCreate(None, (newpattern, ), 1, kCFTypeArrayCallBacks)
    ipaddressDict = SCDynamicStoreCopyMultiple(ds, None, newpatterns)
    for ipaddress in ipaddressDict.values():
        ipv4address = ipaddress['Addresses'][0]
        return ipv4address


def getNetworkInfo():
    ds = SCDynamicStoreCreate(None, 'GetIPv4Addresses', None, None)

    pattern = SCDynamicStoreKeyCreateNetworkGlobalEntity(None,
                                                        kSCDynamicStoreDomainState,
                                                        kSCEntNetIPv4);
    patterns = CFArrayCreate(None, (pattern, ), 1, kCFTypeArrayCallBacks)
    valueDict = SCDynamicStoreCopyMultiple(ds, None, patterns)

    ipv4info = collections.namedtuple('ipv4info', 'ifname ip router service')

    for serviceDict in valueDict.values():
        ifname = serviceDict[u'PrimaryInterface']
        NETWORK_INFO['interface'] = serviceDict[u'PrimaryInterface']
        NETWORK_INFO['service_uuid'] = serviceDict[u'PrimaryService']
        NETWORK_INFO['router'] = serviceDict[u'Router']
        NETWORK_INFO['ip_address'] = getIPAddress(serviceDict[u'PrimaryService'])

        netinfo_dict = dict(
            primary_interface_name = ifname,
            primary_ip_address = NETWORK_INFO['ip_address'],
        )

        # CRITICAL!
        if os.path.exists(conditionalitemspath):
            # "ConditionalItems.plist" exists, so read it FIRST (existing_dict)
            existing_dict = plistlib.readPlist(conditionalitemspath)
            # Create output_dict which joins new data generated in this script with existing data
            output_dict = dict(existing_dict.items() + netinfo_dict.items())
        else:
            # "ConditionalItems.plist" does not exist,
            # output only consists of data generated in this script
            output_dict = netinfo_dict

        # Write out data to "ConditionalItems.plist"
        plistlib.writePlist(output_dict, conditionalitemspath)

getNetworkInfo()
```

This last example, shows a custom condition used as a "conditional\_item" within a manifest.  It specifically demonstrates that if any of the active hardware ports contain the string "Wi-Fi", the 'managed\_install' package, "TestPackage", will be considered for deployment.
```
<key>conditional_items</key>
<array>
    <dict>
        <key>condition</key>
        <string>ANY hardware_ports CONTAINS 'Wi-Fi'</string>
        <key>managed_installs</key>
        <array>
            <string>TestPackage</string>
        </array>
    <dict>
</array>
```

### Nested conditional\_items ###

If you'd like to achieve a higher degree of granularity, you may also nest conditional\_items:

```
  <key>conditional_items</key>
  <array>
    <dict>
      <key>condition</key>
      <string>machine_type == "laptop"</string>
      <key>conditional_items</key>
      <array>
        <dict>
          <key>condition</key>
          <string>os_vers BEGINSWITH "10.7"</string>
          <key>managed_installs</key>
          <array>
            <string>LionVPNProfile</string>
          </array>
        </dict>
        <dict>
          <key>condition</key>
          <string>os_vers BEGINSWITH "10.6"</string>
          <key>managed_installs</key>
          <array>
            <string>CiscoVPNClient</string>
          </array>
        </dict>
      </array>
    </dict>
  </array>
```

which is functionally the same as:

```
  <key>conditional_items</key>
  <array>
    <dict>
      <key>condition</key>
      <string>machine_type == "laptop" AND os_vers BEGINSWITH "10.7"</string>
      <key>managed_installs</key>
      <array>
        <string>LionVPNProfile</string>
      </array>
    </dict>
    <dict>
      <key>condition</key>
      <string>machine_type == "laptop" AND os_vers BEGINSWITH "10.6"</string>
      <key>managed_installs</key>
      <array>
        <string>CiscoVPNClient</string>
      </array>
    </dict>
  </array>
```

But either style is fine, and either might be easier to maintain/understand in different circumstances.