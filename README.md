# tl-switch
Switch context between vanilla TeXLive installed under /usr/local/texlive and the Linux distro version of TeXLive installed on a system like Debian, Ubuntu, Mint, etc.

The script and installation are based on ths answers at:
https://tex.stackexchange.com/questions/150892/multiple-texlive-installations

# Caveat: A Word about `sudo`
Even if one creates a shell script in `/etc/profile.d` in order to put a symbolic link to the vanilla TL path before `/usr/bin` in the command search path, the `sudo` command will not follow the link by default.

The issue is that Debian and friends build `sudo` to use `secure_path`. There are various workarounds to this issue, depending on the user's preference. See:
https://stackoverflow.com/questions/257616/why-does-sudo-change-the-path

When installing vanilla TL as root and using this script, one must type, e.g., `sudo su` to switch contexts to the superuser before running `tlmgr`. Alternatives include:

1. The least invasive route, e.g.:

        sudo env PATH=$PATH tlmgr -gui
    
2. Use the common group route below and do not use `sudo`, but set the directories to what they would have been if you did.

Regardless of the issues above, normal use works as expected.

Observe caution when editing files. For example, `sudo echo "$USER"` should point to the regular user, not root. That means one should avoid shortcuts like `~./` in file paths. One should use unambiguous, full paths.
        
Although the GUI interface of `tlmgr` will not create files owned by root when run via `sudo`, one should avoid using many desktop-integrated GUI programs while running `sudo`. Doing so may create files owned by root in one's home directory tree. That can prevent user programs from saving information properly.
   
To do a full context switch, do either `su` or `sudo su`, depending on the distribution.

# Excursus: Make a Group
Another way to avoid problems with `sudo` is to make the TeXLive installation writeable to all TeX users. The problem here is that chaos might ensue if multiple users meddle with the installation. We include this for completeness:

    sudo addgroup texusers
    sudo addgroup "$USER" texusers
    sudo mkdir -p /usr/local/texlive
    sudo chgrp -R texusers /usr/local/texlive
    sudo chmod -R 2775 /usr/local/texlive

Note that adduser and addgroup are Debian-isms; other distributions (and Debian-based ones too) have the commands `useradd` and `groupadd`. See the man pages for those commands. Thus, you would use instead:

    sudo groupadd texusers
    sudo usermod -a -G texusers "$USER"

Then one can install TL as part of the texusers group.
See also: https://www.tecmint.com/create-a-shared-directory-in-linux/

# Step 1: Install Vanilla TL
For installing vanilla TL see: https://www.tug.org/texlive/acquire.html

**Note: Never install the symbolic links when installing vanilla TL.**

# Step 2: Create Directories
We create paths for each user to create directory links:

    sudo mkdir -p /opt/tex/root
    sudo mkdir "/opt/tex/$USER"
    sudo chown "$USER":$USER" "/opt/tex/$USER"
        
We repeat the final two lines for each user, most likely substituting each username for $USER, e.g.:

    sudo mkdir /opt/tex/bob
    sudo chown bob:bob /opt/tex/bob

# Step 3: Modifying profiles
We put this snippet in each user's `.profile` and in root's `.bashrc`:

    if [ -d "/opt/tex/$USER/bin" ] ; then
        PATH="/opt/tex/$USER/bin:$PATH"
    fi

Another approach would put the snippet in everyone's `.bashrc`, then add `source .bashrc` to everyone's `.profile`. That would renew the path environment every time one opens a terminal. Or one can set terminals to open a login shell.

When editing root's `.bashrc`, remember to use `sudo su` or specify `/root/.bashrc` as the file. Otherwise `sudo nano ~/.bashrc` refers to the user's `.bashrc` file instead.

# Step 4: Install the Script
We go to the directory where we downloaded or cloned the repository and locate the `tl-switch` script. We then type:

    sudo cp ./tl-switch /usr/local/bin
    chmod +x /usr/local/bin/tl-switch
    
All users now will have access to running the script.

# Step 5: Reboot
After the install procedure is done, it is good to restart the machine before using TeXLive so that the paths for root and the users can be updated properly.

# Step 6: Switching to and from Vanilla TeXLive
When a user (or root) wants to enable access to vanilla TL 2019, one need only type:

    tl-switch yes

To specify another installation under `/usr/local/texlive`, use, e.g.:

    tl-switch yes 2018

To disable vanilla TL and use the distro version, one need only type:

    tl-switch no

If one changes context in the middle of a session, the search paths will not change. One way to tackle that (somewhat) is mentioned in Step 3 above.

# Final Thoughts
An immediate downside to this method is needing to, e.g., `sudo su` to switch contexts to the superuser before running `tlmgr`. Its benefits include isolating users from each other and allowing one to change contexts without extensive system modification. Yet contexts only should be changed before logging out and back in again to avoid problems.
