# Introduction #

Adding custom help for Managed Software Center

# Details #

As of build 2.0.0.2163, Managed Software Center will check [Munki's preferences](https://code.google.com/p/munki/wiki/configuration) for a HelpURL key. If that key is present and the user chooses "Managed Software Center Help" from Managed Software Center's Help menu, that URL will be opened using the URL's default application. (In other words, http/https URLs will be opened by the user's default browser, etc.)