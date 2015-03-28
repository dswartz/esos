### SCST I/O Modes ###
ESOS is based on the SCST project, and it really is the core of ESOS. SCST provides a number of different features and I/O modes (pass-through, FILEIO, BLOCKIO, etc.). At the time of writing this document, ESOS supports (9) SCST device I/O modes.

Here is a list and description of each supported I/O mode:
  * **Pass-Through SCSI Disk (dev\_disk)**: SCSI commands coming from initiators on your SAN are passed to the local SCSI disks on the ESOS storage server as is, without any modifications. One-to-many is supported in this mode in which several initiators can safely connect to a single device.
  * **Pass-Through SCSI Disk, Performance (dev\_disk\_perf)**: Similar to the mode described above except no data is transferred to/from the underlying SCSI device; used for measuring performance.
  * **Pass-Through SCSI Changer (dev\_changer)**: SCSI commands coming from initiators on your SAN are passed to the local SCSI changer/robot on the ESOS storage server as is, without any modifications. One-to-many is supported in this mode in which several initiators can safely connect to a single device.
  * **Pass-Through SCSI Tape (dev\_tape)**: SCSI commands coming from initiators on your SAN are passed to the local SCSI tapes on the ESOS storage server as is, without any modifications. One-to-many is supported in this mode in which several initiators can safely connect to a single device.
  * **Pass-Through SCSI Tape, Performance (dev\_tape\_perf)**: Similar to the mode described above except no data is transferred to/from the underlying SCSI tape; used for measuring performance.
  * **VDISK CDROM (vcdrom)**: This handler allows for emulation of a virtual CDROM device which uses an ISO file as the back-end -- easily share an ISO image on your SAN.
  * **VDISK BLOCKIO (vdisk\_blockio)**: This mode performs direct block I/O with a block device, bypassing the Linux page cache for all operations; works ideally with high-end storage HBAs and for applications that either do not need caching between the application and disk or need the large block throughput.
  * **VDISK FILEIO (vdisk\_fileio)**: Use files on file systems that are on your back-end storage devices (RAID volumes, etc.) and benefits from the Linux page cache. The back-end files act as "virtual disks" which can be manipulated offline like normal files (moved, copied, etc.). Sparse files are supported, which can act as a "thin provisioned" storage volume.
  * **VDISK NULLIO (vdisk\_nullio)**: The NULLIO mode is used to create virtual devices. In this mode, no real I/O is done, but success is returned to the initiators. Its intended to be used for performance measurements similar to the dev\_disk\_perf mode described above.

Your specific use/needs will determine the appropriate SCST I/O mode to select and you may use a combination of these for different devices; please see the SCST documentation and SCST mailing list archives for additional information and other use scenarios regarding the SCST I/O modes.

<br>

<h3>Adding/Removing Devices</h3>
To add a new device in ESOS, choose the Add Device function/dialog (Devices -> Add Device). You will first be prompted to choose a SCST device handler (I/O modes described above).<br>
<br>
<ul><li>For the dev_disk, dev_disk_perf, dev_changer, dev_tape, and dev_tape_perf handlers, the next selection choice will be the SCSI disk/changer/tape, and that is the only requirement for setting up one of these devices.<br>
</li><li>The vcdrom handler requires you to select an ISO image file with no additional user input required.<br>
</li><li>For the vdisk_blockio handler you will be prompted to choose a back-end storage block device. This is typically a SCSI disk block device (/dev/sdX) or may be a advanced block device such as DRBD, Linux md RAID, LVM2, etc. After you have selected the block device, you'll be prompted for a device name and a few other parameters which should be self explanatory. See the SCST documentation (README) for a thorough explanation of these SCST device options.<br>
</li><li>For the vdisk_fileio handler you will be prompted to choose a "virtual disk" file. This is a file on a file system that you created and setup previously (<a href='31_Basic_Back_End_Storage_Setup.md'>31_Basic_Back_End_Storage_Setup</a>). After selecting the desired virtual disk file, you'll need to configure a few SCST device parameters, similar to the vdisk_blockio handler options. See the SCST README for more information on these configuration options.<br>
</li><li>With the vdisk_nullio handler you need to only choose a device name and set a few configuration options since no real I/O is performed on a back-end storage device with this I/O mode (handler).</li></ul>

When you no longer need a SCST device, it can easily be removed in the TUI: Devices -> Delete Device<br>
<br>
Be sure there is no active I/O on the device before deleting it! Although when deleting a SCST device, no data is actually removed/deleted/altered on the back-end storage device, you should still be sure its not in use. Minimal human-error checking is performed with this function (eg, not checked for mappings, etc.).<br>
<br>
<br>

<h3>Mapping & Unmapping Devices</h3>
After you have added a SCST device, you then need to map it to a security/host group and choose a logical unit number. This will make the device visible to your initiator(s).<br>
<br>
In the TUI, choose the <code>Map to Group</code> function (Devices -> Map to Group) to add a device-LUN mapping. You will first be prompted to choose a SCST device. After selecting the device, you will need to choose a target -- choose the target that you add the security group to. Next you will be prompted to choose the host/security group, and finally you'll be presented with the device-LUN mapping dialog. Use the up/down arrow keys to scroll to the desired Logical Unit Number (LUN) for the selected device. <i>If your initiator(s) do not have "sparse LUN" support, be sure that you have a LUN 0 presented to the initiator(s).</i> You can also optionally choose to set this device-LUN mapping as read-only; hit OK to complete the device-LUN mapping.<br>
<br>
You can also remove device-LUN mappings in the TUI (Devices -> Unmap from Group). Removing a device-LUN mapping does not delete the device, it only makes the device not visible to the initiators in the selected host group.<br>
<br>
<br>

<h3>Getting Device/LUN Information</h3>
The TUI can be used to gather some more or less basic information on the LUN layout and SCST devices on your ESOS host. Use the CLI for additional information (eg, /etc/scst.conf file).<br>
<br>
Use the LUN Layout dialog (Devices -> LUN Layout) for a nice overview of all of the targets, groups, initiators, LUNs and their relationship on your ESOS storage server.<br>
<br>
Use the Device Information dialog (Devices -> Device Information) for detailed information and attributes of your SCST devices.<br>
<br>
<br>

<h3>Modifying Device Attributes</h3>
While modifying or updating SCST device attributes is not supported directly using the TUI, you can simply unmap and remove the SCST device, and then re-add it if you are changing attributes.<br>
<br>
Or, it may be more convenient to use the CLI (shell) to modify SCST device parameters without possibly disrupting I/O (depending on the change). To do this, you simply update the SCST configuration file and apply the changes using the <code>scstadmin</code> tool. When in the TUI, exit to the shell (Interface -> Exit to Shell). Update the SCST configuration file (<code>/etc/scst.conf</code>) as desired. Then you can apply the changes:<br>
<pre><code>scstadmin -config /etc/scst.conf<br>
</code></pre>
You may be warned of errors or possible I/O disruption for the requested changes. See the SCST project's documentation for more information on using the <code>scstadmin</code> command.<br>
<br>
<br>

<h3>What's Next</h3>
After you've setup your SCST devices and mapped them as LUNs to security groups, you're ready to enable the target(s) which will make the storage targets visible to remote initiators. Continue with the <a href='61_Target_Configuration.md'>61_Target_Configuration</a> document.