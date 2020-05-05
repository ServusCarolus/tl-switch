# tl-switch
Switch context between multiple vanilla TeXLive instances installed under /usr/local/texlive and the Linux distro version of TeXLive. This shell script has been tested on Ubuntuand Linux Mint.

The script and installation are based on the answers at:
https://tex.stackexchange.com/questions/150892/multiple-texlive-installations

# Caveat: Multiple TL Instances

Having multiple instances of TeXLive may make it necessary to remove an instance manually. For example, if one has TL 2019 and TL 2020 installed, and wants to remove TL 2019, `tlmgr` will report a removal, but do nothing. One must do something like `sudo rm -rf /usr/local/texlive/2019/`, then remove all local folders in user directories that refer to TL 2019. As long as one has not created links in `/usr/local/bin`, which one should not do, there should be no side effects. One should remember to look at GUI config files as well to resolve any paths and program references.

# Caveat: A Word about paths

It is quite probable that an IDE will not use the $PATH variable. Instead, it will use its own mechanism for handling paths. One must fix this by putting `/opt/tex/`<user>`/bin` as the first directory in the path used by the IDE, where <user> is replaced by the current username. That will ensure the proper function of this script.

Also, be aware that any symbolic links or stale config files that point to a different path than the $PATH shell variable can cause undocumented behavior. When using this script, it is recommended to check all IDE settings, local texmf trees, symbolic links, etc., to prevent some paths being used in some circumstances, while others are used in different ones. This author has been bitten by this mistake.

# Caveat: A Word about `sudo`
Even if one creates a shell script in `/etc/profile.d` in order to put a symbolic link to the vanilla TL path before `/usr/bin` in the command search path, the `sudo` command will not follow the link by default in Debian-based distributions.

The issue is that Debian and its derivatives build `sudo` to use `secure_path`. There are various workarounds to this issue, depending on the user's preference. See:
https://stackoverflow.com/questions/257616/why-does-sudo-change-the-path

When installing vanilla TL as root and using this script, one must type, e.g., `sudo su` to switch contexts to the superuser before running `tlmgr`. In distributions where the superuser has a password set, one can just use `su`. Alternatives include:

1. The least invasive route, e.g.:

        sudo env PATH=$PATH tlmgr -gui
        sudo env PATH=$PATH tlmgr update -self -all
    
2. Use the common group route below and do not use `sudo` except when creating the directory `/usr/local/texlive/` and setting ownership, group membership, and permissions.

3. Redefine `sudo` in various ways, as the link above discusses. YMMV.

Regardless of the issues above, with a normal user, the script works as expected.

Observe caution when editing files. For example, `sudo echo "$HOME"` will point to the regular user's home directory, not that of the root user. One should avoid all shortcuts like `~./` in file paths. One should use unambiguous, full paths.
        
Although the GUI interface of `tlmgr` will not create files owned by root when run via `sudo`, one should avoid running many desktop-integrated GUI programs using `sudo`. Doing so may create files owned by root in one's home directory tree. That can prevent user programs from saving information properly. One can find such files using something like `find "$HOME" -type f -user root`. Then it is simple to use `sudo chown` to fix the ownership. This tends to be an issue created by quick-and-dirty, bad advice on many online fora.

# Context Is Key
To do a full context switch, do either `su` or `sudo su`, depending on the distribution and whether or not one has set up a password for the root user. Using `su -` will change the current working directory to `/root`. Either method will point `$HOME` to the root account. That is very important to do when using this script, because the whole premise is to isolate users from each other.

# Excursus: Make a Group
We include this excursus for completeness, but we do not implement this approach in the numbered steps below. Nevertheless, experienced users can implement this alternate option.

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

**Note: Never install the symbolic links when installing vanilla TL.**

# Step 2: Create Directories
We create paths for each user to create directory links:

    sudo mkdir -p /opt/tex/root
    sudo mkdir "/opt/tex/$USER"
    sudo chown "$USER":$USER" "/opt/tex/$USER"
        
One only needs the `-p` option for the first `mkdir`; all the others will build off that same path. We repeat the final two lines for each user, most likely substituting each username for $USER, e.g.:

    sudo mkdir /opt/tex/bob
    sudo chown bob:bob /opt/tex/bob

One could turn this step into a script runnable via `sudo`.

# Step 3: Modifying profiles
We put this snippet in each user's `.profile` and in root's `.bashrc`:

    if [ -d "/opt/tex/$USER/bin" ] ; then
        PATH="/opt/tex/$USER/bin:$PATH"
    fi

The reason why root is treated differently is because the way in which the files are loaded in Debian-based Linux distributions. Note that for other setups, possibly different approaches may be needed.

Another approach would put the snippet in everyone's `.bashrc`, then add `source .bashrc` to everyone's `.profile`. That would renew the path environment every time one opens a terminal. Or one can set terminals to open a login shell by default. Consider also visiting the (hidden) files in `/etc/skel` if making new users on a server in order to make changes automatically, but know what you are doing.

When editing root's `.bashrc`, remember to use `su -`, `sudo su -`, or specify `/root/.bashrc` as the file. Otherwise `sudo nano ~/.bashrc` refers to the user's `.bashrc` file instead.

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

**If one changes context in the middle of a session, the search paths will not change. What this means is, in short, one must exit and restart the session to avoid errors. Here is why:**

 1. If the initial path was `/opt/tex/$USER/bin` and you run `tl-switch no`, the original path will not find anything and the system distro will work out of `/usr/bin` and `/usr/share/texlive`. Invoking `tl-switch yes` will cause the original path to work again and the distro version will not be used. BUT this context switch can confuse programs like `kpsewhich`.
 
 2. If the initial path was `/usr/bin`, then running `tl-switch yes` will not change `$PATH`. Even if `/opt/tex/$USER/bin` is in the path, because it did not exist as a symlink at login, it gets skipped. Sourcing `.profile` and the like will update the path in a terminal window, but only in that specific terminal window. Otherwise, the system version generally will be used until one logs out and in again.
 
 3. One first should install/update the system TeX packages as root. After that, one can install vanilla TeXlive and use `tl-switch yes` to have the default TeXlive for the root account point to the desired installation. This allows one to become root and update TL, while users can point to whatever TL version they want. When updating system TeXlive packages, is is recommended first to use `tl-switch no` before updating, then `tl-switch yes` to revert back to the desired vanilla TL version.

See Step 3 above for more on how to tackle these issues.

# Final Thoughts
An immediate downside to this method is needing to, e.g., `sudo su` to switch contexts to the superuser before running `tlmgr`. Another downside is adding a level of complexity when using different versions and updating packages. Its benefits include isolating users from each other and allowing one to change contexts without extensive system modification or restarting the system.
