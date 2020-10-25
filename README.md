# `tl-switch` Readme

Switch context between multiple vanilla TeXLive instances installed under /usr/local/texlive and the Linux distro version of TeXLive. This shell script has been tested on Ubuntu, Linux Mint, and Manjaro.

The script and installation are based on the answers at:
https://tex.stackexchange.com/questions/150892/multiple-texlive-installations

# Caveat: Multiple TL Instances

Using this script adds complexity, which adds more reasons why things might break. It adds complexity to the task of installing and maintaining multiple TL versions, but it also allows users to select different versions of TL independent of each other. Those are the tradeoffs.

The directory `/usr/local/texlive/texmf-local` is used by every version of TL in use. If one should install different versions of TL, one must ensure that anything in the above directory is compatible with all the installed versions.

If one has more than one TL version installed, this may confuse `tlmgr` unless one sets the binary context for root, logs out to the login window, then logs in again. Failure to do so can cause undocumented behavior. For example, if one has TL 2019 and TL 2020 installed, it is possible to set the context to TL 2019 in a terminal window, tell `tlmgr` to remove TL 2019, and have report of a successful removal, but nothing is done. That is where logging out and in hopefully fixes things. See the general notes below for more.

## Arch-based Notes

There is an AUR package https://aur.archlinux.org/packages/texlive-installer that allows one to install the current TL, yet tells Arch, Manjaro, etc., that its package dependencies are met. This script will work with that package, but it is recommended that one not make any major modifications that would wreck the install and removal scripts of the package and potentially create a lot of work to remove the package and ensure the integrity of the package database, cache, and so on.

## General Notes

One should check that the following environment variables point to the right version to be managed. For example, if one is managing TL 2020, the following will be the case:

```
TEXDIR: "/usr/local/texlive/2020"
TEXMFCONFIG: "~/.texlive2020/texmf-config"
TEXMFHOME: "~/texmf"
TEXMFLOCAL: "/usr/local/texlive/texmf-local"
TEXMFSYSCONFIG: "/usr/local/texlive/2020/texmf-config"
TEXMFSYSVAR: "/usr/local/texlive/2020/texmf-var"
TEXMFVAR: "~/.texlive2020/texmf-var"
```

One could export the desired value to these variables by creating a script to do so. If these variables are not pointing to the version to be managed, undocumented behavior can result. In the case of such undocumented behavior, one may have to do something like `sudo rm -rf /usr/local/texlive/2019/`, then remove all local folders in user directories and root's home that refer to TL 2019. As long as one has not created links in `/usr/local/bin`, which one should not do, there should be few, if any, side effects.

One should remind users to look at any configuration files in their home `texmf` directories and to look at GUI config files, shell resource files (`.profile`, `.bashrc`, `.cshrc`, etc.), and any other settings with paths and program references.

# Caveat: A Word about paths

It is quite probable that an IDE will not use the `$PATH` shell variable. Instead, it will use its own mechanism for handling paths. One must fix this by putting `/opt/tex/`<user>`/bin` as the first directory in the path used by the IDE, where <user> is replaced by the current username. That will ensure the proper function of this script.

Also, be aware that any symbolic links or stale config files that point to a different path than the $PATH shell variable can cause undocumented behavior. When using this script, it is recommended to check all IDE settings, local texmf trees, symbolic links, etc., to prevent some paths being used in some circumstances, while others are used in different ones.

Observe caution when editing files with elevated privileges. For example, `sudo echo "$HOME"` will point to the regular user's home directory, not that of the root user. That is why using `su -` to become superuser is important (see below). One should avoid all shortcuts like `~./` in file paths when setting up `tl-switch`. One should use unambiguous, full paths to avoid undocumented behavior.

# Caveat: Distro Differences and `sudo`

## Distro Differences and Root Environments

Among Linux distributions there are differences in how `sudo` works, as well as differences in how `.profile`, `.bashrc`, and other resource and configuration files are sourced and how their changes are retained.

One need only compare a Debian-based `.bashrc` with an Arch-based `.bashrc` to get the sense that some very different philosophies are in play. This is allowable because Unixlike operating systems offer much flexibility within a general framework.

Debian's philosophy tends to be more restrictive in order to make things generally more robust. This tends to work well for end-users wishing to use things out of the box. But if you want to use this script, you likely are not one of those users.

### Invoking `su -` with a Root Environment

Using `sudo` will have different behavior on different Linux distributions. In the case where one must have a clean root environment, please use:

1. Ubuntu-based: `sudo su -`

   Especially Ubuntu does not set up root's account in order to have greater security. One must gain root privileges via `sudo`. Debian. Mint, and so on may or may not require this, but Debian's implementation of `sudo` is a key factor in this section.
   
2. Arch-based and likely others: `su -`

For both of the points above, the hyphen after `su` switches all environment references to root. Otherwise, the environment will use the variables from the user's home where `su` was launched. That could introduce subtle errors. One definitely needs to use this full context switch with the `tl-switch` script. Even when using `install-tl` or `tlmgr`, it may be helpful to use `su -` to switch contexts fully to the superuser. Note that some distros have packages that will run the latest `install-tl` for you.

### Fixing the Mess that Some Online Fora Make

When searching online fora (the neutral plural of *forum*), one often sees someone suggest that a user invoke a desktop-integrated application with `sudo` via a terminal. That is a mistake on many levels and can cause undocumented behavior because it could make the root user, not the normal user, the owner of one or more of the following:

1. That application may have its own hidden directory, e.g. `.appname`, where it saves configurations, incremental changes, and such. That could be in one's home directory or even under `~/.local/share`.

2. It may change the contents of a directory, e.g. `appname`, which exists as a subdirectory of `~/.config` and possibly `~/.cache`.

3. It could change desktop configuration files, such as keys in `~/.config/dconf/user`.

4. It could modify shared files that are synchronized over a LAN or an Internet service, such as DropBox.

If root owns one or more of these files, that could cause the desktop-integrated application not to run properly. One generally should avoid running desktop-integrated GUI applications with `sudo`. Try to use command-line tools that will not touch various files by default.

One can find such files using something like

    find "$HOME" -type f -user root

Then it is simple to use `sudo chown "$USER":"$USER"` on each file to fix the ownership problems. One also may need to fix permissions.

## Debian's `sudo` and `tlmgr`

Even if one were to create a shell script in `/etc/profile.d` in order to put a symbolic link to the vanilla TL path before `/usr/bin` in the command search path, the `sudo` command will not follow that link by default in Debian-based distributions.

The issue is that Debian and its derivatives build `sudo` to use `secure_path`. There are various workarounds to this issue, depending on the user's preferences.

Alternatives include:

1. Using `tlmgr` to manage TeX Live, one could just run it with `sudo` and take the least potentially invasive route using the current user's path if needed:

        sudo env PATH=$PATH tlmgr -gui
        sudo env PATH=$PATH tlmgr update -self -all
    
2. Use the common group route discussed in the excursus below and do not use `sudo` except when creating the directory `/usr/local/texlive/` and setting ownership, group membership, and permissions.

3. Redefine `sudo` in various ways, as in:
https://stackoverflow.com/questions/257616/why-does-sudo-change-the-path

## Excursus: Make a Group

**We do not implement this approach with `tl-switch` in the numbered setup steps below.**

We include this excursus only for completeness. Nevertheless, experienced users can implement this alternate option. YMMV.

Another way to avoid problems with `sudo` is to make the TeXLive installation writeable to all TeX users. The problem that results is that chaos might ensue if multiple users meddle with the installation. One could avoid that by using the group approach and letting only one user in that group have write privileges to the local installation. Both alternatives are shown below.

All users in the `texusers` group have read, write, and searchability:
```
# start logged in as the user who will be admin
sudo addgroup texusers
sudo addgroup "$USER" texusers
sudo mkdir -p /usr/local/texlive
sudo chgrp -R texusers /usr/local/texlive
sudo chmod -R 2775 /usr/local/texlive
```
All users in the `texusers` group have read access and searchability, while only one user in the group has write access:
```
# start logged in as the user who will be admin
sudo addgroup texusers
sudo addgroup "$USER" texusers
sudo mkdir -p /usr/local/texlive
sudo chown -R "$USER":texusers /usr/local/texlive
sudo chmod -R 2755 /usr/local/texlive
```

Note that adduser and addgroup are Debian-isms; other distributions (and Debian-based ones too) have the commands `useradd` and `groupadd`. See the man pages for those commands. Thus, you would use instead, e.g.:

    sudo groupadd texusers
    sudo usermod -a -G texusers "$USER"

Then one can install TL as part of the texusers group.
See also: https://www.tecmint.com/create-a-shared-directory-in-linux/

End of excursus.

# Installing `tl-switch`

## Step 1: Install Vanilla TL

For installing vanilla TL see: https://www.tug.org/texlive/acquire.html

See also the AUR package if appropriate: https://aur.archlinux.org/packages/texlive-installer

**Note: Never install the symbolic links when installing vanilla TL and `tl-switch`.**

## Step 2: Create Directories

We create paths for each user to create directory links:

    sudo mkdir -p /opt/tex/root

One only needs the `-p` option once; all the other user paths will build off the `/opt/tex` path after it is first created.

We repeat the following two lines for each user, most likely substituting each username for $USER, e.g. automatically with a script:

    sudo mkdir "/opt/tex/$USER"
    sudo chown "$USER":$USER" "/opt/tex/$USER"

or manually, for the user `bob` (who has much Slack):

    sudo mkdir /opt/tex/bob
    sudo chown bob:bob /opt/tex/bob

After these directories are created, one *will not see* a path change yet. These directories merely serve as a point where a symlink to a TL bin directory will be created by the `tl-switch` script.

## Step 3: Modifying profiles

Different Linux distributions source `~/.profile` and `~/.bashrc` differently. This suboptimal situation makes designing a universal approach to configuration quite difficult, and it is certainly annoying to users. There are interactive and non-interactive shells, and both have login and non-login variants. Different files are sourced in each case, usually `~/.profile` for a login shell. See https://askubuntu.com/questions/438150/scripts-in-etc-profile-d-being-ignored. Yet this is not necessarily consistent, for example, even among distros in the Debian ecosystem.

In this step we will put the following snippet **somewhere**, depending on the distribution used:

    if [ -d "/opt/tex/$USER/bin" ] ; then
        PATH="/opt/tex/$USER/bin:$PATH"
    fi

### Generic User, MX-Linux

It seems that some Debian-based distros do not necessarily source either `/etc/profile` or `~/.profile` for normal users: https://forum.mxlinux.org/viewtopic.php?t=49505.

One can source `~/.profile` in `~/.xsessionrc` as a workaround and put the snippet above in `~/.profile`.

Or one could put the snippet above in `~/.bashrc`. That seems to be sourced in all cases.

### Generic User, others

Linux Mint, Manjaro, and many other distributions do source `~/.profile` at login. We put the snippet above in each user's `.profile`.

### Root, MX-Linux

MX-Linux sources root's `.profile`, so one can put the snippet above in that file.

When editing root's `.profile`, remember to use `su -`, `sudo su -`, or specify `/root/.profile` as the file.

### Root, others

It appears in many distributions that root's `.profile` is *not sourced* in the same manner as that of a normal user. Instead, root's `.bashrc` is sourced.

If we put the same snippet above in root's `.bashrc`, everything will work as expected. When editing root's `.bashrc`, remember to use `su -`, `sudo su -`, or specify `/root/.bashrc` as the file.

### Alternate Fixes

Here are things to try if the solutions above do not work.

1. Put the snippet in everyone's `.bashrc`, then add to everyone's `.profile` the following:

        source ~/.bashrc

    That would renew the path environment every time one opens a terminal.

2. Set terminals to open a login shell by default.

3. If making system-wide changes on a server, one might need to alter the hidden files in `/etc/skel` in order to ensure automatic setup for each new user.

Even after this step, one *will not see* a path change yet.

## Step 4: Install the Script

We go to the directory where we downloaded or cloned the repository and locate the `tl-switch` script. We then type:

    sudo cp ./tl-switch /usr/local/bin
    chmod +x /usr/local/bin/tl-switch
    
All users now should have access to running the script, since `/usr/local/bin` usually is in the command search path.

The command `ls /opt/tex` should show one subdirectory for each user.

As root, run `tl-switch yes`. One should see something like the following:

    Context switch path exists: /opt/tex/root
    Setting /opt/tex/root/bin to target:
    /usr/local/texlive/2020/bin/x86_64-linux

Exit from the root shell to the normal user's login. Again, run `tl-switch yes`. For the normal user `bob`, one should see something like:

    Context switch path exists: /opt/tex/bob
    Setting /opt/tex/bob/bin to target:
    /usr/local/texlive/2020/bin/x86_64-linux

Note, however, that `$PATH` has not been updated yet. One must log out and log in at least.

## Step 5: Reboot or Logout/Login

After the install procedure is done, it is good to restart the machine before using TeXLive so that the paths for root and the users can be updated properly.

## Step 6: Test

The command `ls /opt/tex` should show one subdirectory for each user.

For the normal user `bob`, one would expect to see something like:

    echo $PATH
    /opt/tex/bob/bin:/home/bob/.local/bin:/usr/local/bin:/usr/bin:/bin:/usr/local/sbin

One should see much the same with `root`. If one does not see this, then check the user's `.profile` if it was edited correctly, or root's `.bashrc`. Also check if the directories exist under `/opt/tex` and if they have the correct permissions.

# Using `tl-switch`

## General

As a new version of TL is released, the default year is updated. When a user (or root) wants to enable access to the most current year release of vanilla TL under `/usr/local/texlive`, one need only type:

    tl-switch yes

To specify another installation under `/usr/local/texlive`, such as older editions or pretests, use, e.g.:

    tl-switch yes 2018

To disable vanilla TL and use the distro version, one need only type:

    tl-switch no

## Context Changes

**After using `tl-switch` to change TL versions, it is best to log out immediately, then log in again to get all the paths and contexts right.**

 1. If the initial path was `/opt/tex/$USER/bin` and you run `tl-switch no`, the original path will not find anything and the system distro will work out of `/usr/bin` and `/usr/share/texlive`. Invoking `tl-switch yes` will cause the original path to work again and the distro version will not be used. BUT this context switch can confuse programs like `kpsewhich`.
 
 2. If the initial path was `/usr/bin`, then running `tl-switch yes` will not change `$PATH`. Even if `/opt/tex/$USER/bin` is in the path, because it did not exist as a symlink at login, it gets skipped. Sourcing `.profile` and the like will update the path in a terminal window, but only in that specific terminal window. Otherwise, the system version generally will be used until one logs out and in again.
 
 3. One first should install/update the system TeX packages as root. After that, one can install vanilla TeXlive and use `tl-switch yes` to have the default TeXlive for the root account point to the desired installation. This allows one to become root and update TL, while users can point to whatever TL version they want. When updating system TeXlive packages, is is recommended first to use `tl-switch no` before updating, then `tl-switch yes` to revert back to the desired vanilla TL version.

See installation Step 3 above for more on how to tackle these issues.
