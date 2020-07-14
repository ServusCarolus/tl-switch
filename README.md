# tl-switch
Switch context between multiple vanilla TeXLive instances installed under /usr/local/texlive and the Linux distro version of TeXLive. This shell script has been tested on Ubuntu, Linux Mint, and Manjaro.

The script and installation are based on the answers at:
https://tex.stackexchange.com/questions/150892/multiple-texlive-installations

# Caveat: Multiple TL Instances
Using this script adds complexity, which adds more reasons why things might break. It adds complexity to the task of installing and maintaining multiple TL versions, but it also allows users to select different versions of TL independent of each other. Those are the tradeoffs.

The directory `/usr/local/texlive/texmf-local` is used by every version of TL in use. If one should install different versions of TL, one must ensure that anything in the above directory is compatible with all the installed versions.

If one has more than one TL version installed, this may confuse `tlmgr` unless one sets the binary context for root, logs out to the login window, then logs in again. Failure to do so can cause undocumented behavior. For example, if one has TL 2019 and TL 2020 installed, it is possible to set the context to TL 2019 in a terminal window, tell `tlmgr` to remove TL 2019, and have report of a successful removal, but nothing is done. That is where logging out and in hopefully fixes things. See the general notes below for more.

## Arch-based Notes

There is an AUR package that allows one to install the current TL, yet tells Arch, Manjaro, etc., that its package dependencies are met. This script will work with that package, but it is recommended that one not make any major modifications that would wreck the install and removal scripts of the package and potentially create a lot of work to remove the package and ensure the integrity of the package database, cache, and so on.

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
It is quite probable that an IDE will not use the $PATH variable. Instead, it will use its own mechanism for handling paths. One must fix this by putting `/opt/tex/`<user>`/bin` as the first directory in the path used by the IDE, where <user> is replaced by the current username. That will ensure the proper function of this script.

Also, be aware that any symbolic links or stale config files that point to a different path than the $PATH shell variable can cause undocumented behavior. When using this script, it is recommended to check all IDE settings, local texmf trees, symbolic links, etc., to prevent some paths being used in some circumstances, while others are used in different ones.

Observe caution when editing files with elevated privileges. For example, `sudo echo "$HOME"` will point to the regular user's home directory, not that of the root user. That is why using `su -` to become superuser is important (see below). One should avoid all shortcuts like `~./` in file paths when setting up `tl-switch`. One should use unambiguous, full paths to avoid undocumented behavior.

# Caveat: Distro Differences and `sudo`

## Distro Differences and Root Environments

Among Linux distributions there are differences in how `sudo` works, as well as differences in how `.profile`, `.bashrc`, and other resource and configuration files are sourced and how their changes are retained.

One need only compare a Debian-based `.bashrc` with an Arch-based `.bashrc` to get the sense that some very different philosophies are in play. This is allowable because Unixlike operating systems offer much flexibility within a general framework. Debian's philosophy tends to be more restrictive in order to make things generally more robust. This tends to work well for end-users wishing to use things out of the box. But if you want to use this script, you are not one of those users.

Using `sudo` will have different behavior on different Linux distributions. In the case where one must have a clean root environment, please use:

1. Ubuntu-based: `sudo su -`

   Especially Ubuntu does not set up root's account in order to have greater security. One must gain root privileges via `sudo`. Debian may or may not require this, but Debian's implementation of `sudo` is a key factor in this section.
   
2. Arch-based and likely others: `su -`

The hyphen after `su` switches all environment references to root; otherwise, the environment will use the variables from the user's home where `su` was launched. Thus, one should avoid running many desktop-integrated GUI programs using `sudo`. Doing so may create files owned by root in one's home directory tree. That can prevent user programs from saving information properly. One can find such files using something like `find "$HOME" -type f -user root`. Then it is simple to use `sudo chown` to fix the ownership. This tends to be an issue created by quick-and-dirty, bad advice on many online fora.

When using `tl-switch`, one should type, e.g., `sudo su -` to switch contexts to the superuser before running `tlmgr`. In distributions where the superuser has a password set, one can just use `su -`. One definitely needs to use the full context switch with the `tl-switch` script, even if one does not need to do that with `install-tl`.
        
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

# Excursus: Make a Group
We include this excursus for completeness, but we do not implement this approach with `tl-switch` in the numbered setup steps below. Nevertheless, experienced users can implement this alternate option.

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

# Step 1: Install Vanilla TL
For installing vanilla TL see: https://www.tug.org/texlive/acquire.html

See also the AUR package if appropriate: https://aur.archlinux.org/packages/texlive-installer

**Note: Never install the symbolic links when installing vanilla TL.**

# Step 2: Create Directories
We create paths for each user to create directory links:

    sudo mkdir -p /opt/tex/root

One only needs the `-p` option once; all the other user paths will build off the `/opt/tex` path after it is first created.

We repeat the following two lines for each user, most likely substituting each username for $USER, e.g. automatically with a script:

    sudo mkdir "/opt/tex/$USER"
    sudo chown "$USER":$USER" "/opt/tex/$USER"

or manually, for the Slack-tastic user `bob`:

    sudo mkdir /opt/tex/bob
    sudo chown bob:bob /opt/tex/bob

# Step 3: Modifying profiles

## Generic
We then put this snippet in each user's `.profile`:

    if [ -d "/opt/tex/$USER/bin" ] ; then
        PATH="/opt/tex/$USER/bin:$PATH"
    fi

## Debian-based
Here root's `.profile` is not sourced in the same manner. If we put the same snippet above in root's `.bashrc`, everything will work as expected.

Another approach would put the snippet in everyone's `.bashrc`, then add `source .bashrc` to everyone's `.profile`. That would renew the path environment every time one opens a terminal. Or one can set terminals to open a login shell by default. Consider also visiting the (hidden) files in `/etc/skel` if making new users on a server in order to make changes automatically, but know what you are doing.

When editing root's `.bashrc`, remember to use `su -`, `sudo su -`, or specify `/root/.bashrc` as the file.

# Step 4: Install the Script
We go to the directory where we downloaded or cloned the repository and locate the `tl-switch` script. We then type:

    sudo cp ./tl-switch /usr/local/bin
    chmod +x /usr/local/bin/tl-switch
    
All users now will have access to running the script.

# Step 5: Reboot
After the install procedure is done, it is good to restart the machine before using TeXLive so that the paths for root and the users can be updated properly.

# Step 6: Switching to and from Vanilla TeXLive
As a new version of TL is released, the default year is updated. When a user (or root) wants to enable access to the most current year release of vanilla TL under `/usr/local/texlive`, one need only type:

    tl-switch yes

To specify another installation under `/usr/local/texlive`, such as older editions or pretests, use, e.g.:

    tl-switch yes 2018

To disable vanilla TL and use the distro version, one need only type:

    tl-switch no

# Context Changes

**After using `tl-switch` to change TL versions, it is best to log out immediately, then log in again to get all the paths and contexts right.**

 1. If the initial path was `/opt/tex/$USER/bin` and you run `tl-switch no`, the original path will not find anything and the system distro will work out of `/usr/bin` and `/usr/share/texlive`. Invoking `tl-switch yes` will cause the original path to work again and the distro version will not be used. BUT this context switch can confuse programs like `kpsewhich`.
 
 2. If the initial path was `/usr/bin`, then running `tl-switch yes` will not change `$PATH`. Even if `/opt/tex/$USER/bin` is in the path, because it did not exist as a symlink at login, it gets skipped. Sourcing `.profile` and the like will update the path in a terminal window, but only in that specific terminal window. Otherwise, the system version generally will be used until one logs out and in again.
 
 3. One first should install/update the system TeX packages as root. After that, one can install vanilla TeXlive and use `tl-switch yes` to have the default TeXlive for the root account point to the desired installation. This allows one to become root and update TL, while users can point to whatever TL version they want. When updating system TeXlive packages, is is recommended first to use `tl-switch no` before updating, then `tl-switch yes` to revert back to the desired vanilla TL version.

See installation Step 3 above for more on how to tackle these issues.

# Final Thoughts
An immediate downside to this method is needing to, e.g., `su -` to switch contexts to the superuser before running `tlmgr`. Another downside is adding a level of complexity when using different versions and updating packages. Its benefits include isolating users from each other and allowing each user to change contexts without extensive system modification or restarting the system. This script is designed with a server or multiuser university environment in mind.
