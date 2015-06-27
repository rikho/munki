# Introduction #

Munki releases are made available via Apple packages here:
http://code.google.com/p/munki/downloads/list

These packages are built and released only for major development milestones. If you want to be on the cutting edge, or want to test a new feature before the official release, you may need to build your Munki tools from source.

# Details #

### Requirements ###

**A Leopard or Snow Leopard machine.** See the next requirement.

**Xcode 3**<br>
Currently, to successfully build the Munki tools from source, you must be using Xcode 3.x on Leopard or Snow Leopard. Building on Lion is not currently supported because there is no way to build "universal" applications that run on PowerPC and Intel under Xcode 4. If you don't care about PowerPC compatibility, it is possible to use Xcode 4 on Lion, but this document does not describe that yet.<br>
<br>
<b>Git</b><br>
While not absolutely, strictly necessary to build the Munki tools, in order to follow this document you must also install Git. Git is a distributed revision control system. It is not installed by default on Leopard or Snow Leopard. You can find installation packages here: <a href='http://code.google.com/p/git-osx-installer/downloads/list'>http://code.google.com/p/git-osx-installer/downloads/list</a>

<h3>Building Munki</h3>

So now a bit of the chicken-and-the-egg problem. You need the Munki build script available here:<br>
<br>
<a href='http://munki.googlecode.com/git/code/tools/make_munki_mpkg_from_git.sh'>http://munki.googlecode.com/git/code/tools/make_munki_mpkg_from_git.sh</a>

You can quickly download and execute it like so:<br>
<br>
<pre><code>curl http://munki.googlecode.com/git/code/tools/make_munki_mpkg_from_git.sh | bash<br>
</code></pre>

This uses curl to download the latest make_munki_mpkg_from_git.sh script, and pipes it to the bash shell, which executes it.<br>
<br>
When it's done, in the current directory, you'll have a Git clone of the current Munki code (munki-git) and a metapackage of the tools.<br>
<br>
<h3>That's still too much work!</h3>

Timothy Sutton has set up an autobuild server for Munki, available here:<br>
<br>
<a href='http://munkibuilds.org'>http://munkibuilds.org</a>

This machine automatically builds and makes available the the most recent Git commits. If you want to test a Git revision and don't want to/can't build it yourself, you can take advantage of these automatic builds.