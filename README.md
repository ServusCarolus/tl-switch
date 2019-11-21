# tl-switch
Switch context between vanilla TeXLive installed under /usr/local/texlive and the Linux distro version of TeXLive installed on a system like Debian, Ubuntu, Mint, etc.

The script and installation are based on ths answers at:
https://tex.stackexchange.com/questions/150892/multiple-texlive-installations

# Step 1: Install Vanilla TL
For installing vanilla TL see: https://www.tug.org/texlive/acquire.html

**Note: Please do not install the symbolic links when installing vanilla TL.**

# Step 2: Make Context for `sudo`
Installing TL as root usually is OK, but one must additionally create a shell script in `/etc/profile.d` in order that the `sudo` command have proper context. Otherwise, `sudo tlmgr version` will show the distro version and not vanilla TL even if one does that as the root user! We create the file `/etc/profile.d/texlive.sh` and open it for editing, such as:

    sudo nano /etc/profile.d/texlive.sh
    
Then we insert the following snippet:

    if [ -d "/opt/tex/root/bin" ] ; then
        PATH="/opt/tex/root/bin:$PATH"
    fi

We save the file and we are done with this step.

# Excursus: Context and `sudo`
When using sudo, $USER will not necessarily point to root; test with:

        sudo echo "$USER"
        
So even if one gets superuser privileges, one's environment and similar context may not have changed. Although the GUI interface of `tlmgr` is not a problem, one should avoid using many desktop-integrated GUI programs while running `sudo`. Doing so may create files owned by root in one's home directory tree. That can prevent user programs from saving information properly.
   
To do a full context switch, do either `su` or `sudo su`, depending on the distribution.

# Excursus: Make a Group
Another way to handle this issue is to make the TeXLive installation writeable to all TeX users. Then the use of `sudo` might be avoided during installation. The problem here is that chaos might ensue if multiple users set conflicting configurations. Nevertheless, we include this for completeness:

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

# Step 3: Create Directories
We create paths for each user to create directory links:

    sudo mkdir -p /opt/tex/root
    sudo mkdir -p "/opt/tex/$USER"
    sudo chown -R "$USER":$USER" "/opt/tex/$USER"
        
We repeat the final two lines for each user, most likely substituting each username for $USER, e.g.:

    sudo mkdir -p /opt/tex/bob
    sudo chown -R bob:bob /opt/tex/bob

The arguments to `mkdir` and `chown` avoid error messages being generated.

# Step 4: Modifying profiles
We put this snippet in each user's `.profile` and in root's `.bashrc`:

    if [ -d "/opt/tex/$USER/bin" ] ; then
        PATH="/opt/tex/$USER/bin:$PATH"
    fi
        
When editing root's `.bashrc`, remember to use `sudo su` or specify `/root/.bashrc` as the file. Otherwise `sudo nano ~/.bashrc` refers to the user's `.bashrc` file instead.

# Step 5: Install the Script
We go to the directory where we downloaded or cloned the repository and locate the `tl-switch` script. We then type:

    sudo cp ./tl-switch /usr/local/bin
    chmod +x /usr/local/bin/tl-switch
    
All users now will have access to running the script.

# Step 6: Reboot
After the install procedure is done, it is good to restart the machine before using TeXLive so that the paths for root and the users can be updated properly.

# Step 7: Switching to and from Vanilla TeXLive
When a user (or root) wants to enable access to vanilla TL 2019, one need only type:

    tl-switch yes

To specify another installation under `/usr/local/texlive`, use, e.g.:

    tl-switch yes 2018

To disable vanilla TL, one need only type:

    tl-switch no
