# Introduction #

One can enable exact logging of user actions in the Managed Software Update (MSU) GUI app. This can help to verify that various events within the GUI are occuring, e.g.

  * When did a user last see a popup notification to update?
  * Did a user choose to install updates, or defer until later?
  * Did the user see available Apple updates?
  * Are available optional software choices being reviewed?

With MSU logging the Munki admin can determine these facts.

# Enable Logging #

By default logs are disabled. To enable MSU logging, set the key MSULogEnabled to True in [ManagedInstalls.plist](configuration.md) on Munki clients:

> `defaults write /Library/Preferences/ManagedInstalls MSULogEnabled -bool TRUE`

# Log Location and Format #

Logs are written to the directory:

> `/Users/Shared/.com.googlecode.munki.ManagedSoftwareUpdate.logs/`

Multiple log files may exist in this directory. Each user who runs MSU will have 1 or more log files written.  By default the log filename will be "_username_.log", however if permissions problems occur additional log files will be written with random suffixes. This is done to avoid potential security problems due to the global writable nature of /Users/Shared/.

The log file consists of one text representation of a log event after another, separated by newlines.

> `float_timestamp INFO username : @@source:event@@ description`

e.g.
> `1300743459.788060 INFO user : @@MSU:appleupdates@@`

Note that log files are opened, written to, and closed upon each single log event write, even during one MSU application instance. Therefore external third party tools to roll or process logs should not have much problem with dangling open files, etc.

# Log Event Properties #

Each log event has 4-5 properties:
  * Timestamp at which the item was written
  * Username of the logged in user
  * Source of the event (source)
  * Event name (event)
  * Optional descriptive text (desc) to supply more detail.

The following table describes source/event pairs in more detail:

| **source** | **event** | **desc** | **Description** |
|:-----------|:----------|:---------|:----------------|
|MSU         | launched  |          | MSU launched and presented the initial GUI window to interact with the user.|
|MSU         | exit\_munkistatus |          | This MSU instance launched only to present a status window, and now it is exiting.|
|MSU         | exit\_installwithnologout|          | MSU is now exiting after being told to install without logout.|
|MSU         | no\_updates|          | MSU was started manually but no updates are available. |
|MSU         | cant\_update | cannot contact server| Underlying munki cannot contact the server, therefore updates cannot occur (and MSU is reporting this).|
|MSU         | cant\_update| failed preflight| Underlying munki tried to run managedsoftwareupdate, preflight failed, therefore updates cannot occur.|
|MSU         | conflicting\_apps| (application names)| An application could not be installed because conflicting apps in (desc) are running.  The user was told to quit these apps.|
|MSU         |cannot\_start|          | Total configuration problem preventing managedsoftwareupdate from running.|
|MSU         | appleupdates |          | Available Apple Software Update packages were presented to the user.|
|user        | cancelled |          | User was given the choice, after deciding to install packages, to optionally install with logout, or just stay logged in and perform the install.  Instead of picking an install action, the user hit "cancel" and did not complete the choice, thus aborting the install.|
|user        | exit\_later\_clicked |          | The user clicked the "later" button and deferred installation of available packages until next MSU run. |
|user        | install\_with\_logout|          | The user selected the "install and logout" option.|
|user        | install\_without\_logout|          | The user selected the "install without logout" option. |
|user        | quit      |          | MSU is exiting after doing nothing, either because of errors (like MSU:cant\_update) or because no updates were available to install.|
|user        | view\_optional\_software |          | User clicked the View Optional Software button.|

# Log Growth Behavior #

Once logs are enabled, the logs will be written to at any point where the MSU GUI generates an event. It is up to the Munki admin to process these logs in some way, and also to roll them away and/or clean them up.

One potential place to perform log harvesting and cleanup would be in a preflight or postflight script.