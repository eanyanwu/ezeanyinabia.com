---
layout: post
title: Computer Setup
excerpt_separator: <!--more-->
---

Since I switched to trying out Gnu/Linux and \*BSD operating systems, I have spent a lot of time tuning my computer setup. I know I'm not going to remember all the little tweaks, so I'm keeping track of them here.

<!--more-->

# Contents

- [Password Management](#password-management)
    - [Basics](#basics)
    - [Automatic username and password completion](#automatic-username-and-password-completion)
    - [Syncing](#syncing)
- [Mounting external devices](#mounting-external-devices)
   - [USB](#usb)


# Password Management

Having strong and unique passwords for all my accounts AND remembering them is hard. What a time to be alive. Thankfully, we have password managers.

I'm moving to a "simpler" approach with [pass](https://www.passwordstore.org). By default everything is stored in gpg-encrypted files on the local machine. Syncing and other fancy features need to be manually setup. This is fine by me because I'm all about wasting countless hours learning how to do things that have already been solved by others. 


## Basics 
- The first step is obviously to download `pass`.
- Using whatever PGP program you have, create a key-pair that the `pass` program will use.
    - For me this meant: `gpg2 --generate-key`
- Now that you have a PGP key-pair, initialize the password store with that keypair: `pass init <KEY_ID>`
- Insert passwords into the store using the multi-line option, so we can save usernames:
    - `pass insert -m website.com`
    - Use the first line for the password an the second for the username like so: 
    ```
    {password}
    username:{username}
    ```

## Automatic username and password completion
Password managers are very convenient because of the browser extentions they usually offer. `pass` doesn't come with anything of the sort.   
Thankfully, creating a system to do the same thing is not that hard. The creator of `pass` has his version of it called [passmenu](https://git.zx2c4.com/password-store/tree/contrib/dmenu), which is where I got the inspiration. 

[This](https://github.com/eanyanwu/.dotfiles/blob/master/scripts/passmenu.sh) is my version of the script. Don't you worry, it's well commented :)

The last step to get this working is to get a graphical version of the _pinentry_ program. From what I understand, _pinentry_ is the program that runs whenever a password-protected gpg key is accessed. The default version usually runs in the terminal. This won't do for our situation because the script won't be running in a terminal.  
So get a version of pinentry that will pop open a window instead of drawing to the terminal. For OpenBSD, I was able to find a gtk version that I installed using `pkg_add`. Once it was intalled, gpg somehow automatically defaulted to using the graphical version, so no further configuration was required.


## Syncing

To sync the password store across multiple devices, you need to setup an ssh server where the password store will live as a git repository. 
- [Instructions on setting up the git repository on the remote machine](https://git-scm.com/book/en/v2/Git-on-the-Server-Setting-Up-the-Server)
- [Instructions for connecting to the remote machine from the passforios app](https://github.com/mssun/passforios/wiki#quick-start-guide-for-pass-for-ios)


# Mounting external devices


## USB 

To figure out the name that the operating system is using to refer to the device, you can check the system log through `dmesg`. 
If the device has already been formatted in a way the machine understands, you should be able to mount it like so:
- `mount /dev/<DEVICE_NAME><PARTITION_NAME> <MOUNT_POINT>`

You can figure out the partition name by checking `fdisk <DEVICE_NAME>` 


If the device has not yet been formatted, you can format it using the following steps (WARNING: this will wipe the usb):
- Recreate the partitions using `fdisk`
- Create a new filesystem on the usb using a variant of `newfs`
