# Raspberry PI Cluster

Notes on creating a k3OS based raspberry pi cluster.

## Standard raspberry pi imaging (collecting MAC addresses for later reference)

List and unmount the target sdcard block devices:

    # run before and after inserting the sdcard, noting the new block device listed:
    lsblk -p

    # umount the new block device mounts in prep for writing an image, e.g. for existing rpi imaged sdcard:
    umount /media/t/boot
    umount /media/t/rootfs  

Writing the image to sd card (pre-download the latest raspberry pi image of choice):

    sudo dd if=~/Downloads/2022-04-04-raspios-bullseye-armhf-lite.img of=/dev/sdb bs=4M conv=fsync

Configure ssh on first boot:

    touch /media/$USER/boot/ssh

Configure a user (where the username is "pi" and password is "raspberry"):

    echo 'raspberry' \
      | openssl passwd -6 -stdin \
      | xargs -I {} echo "pi:{}" \
      > /media/t/boot/userconf.txt

Optional for wifi configuration, create a wifi configuration file:

    touch /media/$USER/boot/wpa_supplicant.conf

    # place the following content in the `wpa_supplicant.conf` file to configure home router and phone hotspot:

    ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
    update_config=1
    country=AU
    
    network={
            priority=1
            id_str="router"
            ssid="<<ROUTER SSID>>"
            psk="<<ROUTER PASSWORD>>"
            scan_ssid=1
    }
    
    network={
            priority=2
            id_str="phone"
            ssid="<<PHONE SSID>>"
            psk="<<PHONE PASSWORD>>"
            scan_ssid=1
    }
    
Power up the pi and wait a few minutes to get the ip address and the MAC address for it with this helper command (using the well known manufacturer MACs):

    sudo nmap -sn 192.168.1.0/24 | egrep "(B8:27:EB|E4:5F:01)"

## Prep for automated k3OS imaging

Roughly following this guide: https://www.chriswoolum.dev/k3s-cluster-on-raspberry-pi

Utilising a fork of this repo (forked to add improvements): https://github.com/sgielen/picl-k3os-image-generator

### Subtree management

Following git subtree notes here: https://www.atlassian.com/git/tutorials/git-subtree

    # add remote for easier tracking
    git remote add -f picl-k3os-image-generator git@github.com:tonymcneil/picl-k3os-image-generator.git
    
    # create a subtree subfolder structure within this repo:
    git subtree add --prefix subtrees/picl-k3os-image-generator picl-k3os-image-generator master --squash

    # updating this repos subtree from the tracked remote
    git fetch picl-k3os-image-generator master
    git subtree pull --prefix subtrees/picl-k3os-image-generator picl-k3os-image-generator master --squash

    # pushing changes to the subtree fork (for eventual PR against the source repo):
    git subtree push --prefix=subtrees/picl-k3os-image-generator picl-k3os-image-generator master

