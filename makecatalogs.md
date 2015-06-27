# Details #

Usage: `/usr/local/munki/makecatalogs [options] [/path/to/repo]`

Options:
| -h | --help | show this help message and exit |
|:---|:-------|:--------------------------------|
| -V | --version | Print the version of the munki tools and exit. |

This tool scans the repo/pkgsinfo directory and builds catalogs from the pkginfo files.

You should run this after any change to pkginfo items to update all catalogs. `munkiimport` will offer to run this for you after a successful import.

`/path/to/repo` is optional if you have configured the path to the Munki repo using `munkiimport --configure` or `manifestutil --configure`. If this value is absent, the value stored in ~/Library/Preferences/com.googlecode.munki.plist will be used.

The pkgsinfo subdirectory must exist and be readable; the catalogs subdirectory must exist and be writable.