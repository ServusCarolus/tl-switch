# tl19-switch
Switch context between vanilla TeXLive and the distro version

This script is used to switch between vanilla TeXLive 2019 and the version of TeXLive installed on a system like Debian, Ubuntu, Mint, etc.

# Installing Vanilla TL
For installing vanilla TL see: https://www.tug.org/texlive/acquire.html

Note: Please do not install the symbolic links when installing vanilla TL.

Installing TL as root usually is OK. One can skip the following step for non-root installation. To avoid installing TL as root, one could do something like:

    sudo addgroup texusers
    sudo addgroup "$USER" texusers
    sudo mkdir -p /usr/local/texlive
    sudo chgrp -R texusers /usr/local/texlive
    sudo chmod -R 2775 /usr/local/texlive

Note that adduser and addgroup are Debian-isms; other distributions (and Debian-based ones too) have the commands `useradd` and `groupadd`. See the man pages for those commands. Thus, you would use instead:

    sudo groupadd texusers
    sudo usermod -a -G texusers "$USER"

Then one can install TL 2019 as part of the texusers group.
See also: https://www.tecmint.com/create-a-shared-directory-in-linux/

The solution below is based on this answer:
https://tex.stackexchange.com/questions/150892/multiple-texlive-installations

We modify the procedure as follows:
1. We DO NOT modify /etc/profile.d because \$USER expands to root when that process runs. Only root can change the link for the TeX binary path. We want a multiuser solution.

2. We create paths for each user to create directory links:

        sudo mkdir -p /opt/tex/root
        sudo mkdir -p "/opt/tex/$USER"
        sudo chown -R "$USER":$USER" "/opt/tex/$USER"
    and so on for each user. When using sudo, $USER will not necessarily point to root; test with:

        sudo echo "$USER"
   One should not use desktop GUI programs while running `sudo`. Doing so can create files owned by root in one's home dir tree. That can prevent user programs from saving information properly. To do a full context switch do one of the following:

        su
        sudo su

3. We put this snippet in each user's .profile and in root's .bashrc:

        if [ -d "/opt/tex/$USER/bin" ] ; then
            PATH="/opt/tex/$USER/bin:$PATH"
        fi
   Remember to use `sudo su` or specify `/root/.bashrc` as the file. Otherwise `sudo nano ~/.bashrc` refers to the user's `.bashrc` file.
