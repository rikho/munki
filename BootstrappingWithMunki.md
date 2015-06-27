# Introduction #

Many Mac administrators have adapted a modular approach to imaging using tools like InstaDMG or Apple's System Image Utility. The "modules" of the modular approach are Apple packages, and these tools build installation images using these packages.

If you are using Munki, you might consider making your images very "thin" and using Munki to bootstrap a machine's configuration.


# Details #

The concept here is simple. Instead of a lengthy process of building an installation image from a great number of packages, you build a "thin" image that consists of the OS, perhaps an admin account, and the munki tools. You restore the image to the target machine, and upon reboot, the munki tools take over and complete the configuration of the machine by installing all the rest of the software your organization needs, including the majority of your configuration packages.

The "thin" image needs to have only enough to allow the target machine to boot and run the munki tools; Munki installs everything else, including any needed Apple Software Updates.

To cause munki to check for updates and install at startup, make sure the file **`/Users/Shared/.com.googlecode.munki.checkandinstallatstartup`** exists. You can create a package that installs that file and make it part of your thin image, or you can have a script `touch` that file.

When `/Users/Shared/.com.googlecode.munki.checkandinstallatstartup` exists, munki will check for and install updates. If a reboot is required after installing updates, after the reboot, munki will check and install again, continuing until there are no updates to be installed. This allows you to bootstrap the configuration of a machine, including installing Apple Software Updates, with no user intervention.

**Pros -** Some advantages of this approach:

  1. Less duplication of effort. You don't need to add packages to both Munki and a modular disk image workflow.
  1. More packages work. There are a non-insignificant number of packages that fail to install correctly in a modular imaging workflow because scripts within these packages make invalid assumptions. Munki installs packages to the current startup disk, which is more likely to result in success with these problematic packages.
  1. Can be used with a "no imaging" approach. Taking the "thin imaging" concept one step further, you may be able to deploy machines without imaging at all. This begins with taking a new Mac out of the box, using the image installed on it at the factory. You install the munki tools only, and allow munki to reconfigure the machine to your standard. This approach allows you to rapidly deploy new hardware releases from Apple without needed to take the time and resources to build a new deployment image.

**Cons -** Some disadvantages of this approach:

  1. Slower. Configuring a machine this way is much slower than block-copying a "pre-compiled" modular image built with InstaDMG, SIU or similar tools.
  1. Integration with other tools - it can be tricky at times to get integration correct. For example, DeployStudio now does some post-imaging tasks on the first reboot after installing an image. If Munki was set to run on the first reboot as well, the DeployStudio scripts could reboot the machine in the middle of a Munki run. The solution in this case is to make the creation of the `/Users/Shared/.com.googlecode.munki.checkandinstallatstartup` file one of the tasks DeployStudio performs after the first reboot. When DeployStudio reboots the machine again, Munki will run on the second reboot.