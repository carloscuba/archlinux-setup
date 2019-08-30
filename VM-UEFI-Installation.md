# Installing Arch Linux on a Virtual Machine (UEFI Mode)

Most of the guides and tutorials  for installing Arch Linux on a VM out there on the internet use **Legacy Mode** instead of **UEFI**, which is usually preferred when installing the OS on real hardware (not virtualized). 

> _You can find more information about both types on [this article](https://www.partitionwizard.com/partitionmagic/uefi-vs-bios.html)_

On this particular guide, we'll still install Arch Linux on a virtual machine but using **Virtual Box's EFI Mode** to simulate the real hardware (and eventually be able to reproduce the steps when doing it on a real PC).

<br>

_**A wired internet connection is preferred to follow along further steps because we're not only going to install packages from the `.iso` itself but also from the internet**_

<br>

## Download the Arch Linux ISO

As simple as going to the [Arch Linux Download page](https://www.archlinux.org/download/) and downloading the file either via torrent or any or the mirrors available worldwide.

<br>

## Creating the Virtual Machine

* Open Virtual Box (if you don't have it installed yet, please download the installer from [Virtual Box's Official Page](https://www.virtualbox.org) and install it on your computer), click the **New** button and let's create a new virtual machine:

  <img style="display: block; margin-left: auto; margin-right: auto; width: 70%;" src="images/createVM.png">

  <br>

  * Give it a **name** _(I'm naming mine `Arch Linux UEFI` because... yes, you guessed it! We'll be simulating the installation on a real device)_
  * Select the **location** where your Virtual Machine's drive will live in
  * Virtual Box is smart enough to infer that, given the name we're using for the VM, its **Type** and **Version** should be `Linux` and `Arch Linux (64-bit)` correspondingly.

* Once done, click **Continue** and let's give our VM some RAM _(1 or 2 gigs will be more than enough for this kind of installation, but if your host computer has the resources, feel free to bump it up to the number of gigs you desire)_:

  <img style="display: block; margin-left: auto; margin-right: auto; width: 70%;" src="images/vmRam.png">

  <br>

* Click **Continue** and then select **Create a virtual hard disk now**:

  <img style="display: block; margin-left: auto; margin-right: auto; width: 70%;" src="images/vmCreateVirtuakDisk.png">
  
  <br>

* Click **Create** and choose the default **VDI** option:

  <img style="display: block; margin-left: auto; margin-right: auto; width: 70%;" src="images/vmHDType.png">

  <br>

* Click **Continue** and choose the default **Dynamically allocated** option:

  <img style="display: block; margin-left: auto; margin-right: auto; width: 70%;" src="images/vmStorageOnHardDrive.png">

  <br>

* Click **Continue** and set the desired size of the virtual disk we're going to create for the VM _(8 gigs of space is what Virtual Box suggests for most of the Linux installations, but we might want to give it a little more extra space to be able to install some additional packages and don't get into troubles if we're running out of space in the middle of the installation)_:

  <img style="display: block; margin-left: auto; margin-right: auto; width: 70%;" src="images/vmFileAndSize.png">

  <br>

* Click **Create** and we're almost there! Just a few additional things to tweak before getting into the real action.

* Right Click on the brand new VM and click **Settings**: 

  * Navigate to the **System** tab and 
    * On the **Motherboard** sub-tab, **make sure that you check the "Enable EFI (special OSes only)" option**, because remember: _We are simulating the installation on a real device via a VM_
    * On the **Processor** sub-tab, configure the number of processors you'll want the VM to have (in my case, I'll set it up with 2)
  * Navigate to the **Display** tab and 
    * Configure how much Video Memory you'll want the VM to have (in my case, I'll set it up with 128 MB)
    * Change the default **Graphics Controller** from **VMSVGA** to **VBoxVGA** (which will work better if later on we decide to install a Graphical Environment)
    * Check the **Enable 3D Acceleration** option
  * Navigate to the **Storage** tab and
    * Click the **Add Optical Drive** button (next to the **Controller: IDE**)
    * Click **Choose Disk**
    * Find the Arch Linux `.iso` you've just downloaded

* Once everything is setup, we're ready to start our VM!

* Double click on it, or select it and click the **Start** button and there we go... (_it might take some time to boot up, don't be scared_ 🤓)

  <img style="display: block; margin-left: auto; margin-right: auto; width: 70%;" src="images/rootArchLinux.png">

<br>

## Initial Setup

So we have boot into the Live Environment and we're automatically logged in as the **root user** because at this point there's only one user on the system. What do we do now? _We only have the prompt..._

One of our best friends from this point will be the [Arch Linux Official Installation Guide](https://wiki.archlinux.org/index.php/Installation_guide), where you'll be able to find an extensive ~~not that much, actually~~ list of resources that can help you out with some additional _fine tuning_ of your current installation. For the purposes of this guide, we'll only focus in the things that we need.

### Keyboard Layout

If you're using an English keyboard, then you don't need to change anything at all. Arch Linux comes by default with an English Keyboard Layout properly setup. On the other hand, if you have any other type of layout, please take a look at [this section](https://wiki.archlinux.org/index.php/Installation_guide#Set_the_keyboard_layout).

### Verify the boot mode

If you're using **UEFI** (as I am) for the current installation, you might want to check that the iso has been booted properly. To do so, run the following command on the prompt:
```bash
$ ls /sys/firmware/efi/efivars
```

You should be prompted with a similar output:

<img style="display: block; margin-left: auto; margin-right: auto; width: 70%;" src="images/vmBootMode.png">

This will confirm that we are actually booting the system on **UEFI** mode.

> _If the directory does not exist, the system may be booted in **BIOS** or **Legacy Mode**, which is fine, but not for the purposes of this guide_

<br>

### Check that we have working internet connectivity

Just run the following command to make sure that we have internet connection _(again, it is recommended to have the Host Machine connected to the internet via wire because it will simplify a lot the process and we'll have more stable connectivity)_:
```bash
$ ping -c 4 archlinux.org
```

If everything is working as expected, you 'll be prompted with the successful response of the pings:

<img style="display: block; margin-left: auto; margin-right: auto; width: 70%;" src="images/vmInternetConnectivity.png">

> _If you don't have access to a wired connection, you can use the `wifi-menu` command to be able to select a WiFi network and connect to it_

<br>

### Update the System Clock

We need to make sure that the System Clock on the Live Environment is set correctly, so let's run the following command:
```bash
$ timedatectl set-ntp true
```

And then run `timedatectl status` to check the service status:

<img style="display: block; margin-left: auto; margin-right: auto; width: 70%;" src="images/vmSystemClock.png">

<br>

### Partition the Disk

Here's where the fun starts... 🥳

<br>

We need to make sure first that we know exactly how many disks we have currently connected into our machine _(VM, in this case)_. To do so, run the following command:
```bash
$ fdisk -l
```

And check the output:

<img style="display: block; margin-left: auto; margin-right: auto; width: 70%;" src="images/vmCurrentDisks.png">

> _You can also run `lsblk` to get some more information about the physical/virtual drives that are currently connected_

<br>

You might get different results depending on how many (and what type of) hard drives you have currently connected, but in this case we can see our Main Hard Drive (the one we created for the VM) -> **`/dev/sda`** and the Arch Linux ISO (which is still mounted and in use) -> `/dev/loop0`.

> _If you have several hard drives connected, make sure to identify which is the one where you want to continue with the installation_

<br>

At this point, the disk we're going to use (brand new) does not have any partitions, so we need to create them.

We're going to use `cfdisk` (which is better/easier to use than the `fdisk` command suggested on the official installation guide) to properly partition the disk:

* Run the following command
  ```bash
  cfdisk /dev/sda
  ```
  > _Remember that `/dev/sda` corresponds to the identifier of the disk in which we're going to install our Arch distribution. Be extra careful with the command, as it could completely wipe out an entire non-desired disk_

  <br>

* And you'll be prompted with the initial screen of the program:

  <img style="display: block; margin-left: auto; margin-right: auto; width: 70%;" src="images/vmPartitioning1.png">

  > _**Legacy** or **BIOS** mode would use **dos**, but for **UEFI** we need to select **gpt**_

  <br>

* With `gpt` selected, hit the `enter` key:

  <img style="display: block; margin-left: auto; margin-right: auto; width: 70%;" src="images/vmPartitioning2.png">

* Let's create our **UEFI** partition first:

  * From the total available space (20G at this point), hit the `enter` key to create a new partition and set the size of it to **512M**

    <img style="display: block; margin-left: auto; margin-right: auto; width: 70%;" src="images/vmPartitioningUEFI1.png">

    <br>
  
  * Hit the `enter` key again and you'll see how the new partition is created (with the previously set 512M):

    <img style="display: block; margin-left: auto; margin-right: auto; width: 70%;" src="images/vmPartitioningUEFI2.png">

    <br>

* Now let's jump into our **SWAP** partition:

  * Move down (with the `down arrow`) to select **Free space** again, make sure that the **[ New ]** option is selected at the bottom and hit `enter` again. This time, for our SWAP partition, **we'll allocate 1G of space** (_in a real machine, you might want to give it up some more extra space, depending of your needs_):

    <img style="display: block; margin-left: auto; margin-right: auto; width: 70%;" src="images/vmPartitioningSWAP1.png">

    <br>

  * With the recently created partition still selected, move to the right (with the `right arrow`) to **[ Type ]**, hit `enter` and select **Linux swap**:

    <img style="display: block; margin-left: auto; margin-right: auto; width: 70%;" src="images/vmPartitioningSWAP2.png">

    <br>

  * Hit `enter` again and we'll return to the program's main screen and we should be able to see the two partitions we have just created (512M for our UEFI and 1G for our SWAP):

    <img style="display: block; margin-left: auto; margin-right: auto; width: 70%;" src="images/vmPartitioningSWAP3.png">

    <br>

* Once done, the only thing left is to create our **Main Partition** (where our filesystem will live):
  
  * Move down again to select **Free space**, make sure the **[ New ]** option is selected at the bottom, hit `enter`, and allocate the remaining available space for the current partition:

    <img style="display: block; margin-left: auto; margin-right: auto; width: 70%;" src="images/vmPartitioningMAIN1.png">

    <br>

  * Hit `enter` again (to actually set the available space for the new partition) and that's it!

    <img style="display: block; margin-left: auto; margin-right: auto; width: 70%;" src="images/vmPartitioningMAIN2.png">

    <br>

  * We now have our **UEFI**, **SWAP** and **MAIN** partitions created... but they are not written into the actual disk yet! 🤯

* To write what we have configured into the actual disk:

  * We need to move right (with the `right arrow`) to the **[ Write ]** option 
  * Once there, hit `enter`
  * Type `yes` (the entire word)
  * And hit `enter` again. You'll see a message confirming the alteration of the partition table:

    <img style="display: block; margin-left: auto; margin-right: auto; width: 70%;" src="images/vmPartitioningWRITTEN.png">

    <br>

  * Move left to **[ QUIT ]** and hit `enter` again. This time we're done with the partitions!
  * _But we still need to properly format them..._

<br>

### Formatting the Partitions

Before jumping into the actual formatting of our partitions, let's double check that the partition table looks as expected.

Run `fdisk -l` once more and check the output:

<img style="display: block; margin-left: auto; margin-right: auto; width: 70%;" src="images/vmFormatting1.png">

From the above, we can say that

* `/dev/sda1` is our **UEFI** partition with 512M of space
* `/dev/sda2` is our **SWAP** partition with 1G of space
* `/dev/sda3` is our **MAIN** partition with 18.5G of space

<br>

To properly format them, let's run some other commands:

* Run the following command to properly format the **UEFI** partition (`/dev/sda1` with 512M of space)
  ```bash
  $ mkfs.fat -F32 /dev/sda1
  ```

* Now run the following command to properly format the **HOME** partition (`/dev/sda3` with 18.5G of space)
  ```bash
  $ mkfs.fat -F32 /dev/sda3
  ```

  <img style="display: block; margin-left: auto; margin-right: auto; width: 70%;" src="images/vmFormatting2.png">

  > _**Make sure that you don't mess with `/dev/sda2` just yet**_

  <br>

* And last but not least, run the following commands to properly format the **SWAP** partition (`/dev/sda2` with 1G of space)
  ```bash
  $ mkswap /dev/sda2
  $ swapon /dev/sda2
  ```

  <img style="display: block; margin-left: auto; margin-right: auto; width: 70%;" src="images/vmFormatting3.png">

<br>

Well done! We have properly partitioned and formatted our disk...

<br>

### Mounting the file system

* Run the following command to mount the file system on the root partition to `/mnt`:
  ```bash
  $ mount /dev/sda3 /mnt
  ```

* And then run `lsblk` to make sure it was properly mounted:

  <img style="display: block; margin-left: auto; margin-right: auto; width: 70%;" src="images/vmMountingRoot.png">

  <br>

* We'll come back later to setup our `efi` partition as required. For now, let's move on to the next steps.

<br>

## Installation Process

So far, we have the required initial setup to start installing some packaged and make our Arch Linux distribution alive!

<br>

### Optimizing our mirrors

This step is optional, but I've found that it might be really useful to speed-up the installation process of some packages.

As previously mentioned, there's a huge list of mirrors all around the globe from where we can install the packages we want, but it is worth to mention that that further they are from our physical location, the more they will take to provide us with the files we need (_we don't want to download a package that is hosted on some server in Russia if we are in Peru, right?_ 🤨).

<br>

For this to be possible, we need to edit `/etc/pacman.d/mirrorlist`. Quite easy, right? But... do you know if there is any available text editor already installed on the system? 🥺

Fortunately, we do have our friends `vim` and `nano` already installed and ready to be used for these kind of purposes! 💪🏻

Run the following command (using your preferred text editor) to get into the mirrors list and optimize it a little bit:
```bash
$ nano /etc/pacman.d/mirrorlist
```

<img style="display: block; margin-left: auto; margin-right: auto; width: 70%;" src="images/vmOptimizeMirrors.png">

Don't freak out... just yet. What we need to do is _"simply find the mirrors that are far away from our physical location and remove them from the list"_

I won't get into much details about this, but the idea is to keep at least 6 or 7 mirrors (the closest to us) and remove everything else.

<br>

> _Don't forget `Ctrl + K` to **delete lines** (you're going to need it), `Ctrl + O` to **save** and `Ctrl + X` to **exit**_

<br>

### Install the base packages

* Run the following command to install the `base` and `base-devel` packages from Arch Linux:
  ```bash
  $ pacstrap -i /mnt base base-devel
  ```

* Once ran, select the default prompted options (by hitting the `enter` key) and, at the end, hit the `y` key (to confirm the installation):

<img style="display: block; margin-left: auto; margin-right: auto; width: 70%;" src="images/vmBaseInstall1.png">
<img style="display: block; margin-left: auto; margin-right: auto; width: 70%;" src="images/vmBaseInstall2.png">

<br>

That's going to take a while, so make yourself comfortable and, if you want, take a look at the packages being installed 🤓

<br>

## Configuring the system

To generate the File System Table

```bash
$ genfstab -U /mnt >> /mnt/etc/fstab
```

To login as root into the new system

```bash
$ arch-chroot /mnt
```

<img style="display: block; margin-left: auto; margin-right: auto; width: 70%;" src="images/vmChroot.png">

> _Note how the prompt has changed... we're no longer root on the Live Environment but **root on our brand new system!**_

<br>

And, as we are already on our system, we can run our dear pal `ls` to list the files on the system:

<img style="display: block; margin-left: auto; margin-right: auto; width: 70%;" src="images/vmFirstLS.png">