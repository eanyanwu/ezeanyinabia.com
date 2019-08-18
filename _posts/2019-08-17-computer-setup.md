---
layout: post
title: Computer Setup
excerpt_separator: <!--more-->
---

Since I switched to trying out Gnu/Linux and \*BSD operating systems, I have spent a lot of time tuning my computer setup. I know I'm not going to remember all the little tweaks, so I'm keeping track of them here.

<!--more-->

# Contents

1. [Password Management](#password-management)
    1. [Basics](#basics)
    1. [Automatic username and password completion](#automatic-username-and-password-completion)


# Password Management

I hope we can all agree that having strong and unique passwords for all the accounts we use AND remembering them all is hard. What a time to be alive. Thankfully, we have password managers.

I have been using 1Password for a while now. No complaints. However, with all these companies getting hacked, I'm realizing 1Password is a pretty big attack target given their number of users. "But it won't matter since the vaults are encrypted!" says a naysayer. Fair point, but (1) how do _really_ know that 1Password has its encryption properly implemented? and (2) give an attacker enough time with an collection of offline vaults and they will eventually get into a few of them since they are only protected by one password.

For these reasons, I'm moving to a simpler approach with [pass](https://www.passwordstore.org). By default everything is stored in gpg-encrypted files on the local compter. Any syncing or other fancy features need to be manually setup. This is fine by me because I'm all about wasting countless hours learning how to do things that have already been solved by others .\_. 
Manging my passwords on my own solves the attack target problem. As long as I don't get too famous, the chances that an attacker is targetting my passwords specifically are quite low.


## Basics 
- The first step is obviously to download `pass`.
- Using whatever PGP program you have, create key-pair that the `pass` program will use.
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
One of the really nice things about 1Password was the browser extensions that allowed to automatically insert credentials. `pass` doesn't come with anything of the sort. 
Thankfully, creating system to do the same thing is not that hard. The creator of `pass` has his version of it called [passmenu](https://git.zx2c4.com/password-store/tree/contrib/dmenu), which is where I got the inspiration. 

[This](https://github.com/eanyanwu/.dotfiles/blob/master/scripts/passmenu.sh) is my version of the script. Don't you worry, it's well commented :)

The last step to get this all working is to get a graphical version of the _pinentry_ program. From what I understand, _pinentry_ is the program that runs whevere a password-protected gpg key is accessed. The default version usually runs in the terminal where the key was accessed. This won't do for our situation because the script won't be running in a terminal.  
So we need a version of pinentry that will pop open a window instead of drawing to the terminal. For OpenBSD, I was able to find a gtk version that I installed using `pkg_add`. Once it was intalled, gpg somehow automatically defaulted to using the graphical version, so no further configuration was required.


## Syncing the passwords between devices

Won't it be nice if I can access my passwords from other devices like my phone? Yes it will. And will will also take some extra setup. 
TBD

