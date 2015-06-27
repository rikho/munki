<a href='Hidden comment: 
Template:
====Q: Question====
*A*: Answer
'></a>

#### Q: How do you pronounce Munki? ####

**A**: Same as "monkey."<br />

#### Q:  Does Munki stand for something? ####

**A**: Nope.  Just a fun name that evokes "helper monkeys."<br />

#### Q: What package formats does Munki support? ####

**A**: Munki supports the following formats:
  * Apple-flat packages (.pkg)
  * Apple-flat meta-packages (.mpkg)
  * Apple-non-flat packages (.pkg)`*`
  * Apple-non-flat meta-packages (.mpkg)`*`
  * Drag-and-drop disk images
  * Adobe CS3/CS4/CS5/CS6 Deployment "packages" created with Adobe's Enterprise Deployment tools - [CS3 info](http://www.adobe.com/support/deployment/cs3_deployment.pdf)  [CS4/CS5/CS6 info](http://www.adobe.com/devnet/creativesuite/enterprisedeployment.html)
  * Many Adobe CS3/CS4/CS5/CS6 product updaters
  * Adobe Acrobat Pro 9.x updater disk images as [downloaded from Adobe](http://www.adobe.com/support/downloads/product.jsp?product=1&platform=Macintosh)

`*` Must be wrapped in a disk image file (.dmg).<br />

#### Q: How does Munki determine if the correct version of an application is installed? ####

**A**: If the path to the application appears in the "installs" array, Munki will first check that path.  If the application is found at the given path, its CFBundleShortVersionStrings string is checked.  If it matches what's listed in the the manifest, Munki knows the correct version is installed.  If the application isn't found at the given path, system\_profiler is used to gather a list of all installed applications.  If the application is listed with the correct version number, Munki knows the correct version is installed.  If it's not listed, or it has an earlier version number, Munki knows the application must be installed.  To speed up searching for applications, Munki caches the list of applications returned by system\_profiler.<br>
If there is no "installs" list, Munki relies on package receipts listed in the "receipts" array to determine installation status.<br>
See HowMunkiDecidesWhatNeedsToBeInstalled for even more information on this topic.<br />

<h4>Q: How often do Munki clients check for updates?</h4>

<b>A</b>: On average, once an hour. The exact time between checks is randomized somewhat to prevent every client from hitting the Munki server all at once. A launchd job defined at /Library/LaunchDaemons/com.googlecode.munki.managedsoftwareupdate-check.plist runs once an hour. It, in turn, calls /usr/local/munki/updatecheckhelper, which sleeps a random time between 0 and 60 minutes before checking with the server. This means that the time between any two Munki runs can vary from a few minutes to almost two hours.<br />

<h4>Q:  How is a user notified of available updates?</h4>

A: An application named "Managed Software Update", which looks very much like Apple's Software Update application, will open and display available updates. Like with Apple's Software Update, the user can then choose to install the updates now, or wait until later. Unlike with Apple's Software Update, the user cannot pick and choose among the updates. Those are managed by the Munki administrator. If any update requires a logout or restart, Managed Software Update triggers a logout before proceeding. Otherwise, the user can choose to update without logging out.<br />

<h4>Q:  What happens if the user chooses to update without logging out, but some of the updates are for applications that are currently open?</h4>

<b>A:</b> Munki can check for certain applications and notify the user to quit them before proceeding. See BlockingApplications for more info on this feature.<br />

<h4>Q:  What if there is no user logged in?</h4>

<b>A</b>: Munki will install available updates if there is no user logged in and the machine has been idle for 10 seconds or longer. A status window is displayed, and the loginwindow is hidden so that no-one can login while updates are occurring.<br />

<h4>Q:  Can Munki install Apple Software Updates?</h4>

<b>A</b>: Yes. See AppleSoftwareUpdatesWithMunki for more information.<br />

<h4>Q: I keep getting 'Can't install Foo-1.0 because the integrity check failed.'</h4>

<b>A</b>: Most likely the disk image containing Foo-1.0 is a read/write disk image. Possible solutions:<br>
<ul><li>Convert the disk image to read-only and reimport the item.<br>
</li><li>Recreate the disk image as read-only and reimport the item.<br>
</li><li>Configure Munki to not verify package checksums (not recommended). See the PackageVerificationMode key in <a href='http://code.google.com/p/munki/wiki/configuration'>http://code.google.com/p/munki/wiki/configuration</a>.<br /></li></ul>

<h4>Q: I keep seeing warnings like <i>WARNING: Could not process item Office2011_update-14.4.2 for update. No pkginfo found in catalogs: production</i>, yet there is definitely an item named "Office2011_update-14.4.2" in the production catalog. What is happening?</h4>

<b>A</b>: A name followed by a hyphen and a version number (or a number!) has special meaning: it means NAME-VERSION. So Munki is actually looking for an item named "Office2011_update" with a version of "14.4.2". It is NOT looking for an item with the name "Office2011_update-14.4.2". To avoid this type of confusion, don't put versions into names. It's rarely good practice to do so. If you must, don't precede the version with a hyphen. (Any number preceded by a hyphen is likely to be interpreted as a version number.)<br />

<h4>Q: Each time Munki runs, it wants to install the same software again. Why is this?</h4>

<b>A</b>: The most likely explanation is that the install has failed. On the next run, Munki sees the item is not installed, and tries again. If the item actually <i>is</i> installed, see the next question...<br />

<h4>Q: Munki <i>successfully</i> installed some software, but now each time Munki runs, it wants to install the software again. Why is this?</h4>

<b>A</b>: Munki uses one of two arrays in the pkginfo to determine if an item is installed. If the "installs" array exists, each item in the array is checked; if it does not exist or the currently installed version is older than the one described in the pkginfo, Munki will attempt to install the item. If there is no "installs" array, Munki will use the "receipts" array, again installing if any receipt is missing or is an older version that that described in the pkginfo.<br>
If Munki repeatedly presents an item for install after a successful installation, then one or more items in the "installs" array is not being installed, or one or more items in the "receipts" array is not being recorded in the receipts database. You'll need to determine what is not being installed and remove it from the installs or receipts array. (Or in the case of a receipt, mark it as "optional".) Alternately, if there is no installs array, you can often resolve this issue by <i>adding</i> an installs array.  See HowMunkiDecidesWhatNeedsToBeInstalled for even more information on this topic.<br />

<h4>Q: Can I deploy Mac App Store apps using Munki?</h4>

<b>A</b>: In most cases, yes. See <a href='AppStoreApps.md'>App Store Apps</a><br />

<h4>Q: Why does does the munkitools.mpkg/launchd.pkg require a restart? This prevents an unattended upgrade of the Munki tools!</h4>

<b>A</b>: It is non-trivial to load launchd jobs, especially user-level LaunchAgents, in the correct Mach context from a package postinstall script in all possible execution contexts in which a package can be installed. But more importantly, the launchd jobs control Munki itself, and so Munki could not unload and reload these jobs without killing itself during the unload. So for maximum reliability, we require a restart so we ensure the jobs are loaded in the correct context and that Munki can actually complete the task!<br />
As for unattended updates of the Munki tools, in most cases it <i>is</i> possible to upgrade them without requiring a logout or restart. See UpdatingMunkiTools for additional information.<br />