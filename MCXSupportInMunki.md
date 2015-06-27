# Introduction #

Version 0.7.0 of the munki tools adds support for MCX management of the munki configuration. You may now use Apple's Managed Preferences to manage the munki configuration on munki clients.


# Details #

Since munki preferences are not user-level preferences, it probably makes the most sense to manage these at the Computer or ComputerGroup level.

To manage one or more munki preferences using MCX, use the Preferences->Details pane in Workgroup Manager to import a configured /Library/Preferences/ManagedInstalls.plist. Choose the option "Manage imported preferences: Always". (Once and Often work only for user-level preferences.)

**Do not** manage the following preferences, or you may see unexpected/undesired behavior:
  * LastCheckDate
  * LastCheckResult
  * LastNotifiedDate

If any of these keys are imported when you import /Library/Preferences/ManagedInstalls.plist, make sure to delete them in Workgroup Manager.

ManagedInstalls keys defined in MCX take precedence over the same key defined in /var/root/Library/Preferences/ManagedInstalls.plist and /Library/Preferences/ManagedInstalls.plist.

But wait, there's more: Managed Software Update.app contains an MCX manifest ready for import by Workgroup Manager!