# archbtw
if you have an nvme SSD with UEFI and no ethernet cable and want to encrypt your drive and install a bunch of other stuff, YOU ARE IN THE RIGHT PLACE.

# First Steps 

Burn the iso image to a usb drive, boot up press F12 repeatedly to start the live installer

run this:

```
iwctl
station wlan0 connect "your_wifi_name" 
```
on prompt enter the passphrase, you can always type help to list your devices and available networks.
Verify your connectivity:
```
ip link
ping archlinux.org
```

## Date and time set up
```
ln -s /usr/share/zoneinfo/Region/City /etc/localtime
hwclock --systohc --utc
```

## Partition time

```
fdisk /dev/nvme0n1
```
enter d and keep pressing enter you have deleted all the existing partitions.

now press g to make it a gpt disk.
Now we are in fdisk, you can press m for help in the program, But if you do exactly as below everything should be alright

Hit d and then enter untill there are no more partitions

Hit o, Then press enter, This will make it a mbr disk

press n and then enter to make our first partition, hit p and enter to make it primary, give it partition number 2 and press enter then just press enter again to put it in the beginning of the disk then type +2G to give it 2GB for the boot partition

press n again to create our swap partition, hit p and enter to make it primary, give it partition number 3 and press enter then just press enter again to put it next to the previous partition then type +8G to give it 8GB swap, If you have less ram I would recommend making it equal to your ram, So +4G if you have 4GB Ram.

press n again to create our final partition, hit p and enter to make it primary, give it partition number 1 and press enter then just press enter again to put it next to the previous partition then press enter again to give it the remaining space on the disk

Now hit t and press enter select 1 and press enter type 83 and hit enter this will make the first partition a Linux filesystem.

Now hit t and press enter select 2 and press enter type a and press enter this will make the second partition the boot partition

Now hit t and press enter select 3 and press enter type 82 and press enter this will make the third partition a swap partition

Finally hit w and press enter to write all changes to disk

## Format the disks
```
mkfs.ext4 /dev/nvme0n1p1
mkfs.fat -F32 /dev/nvme0n1p2
mkswap /dev/nvme0n1p3
swapon /dev/nvme0n1p3
```

### Encrypt your disks (Optional not recommended)
```
cryptsetup -y -v luksFormat /dev/nvme0n1p1
cryptsetup open /dev/nvme0n1p1 cryptroot
mkfs.ext4 /dev/mapper/cryptroot
```
You may encrypt the swap and the boot as well, but you need to touch some grass if you are that paranoid.

Now the disk /dev/nvme0n1p1 is refferd to as /dev/mapper/cryptroot

## Mount your the paritions:

```
mount /dev/mapper/cryptroot /mnt 
mkdir /mnt/boot
mount /dev/nvme0n1p2 /mnt/boot
```
Mount with no encryption trust me bro you don't need encryption is unstable

```
mount /dev/nvme0n1p1 /mnt
mkdir /mnt/boot
mount /dev/nvme0n1p2 /mnt/boot
```

# Install base packages