### About Upgrading ESOS ###
First, if the storage server is not part of a cluster, the upgrade process for Enterprise Storage OS is disruptive and will require a small amount of down time. There are a couple different methods/options for upgrading ESOS:
  * For the paranoid, you can install the new version of ESOS to an entirely new, fresh USB flash drive. You'll then copy your configuration files from the current (old) ESOS flash drive to the new USB device. Finally you remove the old flash drive from the server, and replace it with the new flash drive and reboot your storage server. While this upgrade procedure is cumbersome, and does involve some down time, it has one important benefit: It is very easy to revert to your old ESOS version if there is some type of problem when running the new version -- simply power off the storage server, and swap USB flash drives (use the old one).
  * Another more practical approach to upgrading ESOS: Either take the existing USB flash drive from the running server, or an entirely new USB device and use the ESOS installer script (on another system) to write the new image file and add any proprietary RAID CLI tools. Then plug the new (or updated) drive into the running ESOS server, and synchronize the configuration (in the TUI or CLI with `conf_sync.sh`). This will dump the running configuration to the USB flash drive. Double-check your configuration files are there, and then reboot.
  * The final method is for the brave (or confident). This would be an ideal upgrade method if you don't easily have access to the physical server. You will also need enough physical RAM on the server so you extract the Enterprise Storage OS installation package. You'll transfer the ESOS tarball to the server, extract it, and then run the installer (`install.sh`) as normal. Once the image has been written and any proprietary tools added, you can then synchronize the configuration (in the TUI or CLI with `conf_sync.sh`). Check that all of your configuration is on the flash drive (esos\_conf) and reboot.

_It is highly recommended to label your ESOS USB flash drives with some details such host name, ESOS version, date, etc. to keep organized._

Choose an upgrade method and follow the steps below to upgrade Enterprise Storage OS.

<br>

<h3>Upgrade Procedure # 1 (Paranoid)</h3>
The first step is to install the new ESOS version to an entirely new USB flash device. Follow the <a href='12_Installation.md'>12_Installation</a> guide.<br>
<br>
After you have successfully created your new ESOS flash drive, its probably not a bad idea to try it on another computer/server to at least make sure it boots. This could save you some down time if there is a problem with the new ESOS USB device.<br>
<br>
Since ESOS is a sort of memory resident operating system, you can open an ESOS shell and run <code>conf_sync.sh</code> to synchronize your configuration files. Next, remove the ESOS USB flash drive from the running, powered on ESOS storage server you are upgrading.<br>
<br>
For the next step, you will need to make use of a Linux workstation that has accessible USB ports. First, plug in the current/old ESOS USB drive and run <code>lsscsi</code> to get the device node:<br>
<pre><code>[2:0:0:0]    disk    ATA      ST9160412AS      D005  /dev/sda <br>
[3:0:0:0]    cd/dvd  TSSTcorp DVD+-RW TS-L633C DW50  /dev/sr0 <br>
[9:0:0:0]    disk    Kingston DataTraveler G3  1.00  /dev/sdb <br>
</code></pre>

We can see that '/dev/sdb' is the device node for our old ESOS flash drive; we can now mount the ESOS configuration filesystem:<br>
<pre><code>mkdir /mnt/old_esos<br>
mount /dev/sdb3 /mnt/old_esos<br>
</code></pre>

Now plug in the newly created (updated) ESOS flash drive to the Linux workstation and grab the device node:<br>
<pre><code>[2:0:0:0]    disk    ATA      ST9160412AS      D005  /dev/sda <br>
[3:0:0:0]    cd/dvd  TSSTcorp DVD+-RW TS-L633C DW50  /dev/sr0 <br>
[9:0:0:0]    disk    Kingston DataTraveler G3  1.00  /dev/sdb <br>
[10:0:0:0]   disk    Imation  Clip             PMAP  /dev/sdc <br>
</code></pre>

The new/updated ESOS USB drive device node is '/dev/sdc' in this example. Mount the configuration filesystem:<br>
<pre><code>mkdir /mnt/new_esos<br>
mount /dev/sdc3 /mnt/new_esos<br>
</code></pre>

Finally copy over everything from the old drive to the new drive:<br>
<pre><code>cp -pR /mnt/old_esos/* /mnt/new_esos/<br>
umount /mnt/old_esos<br>
umount /mnt/new_esos<br>
</code></pre>

You can now insert the newly created (upgraded) ESOS USB flash drive into the USB port on your ESOS storage server, and then reboot the machine. When it comes back up, it will be running off of the new USB drive and the upgraded version of ESOS will be loaded.<br>
<br>
<br>

<h3>Upgrade Procedure # 2 (Normal)</h3>
With this method, you can use either a brand new USB flash drive (safer) or remove the existing USB device from your running ESOS server. Take that flash drive to another Linux system and follow the <a href='12_Installation.md'>12_Installation</a> wiki page.<br>
<br>
Once the image has been installed to the flash drive, insert the USB drive into your running Enterprise Storage OS server, and run <code>conf_sync.sh</code> to synchronize your configuration with the new device.<br>
<br>
<br>

<h3>Upgrade Procedure # 3 (Brave)</h3>
This is the "in-place" upgrade method and is best used when you don't have physical access to the server, or you'd like to be <i>efficient</i> in your work. This method requires you have enough physical RAM in your server as the ESOS installation package will be transferred to the running Enterprise Storage OS server (via SCP/SFTP) and then extracted.<br>
<br>
So, the first step is to make sure you have enough space in the /tmp file system. The /tmp tmpfs file system defaults to half of your available physical RAM. It needs to be big enough to hold the ESOS install tarball, the extracted image (4 GB), and any proprietary tools you want copy up for installation. Use this command to change the size of the /tmp file system (don't make it bigger than your amount of physical RAM):<br>
<pre><code>mount -o remount,size=5G /tmp<br>
</code></pre>

Next we need to get the new ESOS package to your running storage server:<br>
<pre><code>scp esos-0.1-r579.tar.xz root@10.26.2.70:/tmp<br>
</code></pre>

Now you can proceed with the normal installation procedure found on the <a href='12_Installation.md'>12_Installation</a> guide page. You'll SCP/SFTP any proprietary CLI RAID tools to the running ESOS storage server location (that the install script gives you).<br>
<br>
After the install is complete, run <code>conf_sync.sh</code> to synchronize your configuration with the USB flash drive. Its not a bad idea to double-check your configuration synchronized correctly by mounting the <code>esos_conf</code> file system and giving it a once-over. Then you can reboot and you'll be running the new Enterprise Storage OS image!<br>
<br>
<br>

<h3>Post Update</h3>
Be sure to keep your old/previous ESOS USB flash drive handy if you chose a method that allows this! Should you have any problems running the new version of ESOS, you will want that old, known-to-work copy of ESOS so you can revert quickly.