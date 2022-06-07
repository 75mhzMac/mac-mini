# Mac mini G4 - 3x OS Install with OpenBSD

## Requirements
- Mac mini G4 with 1.25, 1.33, 1.42, 1.5 Ghz Processor
- Mac mini OS 9.2 (V9) ISO burned to disk (Macintosh Garden) 
  - https://macintoshgarden.org/apps/mac-mini-g4-macos-92-v9
- Mac OS X Install DVD
- OpenBSD macppc 7.1 ISO written to USB flash drive
  - https://cdn.openbsd.org/pub/OpenBSD/snapshots/macppc/install71.iso
- Ethernet internet connection

### Speakers or Headphones Recommended 
The internal speaker does not function under Mac OS 9.


## Preparing Your Hard Disk Using OS X
For the sake of simplifying the partition map and avoiding having to repair the disk drivers once OpenBSD is installed, we will use Mac OS X to format your hard disk.

1. Start the computer using the Mac OS X Install DVD
2. Choose your language.
3. Open `Disk Utility` from the `Utilities` menu.
4. Select your internal hard disk.
5. Click on the `Partition` tab.
6. Under `Volume Scheme`, choose `3 Partitions`.
7. Make sure the box `Install Mac OS 9 Disk Drivers` is unchecked. 
8. The first partition is for Mac OS 9. `Mac OS Extended` format. 
9. The second is for Mac OS X. Use `Mac OS Extended (Journaled)` format.
10. Select the partition labeled `untitled 3`. Format will be `Free Space`.
11. Click on the `Partition` button.
12. Choose `Partition` to confirm.
13. Quit Disk Utility

## Switching Disks 
This step is not necessary if you are using an external optical drive.

- Restart your computer.
- Enter the Open Firmware prompt. `Command`+`Option`+`O`+`F`
```
eject cd
```
- Now you can insert your Mac OS 9 installer DVD.

Enter the following into the prompt and press `Return`
```
mac-boot
```
If the installer disk isn't found during startup, reboot the computer and enter the Startup Manager using the `Option` key.

## Install Mac OS 9
 
1. Open the Apple Software Restore application from the Mac mini OS 9 CD located on your desktop.
2. Click on the `Switch Disk` button until the your Mac OS 9 partition is chosen, if it isn't already.
3. Click `Restore` to begin. Click `OK` to confirm.
4. The disk will be renamed to "Macintosh HD" upon completion.
5. Restart the computer, holding down the `Option` key to access the Startup Manager.
6. Boot into your new installation.

## Skip Setup

When greeted with the Mac OS Setup, you can quit the installer by pressing `Command`+`Q`.

## Copying Boot File for OpenBSD
1. Insert your OpenBSD USB installer disk. Leave this disk attached until the OpenBSD installation is complete.
2. At the root of the disk is a file named `ofwboot`. 
3. Copy this file to the root of your Mac OS 9 partition.
4. Eject the Mac mini OS 9 CD and insert your Mac OS Install DVD.
5. Restart the computer.
6. Choose your Mac OS X Install DVD from the Startup Manager using `Option` during boot.


## Install Mac OS X

#### Note For Installing Sorbet Leopard Revision 1.5 from External Hard Disk
I have noticed that the partition is missing from the Startup Manager after installing using Carbon Copy Cloner.
Repairing the disk using Disk Utility and choosing it as a startup disk in System Preferences remedied the issue.

1. Begin by starting up from your OS X installation disc. Choose your language.
2. Continue the installation, choosing your Mac OS X disk as the destination.
3. Choose `Custom Install` to customize.
4. Proceed with installation.

### Important
**The computer will restart after installation, so be ready to enter the Open Firmware prompt to install OpenBSD.**

If you end up not entering the Open Firmware prompt in time, your computer will boot into your new Mac OS X installation. Simply setup your Mac OS X system and restart.


## Install OpenBSD 7.1

Type the following into the Open Firmware prompt:

`Command`+`Option`+`O`+`F`
```
dev usb0
ls
```

- You will now see a list of available devices. What you are looking for is `disk@`. 
- If you have found your disk in the list, take note of the number your disk is located on.

If no disks are found, examine the next USB port with the following command.
```
dev usb1
ls
```

Let's say the USB disk is located at usb1 on `disk@1`.

``` 
boot usb1/disk@1:,ofwboot 7.1/macppc/bsd.rd
```
- Ensure your Mac mini is now connected to the internet via Ethernet. 
- Eventually you will be greeted with the OpenBSD installer. 
- Press the `i` key, then `return` to begin the installation.

```
Enter your hostname. [Mac mini]
Which network interface do you wish to configure? [gem0]
IPv4 address for gem0? [autoconf]
IPv6 address for gem0? [none]
Which network interface do you wish to configure? [done]

Enter root account password.
Start ssh(8) by default [yes]
Do you want the X Window System to be started by xenodm(1)? [no]
Change the default console to zstty0? [no]
Setup a user? (Enter a username)
Password? 
Allow root ssh login? [no]
Timezone? (Press ? if not correct).

Which disk is the root disk? [wd0]
Use HFS or MBR partition table? [hfs]
Modify or abort? [Modify]
```

### Modifying The Apple Partition Map

```
Command: p   (Note where your unformatted partition space is located.)
Command: t
Partition number: 6 
New type of partition: OpenBSD
Command: n
Partition number: 6
New name of partition: OpenBSD
Command: p
Command: w
Writing the map destroys what was there before. Is that OK? y
Command: q


Use (A)uto layout, (E)dit auto layout, or create (C)ustom layout? [a]

Which disk do you want to initialize? [done]
```

### Bring in the OpenBSD
It is OK to use whatever server is automatically suggested.
```
Location of sets? [http]
Use proxy URL? [none]
HTTP Server? [cdn.openbsd.org]
Server directory? [pub/OpenBSD/snapshots/macppc/]

Set name(s)? [done]
```

OpenBSD will now proceed to install onto your hard drive.

```
Location of sets? [done]
Exit to shell? [reboot]
```

## Setting a default OS
You can set OpenBSD to be the default OS at startup, using the `Option` key to choose between Mac OS versions.

`Command`+`Option`+`O`+`F` 
```
setenv boot-device hd:,ofwboot \bsd
reset-all
```

**The alternative to this is choosing an OS through the Startup Disk control panel.**

Boot OpenBSD each time by using the following Open Firmware command:
```
boot hd:,ofwboot \bsd
```

This prompt can also be accessed from the Startup Manager by pressing `Control`+`Z`.


## OpenBSD Initial Setup
1. Boot into your OpenBSD installation.
2. Login.

#### Tip
If your keyboard cannot type a tilde, then replace `~/` with `/home/yourownusername/`.
The `~/` is just a shortcut to your home folder.

## Install Nano & IceWM
Type the following into the terminal:
```
su
pkg_add nano icewm
exit
nano ~/.xsession
```
Next, type this into the text editor.
```
exec icewm
```
- `Control`+`o`
- `Return` to save changes
- `Control`+`x` to exit


## Enable Xenodm
This allows the window manager to run at startup.
```
rcctl enable xenodm
```


## Suggested Setup (Optional)
```
usermod -G operator yourusername
usermod -G staff yourusername
usermod -G wheel yourusername
nano /etc/doas.conf
```
Enter the following into the text editor:
```
permit persist keepenv :wheel
```
- `Control`+`o`
- `Return` to save changes
- `Control`+`x` to exit

You can now use doas instead of having to enter/exit su, similar to sudo in Linux.


## Command to Restart The Computer
```
shutdown -r now
```

## Fixing Screen Issue with Terminal Command
**If you see the XConsole window in addition to the login window, then skip this step.**

- This time when you boot into OpenBSD, you will be greeted with a GUI login screen.

 To put it simply, the display is not set up correctly. Go ahead and log in.

- Press `Control`+`Alt`+`T` to open a terminal window.
- You will ***NOT*** see the terminal window on your screen, but it is open and active.

Enter the following terminal command to temporarily fix this:
```
xrandr --output S-video --off
```

## Permanent Screen Fix
Use this terminal command:
```
doas nano /etc/X11/xenodm/Xsetup_0
```
Insert the `xrandr` command you used earlier into the end of the file.
```
xrandr --output S-video --off
```
- `Control`+`o`
- `Return` to save changes
- `Control`+`x` to exit

## Command to Power Off The Computer
```
shutdown -p now
```
# Setup Completed

Take a well deserved break.


## Extras

### Hide XConsole
This will hide XConsole at startup for the login screen and window manager.
```
doas nano /etc/X11/xenodm/Xsetup_0
```
Comment out the line pertaining to xconsole by adding # to the beginning of the line.


### Change The LoginScreen/Desktop Background Color
- Doesn't apply to MLVWM session. See https://github.com/morgant/mlvwmrc for more info on configuration.
```
doas nano /etc/X11/xenodm/Xsetup_0
```
Add this line to the end of the file.
```
xsetroot -solid "#79a0d9"
```
Change the hex color to your liking.


### Fun Packages To Install
- audacious
- burgerspace
- htop
- neofetch
- tigervnc
- dillo
- netsurf
- lynx


### Check The Weather
```
curl wttr.in/YourZipCode
```

### Raspberry Pi owners
Use tigervnc to connect to your pi to handle web browsing with RealVNC server enabled.


