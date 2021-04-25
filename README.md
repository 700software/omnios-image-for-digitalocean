Hosting OmniOS on DigitalOcean by creating a Custom Image (create with VirtualBox)
=========================================================

What we are trying to do is run OmniOS on Digital Ocean.
This is for those who want an alternative to AWS or Joyent.

Reasoning and background
------------------------

OmniOS is an Illumos distribution. (open-source and free, unlike Oracle's Solaris)
Illumos has been updated from OpenSolaris which was maintained by Sun Microsystems before their acquisition.

OmniOS / Illumos natively support ZFS. This is by far the best filesystem supporting best-in-class crash-recovery,
features like compression and deduplication, resizable partitions and filesystems.
Furthermore Illumos supports ZFS as the root filesystem; and uses this to enhance OS upgrade seamlessness.

There have been efforts to provide compatibility of Linux executables to run on Illumos as well.
Considerings Zones, DTrace and other features, it would seem that OmniOS is superior,
but with a much shorter history of being open-source compared to Linux and other contendors.
I personally believe in OmniOS as an OS, and highly recommend ZFS for any bare-metal installations.

ZFS has great features even for VM, but with bare-metal the benefits become overwhelming.
We may or may not transition there, but for now we are going VM.

So far, the only low-cost options for OmniOS or Illumos VM / server hosting seem to be

 - AWS
    - [available as AMI]
    - they support [woke] agenda and other un-Christian behaviour
    - their interface is more complex and pricing is less transparent
 - Joyent
    - only supports SmartOS
    - seems focused on bigger accounts

The best alternative I could find so far was

 - DigitalOcean
    - Best known for: simple user interface, better pricing than AWS for small projects
    - A custom image must be built to run this OS. (Thus creation of this repo.)
    - They chose hCaptcha over reCaptcha, which implies a prioritization of privacy
    - They made [no comment on BLM], which implies hesitation to go [woke].
    - support competition in the marketplace by not using AWS, Azure, Google Cloud

Unlike SmartOS (Joyent's preference), OmniOS supports traditional ISO installations and boot-from-disk we take for granted, without requiring network or USB boot each time. This simplifies future migration to bare-metal, as well as diversifying with temporary VirtualBox installations as-needed.

Unlike OpenIndiana, OmniOS is lightweight, focused on server environments over desktop.

Creating the image from scratch
-------------------------------

using latest ISO image from https://omnios.org/download.html

1. Start download of the ISO image
2. Download and install [VirtualBox]
3. In VirtualBox, create a New viritual machine
   - Name: My OmniOS Image
   - Type: Solaris
   - Version: Oracle Solaris 10/09 and later (64-bit)
   - (Next) Disk capacity can be 8 GB instead of 16 GB.+
4. Start the VM (Normal start)
5. Add the ISO image (from step one) as the startup disk
6. Install with all default options. (Remember, you are creating a generic image right now.)
7. Configure the OS similar to
8. Unmount the CD and shut down the VM

Then you can upload the image to DigitalOcean.
Then start the droplet. It will ask you to set the root SSH key pair, but keep in mind that will not be applied... It is not supported because

 - we have not configured cloud-init, (speaking of which, I think [illumos/metadata-agent] might actually work for this) and
 - possibly there'd be an issue with us not using ext3 or ext4 as root filesystem but that may not be an issue.

So what we will do is

1. Go ahead anyway and upload, start a Basic droplet up. 
2. Open the droplet's **Console** on DigitalOcean
3. Type `dladm`. Notice it has `vioif1` (instead of the `e1000g0` that it had in VirtualBox environment)
4. Type `ipadm`. Notice it has only the loopback interfaces.
5. Run `ipadm create-if vioif1; ipadm create-addr -T dhcp vioif1/dhcp`
6. Test the internet connectivity with `dig +short google.com @8.8.8.8`
7. Configure default DNS servers in `/etc/resolv.conf`
8. Set up a user for remote access, set up their `sudo` and set a `root` password.


  [available as AMI]: https://omnios.org/download.html
  [woke]: https://radio.foxnews.com/2021/01/28/ben-shapiro-on-the-woke-lefts-quest-to-deplatform-conservative-media/
  [no comment on BLM]: https://www.reddit.com/r/digital_ocean/comments/gvhwyz/has_digitalocean_provided_any_communication_on_blm/
  [VirtualBox]: https://www.virtualbox.org/
  [illumos/metadata-agent]: https://github.com/illumos/metadata-agent
