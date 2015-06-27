# Introduction #

Where to start troubleshooting when things aren't working as you expect.

See also the [FAQ](http://code.google.com/p/munki/wiki/FAQ)


# Details #

### Command line ###
Run `/usr/local/munki/managedsoftwareupdate -vvv` to get more verbose output.

### Logs ###
By default, logs are written to /Library/Managed Installs/Logs/

ManagedSoftwareUpdate.log is the main log; errors and warnings from the most recent run are in errors.log and warnings.log, respectively.

### Server troubleshooting ###
The Munki client talks to the Munki server via normal web protocols -- you may be able to troubleshoot by using a web browser to download the same info:

http://yourmunkiserver/repo/manifests/name_of_client_manifest

http://yourmunkiserver/repo/catalogs/production

http://yourmunkiserver/repo/catalogs/testing

**Still stuck?** Ask for help here: http://groups.google.com/group/munki-dev