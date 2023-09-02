---
title: Unable to find a supported device
author: admin
type: post
date: 2010-05-07T04:11:36+00:00
url: /archives/3567
IM_data:
 - 'a:4:{s:76:"http://www.vm-help.com/esx/esx3i/ESXi_install_to_IDE_drive/Install_Error.jpg";s:73:"http://blog.haohtml.com/wp-content/uploads/2011/03/bc11_Install_Error.jpg";s:83:"http://www.vm-help.com/esx/esx3i/ESXi_install_to_IDE_drive/Install_start_screen.jpg";s:80:"http://blog.haohtml.com/wp-content/uploads/2011/03/92e0_Install_start_screen.jpg";s:80:"http://www.vm-help.com/esx/esx3i/ESXi_install_to_IDE_drive/Install_IDE_drive.jpg";s:77:"http://blog.haohtml.com/wp-content/uploads/2011/03/cdf9_Install_IDE_drive.jpg";s:79:"http://www.vm-help.com/esx/esx3i/ESXi_install_to_IDE_drive/Install_Complete.jpg";s:76:"http://blog.haohtml.com/wp-content/uploads/2011/03/edd3_Install_Complete.jpg";}'
IM_contentdowned:
 - 1
categories:
 - 服务器
tags:
 - vmware

---

**Install ESXi 3.5 to an IDE drive**

By default, if the ESXi install can not find a supported device to install to, then the installer will quit with the error message: “Unable to find a supported device to write the VMware ESX Server 3i 3.5.0 image to.” If it is the case that the IDE drive in your host is recognized by ESXi, then you will be able to modify the install script TargetFilter.py to recognize your IDE device as a supported install device. You can find the list of devices that ESXi can recognize [here](http://www.vm-help.com/esx/esx3i/Hardware_support.php).

The install process for this can be found below, and a video of the process can be found at the bottom of this page. This demo was created using a VMware Workstation VM which only had a 4 GB IDE drive.

![](http://www.vm-help.com/esx/esx3i/ESXi_install_to_IDE_drive/Install_Error.jpg)

**Install Process**

1) If you have encountered the error shown above, you can press ALT-F1 to access the console of the ESXi install. You’ll be prompted for a login and you can use ‘root’. The password for root with be blank. As mentioned above, this process assumes you have an IDE drive that ESXi can recognize. You can use the command **lspci** to show the list of devices that ESXi can recognize and then compare that with this list of [devices](http://www.vm-help.com/esx/esx3i/Hardware_support.php). Also if you run **fdisk -l**, you should see your IDE drive listed.

2) After you have console access you will enter the command **vi /usr/lib/vmware/installer/Core/TargetFilter.py** (note that the path and filename are case-sensitive).

3) Scroll down in the document until you find the section “def IDEFilter(lun)”. You will be changing the text:

**return interface.GetInterfaceType() == ScsiInterface.SCSI\_IFACE\_TYPE_****IDE

to
return interface.GetInterfaceType() == ScsiInterface.SCSI\_IFACE\_TYPE_ISCSI**
 ****
If you have not used vi before, move the cursor to the end of “TYPE_IDE” and the press the Insert key. The press backspace to delete IDE and type in ISCSI. Then press the ESC key, type in the command **:wq** and press Enter to save the file and exit.

4) You will now be back at the console. If you had stopped the installer at the screen show below, you can press ALT-F2 to return to the screen and press Enter to start the install, but it will still generate the error shown in the image above. You will need to press ALT-F1 and then type in **install** and press enter.

5) When you run the **install** command, it is important to note that the installer will switch you back to the ALT-F2 (DCUI) screen. Press ALT-F1 to return to the console again. You will see the below screen again with the prompt to press Enter to install. Do so and the install will proceed.

![](http://www.vm-help.com/esx/esx3i/ESXi_install_to_IDE_drive/Install_start_screen.jpg)

6) Press F11 on the next screen and you should then see your IDE drive as show below. Press Enter to continue and after a few minutes the install should complete and you will be prompted to reboot.

![](http://www.vm-help.com/esx/esx3i/ESXi_install_to_IDE_drive/Install_IDE_drive.jpg)

7) After the reboot, you will be able to connect with the VI client and see that ESXi has installed to and created a VMFS partition on your IDE drive.

![](http://www.vm-help.com/esx/esx3i/ESXi_install_to_IDE_drive/Install_Complete.jpg)