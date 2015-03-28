### Getting Started ###
There are two broad steps when provisioning storage with ESOS: The first is configuring your back-end storage device which is typically a high-end RAID controller, or virtual disk files on file systems, or can be anything else that appears as a SCSI block device in Linux. Enterprise Storage OS includes a number of different Linux block device subsystems such as DRBD, Linux software RAID (md), and LVM2 so the possibilities are quite endless.

The second step is configuring your front side target devices, the targets that are made available via your SAN. _This will be covered in the subsequent ESOS wiki pages (hosts/initiators, devices, and targets)._

<br>

<h3>RAID Controller TUI Functions</h3>
ESOS has limited support for provisioning/configuring logical drives via the TUI. Currently only LSI Logic MegaRAID adapters are supported using the TUI logical drive / adapter functions. Using these functions requires the MegaCLI tool which is an option during the USB flash drive installation. Advanced adapter properties, multi-span logical drives, etc. are not supported using the TUI.<br>
<br>
A few basic RAID controller/adapter properties are configurable via the TUI: Back-End Storage -> Adapter Properties<br>
<br>
Using the adapter information dialog (Back-End Storage -> Adapter Information) will allow you to select a RAID controller and will display an overview of the attached storage (enclosures + devices).<br>
<br>
To add a new logical drive (Back-End Storage -> Add Volume) you will first select the desired RAID adapter; next you will be presented with screen that contains the available disks. Select the disks and hit ENTER (or ESCAPE to cancel). On the final dialog, you will choose the RAID level, strip size, cache options, etc. and then hit 'OK' to proceed with provisioning the volume or 'Cancel' to do nothing and close the dialog.<br>
<br>
Deleting volumes / logical drives (Back-End Storage -> Delete Volume) and modifying volume properties (Back-End Storage -> Volume Properties) is also possible via the TUI.<br>
<br>
After you have configured your RAID logical drive(s), you can now proceed to the advanced back-end storage document, or below for creating a file system on the new device and adding virtual disks, or move on to adding your SCST device(s).<br>
<br>
<br>

<h3>Back-End Storage File Systems</h3>
Once you have your back-end storage block device(s) configured, either using the basic methods described above, or an advanced block device (DRBD, LVM2, etc.) configured, you can then create a file system on these devices if you choose. The purpose of doing this would then allow you to use the SCST FILEIO mode for your devices. The FILEIO mode has a couple differences from the SCST BLOCKIO mode: 1) FILEIO mode takes full advantage of the Linux page cache, and 2) allows you to have multiple "virtual disk" devices on a single back-end storage block device.<br>
<br>
While you may see better I/O performance when using FILEIO mode on systems with slower back-end storage, a big advantage is the ability to have multiple virtual disk files on a single block device (eg, RAID logical drive, DRBD device, md device, LVM2 device, etc.) in which each appear and function as an independent volume to the remote initiators.<br>
<br>
Here is an example of such a configuration:<br>
<ul><li>A RAID5 volume is configured consisting of (4) 3 TB disks giving you less than 9 TB of usable disk space.<br>
</li><li>You can now create a file system on this new logical drive (volume) and you have less than 9 TB of space to create various virtual disks.<br>
</li><li>Then you decide to create (3) 50 GB virtual disk files for your three new servers (eg, boot_host1, boot_host2, boot_host3).<br>
</li><li>Now you want some data volumes for each server: (2) 500 GB data volumes (eg, host2_data1, host2_data2); (3) 100 GB data volumes (eg, host3_data1, host3_data2, host3_data3).<br>
</li><li>You'll then have (8) virtual disk files (volumes) available to your servers with a single back-end storage block device.</li></ul>

Obviously your specific I/O requirements (performance, space, etc.) will all come into play on how many virtual disk files you can create on each back-end storage block device, but that should give you the general idea of this FILEIO mode concept.<br>
<br>
Now, to create a file system on your newly configured back-end storage block device, go to the create-file-system dialog in the TUI (Back-End Storage -> Create File System). You'll be prompted to select a block device, and then on the next dialog screen you'll be show the current disk label layout (partition table) if any exists. Type a useful FS label name for the new file system, and choose the file system type (typically 'xfs' provides the best performance). Then hit 'OK' and a new GPT partition (disk label) will be created on the block device spanning the entire device and the new file system will be created on the first (only) partition. If prompted to mount the new file system, do so; your new file system is ready for virtual disk files, continue below.<br>
<br>
You can also easily remove/delete file systems in the TUI (Back-End Storage -> Remove File System). The file system will be unmounted and removed from the fstab file.<br>
<br>
<br>

<h3>Virtual Disk Files</h3>
Now that you have a file system created/mounted and available (described in the section above) you can create virtual disk files. These virtual disk files are independent storage volumes that can be shared with initiators, which is described in the subsequent ESOS wiki pages.<br>
<br>
Adding virtual disks is quite easy; in the TUI open the dialog (Back-End Storage -> Add Virtual Disk File). You'll be shown the total disk space on the file system, and the available disk space. You'll need to pick a meaningful name for the virtual disk file and the new virtual disk size -- this will be the actual virtual disk size that is presented to the initiators. Hit 'OK' can your new virtual disk will be created; the file needs to be zero'd out, so it may take a while depending on the speed of your back-end storage and the size of the new file.<br>
<br>
When you no longer need a virtual disk, use the TUI to delete it (Back-End Storage -> Remove Virtual Disk File). <i>Be sure to end all I/O associated with the virtual disk and remove the SCST device first.</i>

<br>

<h3>What's Next</h3>
Now that you have setup and configured your basic back-end storage, you can continue on to the <a href='41_Hosts_and_Initiators.md'>41_Hosts_and_Initiators</a> wiki document to setup your security groups and initiators. Or if you have advanced back-end storage needs, use the <a href='32_Advanced_Back_End_Storage_Setup.md'>32_Advanced_Back_End_Storage_Setup</a> page for additional information.