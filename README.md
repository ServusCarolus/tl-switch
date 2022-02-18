# `tl-switch` Readme

Switch context between multiple vanilla TeXLive (TL) instances installed under `/usr/local/texlive` and/or the Linux distro version of TeXLive.

This shell script has been tested on Ubuntu, Linux Mint, and Manjaro.

The script and installation are based on the answers at:
https://tex.stackexchange.com/questions/150892/multiple-texlive-installations

# Why Multiple TL Instances?

Generally speaking, one is advised to use the distribution of TL packaged with one's distribution of GNU/Linux. Reasons for doing that are:
* The fonts and file viewers (e.g., `evince`) are integrated well with both TL and the system.
* This means less manual fiddling by the user to get things to work correctly.
* This also facilitates clean upgrades.

In certain cases, however, using the distro packages might be problematic if. e.g.:
* One has published documents using specific versions of TL.
* One is developing packages and needs to be close to current development.
* One is developing packages and needs to be backward-compatible.

# *Hic Sunt Dracones*

Using `tl-switch` allows both root and multiple regular users independently to switch context between multiple TL versions. While each user might log out and back in again to ensure a full context switch, the whole machine does not need to go down.

Other solutions have assumed that one user becomes root, changes the context, drops back to regular user, logs out, and logs in again. Such solutions do not address either having multiple users or having root do maintenance on multiple TL instances independently of other users.

So why the dragons? By design, this script adds complexity. That action increases the reasons why things might break. Upgrades will not be simple. Maintenance will not be simple. Subtle errors might result.

## Preliminary Caveats

* One never should use `tlmgr` to set up system-wide symlinks when using `tl-switch`. Otherwise, undocumented behavior will occur.

* The directory `/usr/local/texlive/texmf-local` is used by every version of TL. Anything in this directory must be compatible with all the installed versions.

* One always must be clear regarding the binary context of the current version of TL in use. It helps if one uses `which latex` or the like to show which path is being used. Moreover, if one happen to be user "bob" (think J.R. "Bob" Dobbs), then when using `tl-switch` one's path might be `/opt/tex/bob/bin`. That is a symbolic link (symlink). One would use, e.g., `readlink -f /opt/tex/bob/bin/latex` or `realpath /opt/tex/bob/bin/latex` to see where the actual binary is located. Losing track of this context can result in one working with `tlmgr` in ways that cause subtle errors. This context is set by the PATH environment variable.

* Other environment variables point to the context in use. It can be helpful to check them. For example, if one is running TL 2020, the following will be the case:
```
TEXDIR: "/usr/local/texlive/2020"
TEXMFSYSCONFIG: "/usr/local/texlive/2020/texmf-config"
TEXMFSYSVAR: "/usr/local/texlive/2020/texmf-var"
TEXMFVAR: "~/.texlive2020/texmf-var"
TEXMFCONFIG: "~/.texlive2020/texmf-config"
```
* These other environment variables do not change (there is no year number in them):
```
TEXMFHOME: "~/texmf"
TEXMFLOCAL: "/usr/local/texlive/texmf-local"
```
* One should examine `~/texmf`, TeX IDE config files (see below; they may not respect PATH), shell resource files (`.profile`, `.bashrc`, `.cshrc`, etc.), and any other settings with paths and program references.

## AUR and Debian `equivs`

There is an AUR package https://aur.archlinux.org/packages/texlive-installer that allows one to install the current TL. This package satisfies the dependencies in Arch, Manjaro, etc.

Instructions at https://tug.org/texlive/debian.html will permit one to do much the same thing with Debian-based distros.

Using `tl-switch` will work with these methods, but do take care. In particular, using `tl-switch no` could result in the current TL binaries not being found in PATH unless one ensures manually that they can be found.

# Caveat: (Mis-)Using `sudo`

## Distro Differences

Among Linux distributions there are differences in how `sudo` works, as well as differences in how `.profile`, `.bashrc`, and other files are sourced. The install instructions try to address some common differences. One may need to adapt the installation to one's own distro.

## What Is Context Hell?

Observe caution when editing files with elevated privileges.

For example, `sudo echo "$HOME"` will point to the regular user's home directory, not that of the root user.

That is why using `sudo -i` or `su -` to become superuser is important.

Given potential ambiguities regarding context, one should avoid using relative paths when setting up `tl-switch`. One should use unambiguous, absolute paths to avoid undocumented behavior. Absolute paths start at the root directory and explicitly reference every subdirectory in the file path until one gets to the specific file.

### Get a Root Login Context

1. Ubuntu-based: `sudo -i <command>` or `sudo -i su`, followed by commands.

   Especially Ubuntu and its derivatives do not set up root's account in order to have greater security. One must gain root privileges via `sudo`. Debian. Mint, and so on may or may not require this, but Debian's implementation of `sudo` is a key factor in this section.
   
2. Arch-based and likely others: `sudo -i <command>` or `su -`, followed by commands.

### Some Online Fora Make Mistakes

Certain online fora (plural of *forum*) tell users to change system-wide config files instead of user settings. They also tend to advise running graphical, desktop-integrated applications with `sudo`.

Such advice is wrong on many levels and will break things:

1. Altering system-wide configs to address user configuration issues is poor practice that can lead to undocumented behavior.

2. Running graphical applications with `sudo` can modify multiple files and directories in a user's home directory and change their ownership to root.

3. One might see changes in ownership of the configuration files for the desktop environment, such as keys in `~/.config/dconf/user`. That could cause serious issues.

4. One might see changes of ownership in shared files, creating undocumented behavior.

One should try to stick with CLI (command-line interface) applications when running `sudo` or `su`.

One can mitigate the damage caused by bad advice using something like

    find "$HOME" -type f -user root

Then it is simple to use `sudo chown "$USER":"$USER"` on each file to fix the ownership problems. One also may need to fix file permissions.

# Installing `tl-switch`

## Step 1: Install Vanilla TL as Root

For installing vanilla TL see: https://www.tug.org/texlive/acquire.html

There is an AUR package: https://aur.archlinux.org/packages/texlive-installer

Links at https://tug.org/texlive/debian.html will aid one in installing vanilla TL in Debian-based distros.

* If one simply opts for the distro-specific packages, one might not be able to switch cleanly among TL versions.

* Note: Never install the symbolic links when installing vanilla TL and `tl-switch`.*

## Excursus: Making a Group

We do not implement this approach here. Nevertheless, one way to avoid context problems with `sudo` is to install TL in a directory owned by the TeX user group. Yet this could pose a security risk. Here are two ways of doing this, using commands common to many distros:

* All users in the `texusers` group have read, write, and searchability:
```
# start logged in as the user who will be admin
sudo groupadd texusers
sudo usermod -a -G texusers "$USER"
sudo mkdir -p /usr/local/texlive
sudo chgrp -R texusers /usr/local/texlive
sudo chmod -R 2775 /usr/local/texlive
```
* All users in the `texusers` group have read access and searchability, while only one user in the group has write access:
```
# start logged in as the user who will be admin
sudo groupadd texusers
sudo usermod -a -G texusers "$USER"
sudo mkdir -p /usr/local/texlive
sudo chown -R "$USER":texusers /usr/local/texlive
sudo chmod -R 2755 /usr/local/texlive
```
Now, one is ready to install TL as part of the texusers group.
See also: https://www.tecmint.com/create-a-shared-directory-in-linux/

## Step 2: Create Directories

We create paths for each user to create directory links:

    sudo mkdir -p /opt/tex/root

One only needs the `-p` option once; all the other user paths will build off the `/opt/tex` path after it is first created.

We repeat the following two lines, substituting each user's account name for `<user>`:

    sudo mkdir /opt/tex/<user>
    sudo chown <user>:<user> /opt/tex/<user>

After these directories are created, one *will not see* a path change yet. These directories merely serve as a point where a symlink to a TL bin directory will be created by the `tl-switch` script.

## Step 3: Modifying profiles

Now we need to have each user's login shell find the right command path.

Linux distros take different approaches to sourcing `~/.profile` and `~/.bashrc`.

Usually `~/.profile` is sourced for a login shell.

In this step we will put the following snippet **somewhere**, depending on the distribution used:

    if [ -d "/opt/tex/$USER/bin" ] ; then
        PATH="/opt/tex/$USER/bin:$PATH"
    fi

### Generic User, MX-Linux

Some Debian-based distros do not necessarily source either `/etc/profile` or `~/.profile` for normal users: https://forum.mxlinux.org/viewtopic.php?t=49505.

One can source `~/.profile` in `~/.xsessionrc` as a workaround and put the snippet above in `~/.profile`.

Or one could put the snippet above in `~/.bashrc`. That seems to be sourced in all cases.

### Generic User, others

Linux Mint, Manjaro, and many other distributions do source `~/.profile` at login. We put the snippet above in each user's `.profile`.

### Root, MX-Linux

MX-Linux sources root's `.profile`, so one can put the snippet above in that file.

When editing root's `.profile`, remember to use `su -`, `sudo su -`, or specify `/root/.profile` as the file.

### Root, others

It appears in many distributions that root's `.profile` is *not sourced*. Instead, root's `.bashrc` is sourced.

If we put the same snippet above in root's `.bashrc`, everything should work as expected.

When editing root's `.bashrc`, remember to use `su -`, `sudo su -`, or specify `/root/.bashrc` as the file.

### Alternate Fixes

Here are things to try if the solutions above do not work.

1. Put the snippet in everyone's `.bashrc`, then add to everyone's `.profile` the following:

        source ~/.bashrc

    That would renew the path environment every time one opens a terminal.

2. Set terminals to open a login shell by default.

3. If making system-wide changes on a server, one might need to alter the hidden files in `/etc/skel` in order to ensure automatic setup for each new user.

After the snippet is inserted where needed, one *will not see* a path change yet. But we are ready to set that up.

## Step 4: Install the Script

Download or clone this repository. Unpack the archive if needed. Go to the directory where the archive was unpacked or the repository ws cloned.

Locate the `tl-switch` script, then type:

    sudo cp ./tl-switch /usr/local/bin
    chmod +x /usr/local/bin/tl-switch
    
All users now should have access to running the script, since `/usr/local/bin` usually is in the command search path.

The command `ls /opt/tex` should show one subdirectory for each user.

As root, run `tl-switch yes`. One should see something like the following:

    Context switch path exists: /opt/tex/root
    Setting /opt/tex/root/bin to target:
    /usr/local/texlive/2020/bin/x86_64-linux

The year `20xx` should correspond to the latest version of TL installed. Exit from the root shell to the normal user's login. Again, run `tl-switch yes`. For the normal user `bob`, one should see something like:

    Context switch path exists: /opt/tex/bob
    Setting /opt/tex/bob/bin to target:
    /usr/local/texlive/2020/bin/x86_64-linux

Note, however, that `$PATH` has not been updated yet. That happens in the next step.

## Step 5: Reboot

After the install procedure is done, it is good to restart the machine before using TeXLive so that the paths for root and the users can be updated properly.

## Step 6: Test

Run `which latex` to show which path is being used.

Check PATH. For the normal user "bob", one would expect to see something like:

    echo $PATH
    /opt/tex/bob/bin:/home/bob/.local/bin:/usr/local/bin:/usr/bin:/bin:/usr/local/sbin

One should see much the same with root.

If one does not see `/opt/tex/<user>/bin`, where `<user>` corresponds with the current user, then check the user's `.profile` and `.bashrc`, including root (as mentioned above). Also check if the directories exist under `/opt/tex` with `ls -l /opt/tex`, which also should indicate ownership and permissions.

Test the symlinked directory for user "bob" with, e.g., `readlink -f /opt/tex/bob/bin/latex` or `realpath /opt/tex/bob/bin/latex` to see where the actual binary is located. Do the same for all users.

## Step 7: Fix IDE Paths

TeX authoring has been enhanced by many IDEs. Yet these programs sometimes do not reference the `PATH` environment variable properly. Failing to do this step will cause IDE sessions to fail even as running TL binaries from the CLI works.

Different IDEs use different configuration settings to find the TL binaries in use. One must fix this by putting `/opt/tex/$USER/bin` as the first directory in the path used by the IDE, where `$USER` is replaced by the current username. If the IDE does not disambiguate $USER automatically, then one must substitute one's username for `$USER` manually. That will ensure the proper function of this script.

# Using `tl-switch`

## As Root Using `sudo` (Debian-based)

Some Linux distros will permit an installer to insert a shell script into `/etc/profile.d` to alter the command search path for all users. Debian-based distros treat such an approach as a security risk. They build `sudo` to use `secure_path`. This could complicate using `tl-switch` and `tlmgr` with `sudo`. Some workarounds include:

1. Using the current user's path to find the right version of `tlmgr`:

        sudo -i env PATH=$PATH tlmgr -gui
        sudo -i env PATH=$PATH tlmgr update -self -all
   or
        sudo su -
        env PATH=$PATH tlmgr -gui
        env PATH=$PATH tlmgr update -self -all

2. Use the common group route discussed above, using `sudo` to create `/usr/local/texlive/`, the directories mentioned above under `/opt`, and setting ownership, group membership, and permissions.

3. As a last resort, redefine `sudo` in various ways, as in:
https://stackoverflow.com/questions/257616/why-does-sudo-change-the-path

## General

As a new version of TL is released, the default year is updated. When a user (or root) wants to enable access to the most current year release of vanilla TL under `/usr/local/texlive`, one need only type:

    tl-switch yes

To specify another installation under `/usr/local/texlive`, such as an older edition, use, e.g.:

    tl-switch yes 2018

To disable vanilla TL and use the distro version, one need only type:

    tl-switch no

The caveat here is if the distro version is not used (which is recommended), and vanilla TL, the AUR package, or Debian `equivs` version is used instead, then one will have to account for that situation. The easieest fix is, e.g., if TL 2020 is used, then use `tl-switch yes 2020`.

## Avoid Context Hell

After using `tl-switch` to change TL versions, it is best to log out immediately, then log in again to get all the paths and contexts right.

1. Logging out and back in after using `tl-switch` will help `kpsewhich` work correctly.
 
2. Any path that does not exist as a symlink at login will get skipped. Sourcing `.profile` and the like will update the path in a terminal window, but only in that specific terminal window. Otherwise, the system version generally will be used until one logs out and in again.
