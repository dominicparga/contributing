# Raspberry Pi

Although this description seems to be complete, it might be not.
It's main purpose is being a collection of some useful snippets or links.


## Table of Contents <a name="toc"></a>

1. [Handy commands](#handy_commands)
1. [Install Ubuntu Server 20.04](#install-ubuntu-server-20.04)
1. [Setup Internet and update system](#setup_internet_and_update_system)
1. [Setup user-management](#setup-user-management)
1. [Setup ssh (secure shell)](#ssh)
1. [Setup ufw (firewall)](#ufw)
1. [Setup domain and connect it to your server](#setup_domain_and_ddns)
    1. [Setup the DNS-server (via namecheap.com)](#setup_dns-server)
    1. [Setup your router](#setup_router)
1. [Setup TLS](#setup_tls)
1. [Install nextcloud](#install_nextcloud)


I'm using my Raspberry Pi with `Ubuntu Server 20.04` headlessly, so just plain in the cmdline, without any Desktop like `Gnome` installed.
In my experience, `Ubuntu` feels a bit heavy, but `Ubuntu Server 20.04` feels very light and is really nice to use.


## Handy commands <a name="handy_commands"></a>

Some handy commands, e.g. from [linuxhandbook][linuxhandbook/list_users_in_group] or [linuxhandbook][linuxhandbook/has_user_sudo-rights].

```zsh
# view all users
less /etc/passwd

# view all groups of the current user (or the user dominic)
groups
groups dominic

# view all existing groups
less /etc/group

# check if user dominic has sudo-rights
sudo -l -U dominic

# find all users in group sudo
grep '^sudo:.*$' /etc/group | cut -d: -f4
```


## Install Ubuntu Server 20.04 <a name="install-ubuntu-server-20.04"></a>

Installing `Ubuntu Server 20.04` is very simple using the `Raspberry Pi Imager`.
It's just writing the `OS` to the `SD`-card.
After starting the `OS`, login with the default login (probably `ubuntu` as both user and password).

Change the default keyboard-layout system-wide with (from [this forum][superuser/input-keyboard-layout])

```zsh
sudo dpkg-reconfigure keyboard-configuration
```


## Setup Internet and update the system <a name="setup_internet_and_update_system"></a>

Check if `nmcli` (`nm` for network-manager, `cli` for commandline-interface) is installed (for handling Internet via wifi).

```zsh
nmcli --version
```

If it is not installed, connect your Raspberry Pi via LAN with your router (but don't install `nmcli` yet).
In both cases, start your Pi and execute following commands.

```zsh
# first of all, update the system
sudo apt update && sudo apt upgrade
# install nmcli now, if not installed
sudo apt install network-manager

# reboot after upgrading the system
sudo reboot

# now, you can unplug your LAN-connection

# show all wifi-networks
nmcli device wifi list
# and connect to one (--ask is for password)
nmcli --ask device wifi connect 'wifi-name'
# if not all wifi-connections are present, try it with sudo
```


## Setup user-management <a name="setup-user-management"></a>

For more info, see [ubuntu.com][ubuntu/security-users].

The initial user and password are predefined and probably `ubuntu` (both user and password).
Per default, this user `ubuntu` is in one of the groups `sudo`, `wheel` or `admin` (all three are kind of the same thing with different names for backwards-compatibility).

1. Change the initial password of the current user using the following command (`<username>` is probably `ubuntu`).

    ```zsh
    passwd <username>
    ```

    This is important, since the default-user `ubuntu` is allowed to execute sudo-rights (even without being asked for the password).
    Later, when using `ssh` to access the system through your wifi-network (or Internet), this would be a __high risk__!

1. Set a root-password (or change the existing one?)

    ```zsh
    sudo passwd root
    ```

1. Attention: Edit `/etc/sudoers`, but only using `sudo visudo`.
    If you mess this file up (while not using `sudo visudo`), you can't access sudo-commands anymore.
    You can fix this by reinstalling the whole system or by booting with another OS (e.g. via USB-stick) to edit the file from there.
    That's why `sudo visudo` ensures, that your changes are valid and have no errors.
    I would add and/or edit following lines:

    Note: The editor is using `vim`.
    Long story short, move the cursor up/down/left/right with `k`/`j`/`h`/`l`, press `i` for inserting text, press `escape` for moving the cursor again.
    Save and exit by typing `:wq`.
    For more info, see [vim.md][github/self/vim.md].

    My `/etc/sudoers` looks like this:

    ```zsh
    ## Optional and only recommended, if you are familiar with vim.
    ## Otherwise, don't add this line.
    ## It just ensures, that only vim is allowed to open sudoers with visudo,
    ## which helps when you have set ${VISUAL}=code
    Defaults editor=/usr/bin/vim:/usr/bin/vi

    ## User privilege specification
    ## user hostname=(runas-user:runas-group) command
    root ALL=(ALL:ALL) ALL

    ## Members of the admin group may gain root privileges admin is legacy, sudo
    ## is preferred
    %admin ALL=(ALL) ALL

    ## Allow members of group sudo to execute any command (but with password)
    %sudo ALL=(ALL:ALL) ALL

    ## I would delete this line, since this line allows sudo without being asked
    ## for root-password.
    #%sudo ALL=(ALL:ALL) ALL NOPASSWD: ALL

    ##
    ## !!! ATTENTION !!!
    ##
    ## I would highly recommend to delete this line, since a file in
    ## /etc/sudoers.d permits the user ubuntu to execute sudo without any
    ## password. Besides that, any file in there might corrupt your sudoers.
    ## #includedir /etc/sudoers.d
    ```

1. Create a new user for yourself.

    ```zsh
    # create the user dominic (just my name, you might use your name instead)
    #
    # -m
    # Create home-directory for this user
    #
    # -G sudo
    # Add this user to your sudo-group
    #
    # -s /usr/bin/bash
    # Set bash as default shell for this user (otherwise, it might be dash, which is not fully compatible with bash)
    sudo useradd -m -G sudo -s /usr/bin/bash dominic
    sudo passwd dominic
    ```

1. Logout (just `logout`) and login with your new user.
    If everything seems to work fine, you can continue.
    It is important, that you can use `sudo` with this user.
    Otherwise, after removing `ubuntu` in the next step, you have to login as root and fix your `sudo`-permissions, which is not recommended.

    ```zsh
    sudo -l -U dominic
    # or just this, if logged in as dominic
    sudo -l
    ```

1. Remove `ubuntu` from all `sudo`-groups and delete the user.
    You can just execute the commands, even if the groups don't exist.

    ```zsh
    sudo deluser ubuntu admin
    sudo deluser ubuntu sudo
    # Probably not used in Ubuntu Server, but Arch Linux is using wheel
    # If this group doesn't exist, it just prints some text.
    sudo deluser ubuntu wheel

    # delete the user, if you can use sudo with your new, safe user
    sudo userdel ubuntu
    ```

## Setup ssh (secure shell) <a name="ssh"></a>

Background for ssh: [digitalocean][digitalocean/ssh-encryption]

If you don't know `ssh`, it is basically a small service (= program) running on your Raspberry Pi, that allows you to open a terminal through Internet.

> It is important to edit your sudoers-file as mentioned above.
> Otherwise, you might have a default-user with default-password (`ubuntu`), which is able to use `sudo` without root-password.
> Obviously, this is a high security-risk for your server.

Most of this info is from [this blog][devconnected/ssh-server].
It has a quite perfect explanation, although I'm not confirming every code-snippet.
I'm mentioning below, where I disagree with their explanation and choices.

By default, `ssh` should already be installed.
This is the client-version, so you might connect to somewhere else.
What you probably want is connecting to your Raspberry Pi, so `ssh` doesn't help.
You need to install `openssh-server`.

```zsh
sudo apt install openssh-server
```

In `/etc/ssh/sshd_config`, you can configure your `sshd`, which stands for `ssh-daemon`.
A daemon is a service, that is running in the background and waiting for something, e.g. a request through Internet to connect via `ssh` to your Raspberry Pi.

> Per default, every user with a non-empty password is allowed to login (source: [stackexchange][stackexchange/which_users_are_allowed_to_log_in_via_ssh]).
> To change this behaviour, [this blog (ostechnix)][ostechnix/ssh_access_per_user] might help.

Good sources for security are:

- [howtogeek - The Best Ways to Secure Your SSH Server][howtogeek/best_ways_to_secure_ssh]
- [raspberrypi][raspberrypi/security]
- [6 ssh authentication methods to secure connection (sshd_config)][golinuxcloud/ssh-authentication-methods]
- [howtogeek - How to Create and Install SSH Keys From the Linux Shell][howtogeek/ssh-copy-id]

```zsh
# in shell
sudo vim /etc/ssh/sshd_config
```

```zsh
# in /etc/ssh/sshd_config

# Prohibit login as root
PermitRootLogin no

# allow login only by public/private keys, not by password
# (replace dominic by your username)
PasswordAuthentication no
PubkeyAuthentication yes
# and only for specific users
AllowUsers dominic

MaxAuthTries 6


# allow login only for users with non-empty password
# (if passwords are enabled)
PermitEmptyPasswords no

# nope
ChallengeResponseAuthentication no

# you don't need it, so turn it off
X11Forwarding no
# yes so you will be logged out when sudo reboot
UsePAM yes
```

```zsh
# back in shell: apply changes
sudo systemctl restart sshd
```

On your computer, that wants to access via `ssh`, create a `ssh`-key-pair and copy the public(!) key to the server.
Please note, that you have to enable `PasswordAuthentication yes` to copy your public key.

```zsh
# create public/private key-pair
ssh-keygen
# enter info

# copy to server
ssh-copy-id dominic@parga.io
```

> In contrary to [SCHKN][devconnected/ssh-server], I would not change the default-port.
> It doesn't bring you much more security, except for you change it to something quite unusual, which is unhandy and thus might confuse or "break" other software.

To check your daemons on a system, you can use `systemctl`.

```zsh
# check status of your sshd (probably active and already running)
sudo systemctl status sshd

# start sshd at boot
sudo systemctl enable sshd

# start sshd right now
sudo systemctl start sshd

# restart sshd, e.g. after updating the sshd_config
sudo systemctl restart sshd

# stop sshd right now
sudo systemctl stop sshd

# stop sshd to start at boot
sudo systemctl disable sshd
```


## Setup ufw (firewall) <a name="#ufw"></a>

Setup your firewall (ubuntu firewall, called `ufw`), which is basically just a door for incoming and leaving Internet-connections.
A great source of code-snippets is [this blog][digitalocean/ufw_on_ubuntu_20.04].
You define these connections by ports.
The default port for `ssh` is 22, so this port has to be enabled.

```zsh
# shows current firewall rules
sudo ufw status verbose

# default-policy is probably
# - allow all leaving traffic
# - deny all incoming traffic
# but to change it:
sudo ufw default deny incoming

# allow ssh (which stands for port 22 according to /etc/services)
sudo ufw allow ssh
# is equal to
sudo ufw allow 22
# better:
# (limits access when trying to login more than 6 times in 30 seconds)
sudo ufw limit ssh

# for seeing the ports with their respective names, execute
less /etc/services

# for deleting rules, just add 'delete' in between
sudo ufw delete allow ssh

# enable ufw
sudo ufw enable
sudo systemctl restart ufw
```

We're almost ready to connect.
What's missing is the ip-address.
Get it via `nmcli` as follows.
[SCHKN][devconnected/ssh-server] suggests `sudo ifconfig`, which is now deprecated.

```zsh
# show all devices with info, including your ip-address in your wifi-network
nmcli --pretty device show
# in short
nmcli -p d show

# prints something like
# ...
# IP4.ADDRESS[1]       192.168.178.58/24
# ...

# connect from your computer to your Raspberry Pi
# on your computer
ssh dominic@192.168.178.58
```

Ignore the suffix `/24`, which is the number of bits, that refer to your inner network.
Here, 24 bits are 3 bytes, which means that each device in your home-network has the same prefix `192.168.178`.
This also means, that your wifi-settings are probably available at `192.168.178.1` (entering it in a browser-window).
This is just the ip-address inside your network, so this is not unique on global scale and not visible from outside of your wifi-network.
To get your global ip and enable port-forwarding (sometimes called port-permissions), you have to visit your wifi-router-settings.
This is explained below.


## Setup domain and connect it to your Raspberry Pi <a name="setup_domain_and_ddns"></a>

Assume we want to use `domain.com` as our server-domain.
To reach the Raspberry Pi through the Internet, the following chain has to be complete:

1. A user enters `domain.com` into the browser (or you want to use `ssh dominic@domain.com`).

1. A DNS-server finds your routers ip-address using `domain.com`.

1. Your router forwards the request to your Raspberry Pi

1. The Raspberry Pi accepts the request (through `ufw` as already setup above)

In the following, we configure this domain according to my setup on `namecheap.com`.
Your router is changing it's public ip-address on a regular basis (probably daily).
So, to get the DNS-server to find your ip-address without updating the ip-address of `domain.com` at `namecheap.com` everyday, you might use dynamic DNS.

### Setup the DNS-server (via namecheap.com) <a name="setup_dns-server"></a>

1. Buy or rent a domain, e.g. on `namecheap.com`, which has pretty nice documentation and a nice UI.

1. Activate `DynDNS` via toggle-button.
    You will get your Dynamic DNS Password, let it be `asdfasdf`.

1. Add an A Record, which means that your domain is being mapped to an IP4-address.
    Make sure that this A Record has `DynDNS` included.
    On `namecheap.com`, this option is called `A + Dynamic DNS Record`
    - The host is just `@`, which is a special value here, representing your plain domain.
    - The value is your ip-address from your router.
      It doesn't matter, what you enter here, since this will change automatically after setting up your server.
      Just enter `127.0.0.1` (`localhost`).

1. You might want to add a CNAME Record, which maps a provided subdomain to the ip-address of an A Record.
    - e.g. `www.domain.com` or `nextcloud.domain.com` or `something.domain.com` should go directly to `domain.com`, which will be your router.
    - The host is just `www` (or `nextcloud` or `something` respectively).
    - The value is your domain, which should be the destination, so probably `domain.com`.
    - See [namecheap.com][namecheap/create_subdomain] for more info


### Setup your router <a name="setup_router"></a>

1. Visit your router's settings page in your browser (e.g. `fritz.box` or by the network-ip, in my case `192.168.178.1`, as mentioned above).

1. Depending on your router, this might differ.
    Enable port-forwarding (or port-permissions) for your ports.
    For now, this is just ssh, so port 22.

1. ATTENTION: Do not allow self-maintaining port-permissions.
    You don't need it and it's just a possible security-risk.

1. Enable `DynDNS` and your router will inform your domain-provider automatically in case of a ip-change.
    Enter your credentials for your domain-provider (`namecheap.com` in my case).
    Since `namecheap.com` is not pre-configured, you have to use the update-URL, which is provided by `namecheap.com` on some [documentation-website][namecheap/dyndns-update-url].
    The URL will be something like `https://dynamicdns.park-your-domain.com/update?host=@&domain=domain.com&password=asdfasdf` as explained in the following.

    ```text
    https://dynamicdns.park-your-domain.com/update?host=[HOST]&domain=[DOMAIN]&password=[DDNS_PASSWORD]&ip=[IP]

    [HOST]:
    @

    [DOMAIN]:
    domain.com

    [DDNS_PASSWORD]:
    asdfasdf

    [IP]:
    If not provided, the caller of this URL uses its own ip-address.
    You want your router to call this URL, so the ip-address will be the new router's ip.
    ```

You can (hopefully) connect via `ssh dominic@domain.com`.


## Setup Apache and TLS <a name="setup_tls"></a>

`TLS` is for accessing your domain via `https`.
Great sources:

- [How to Get Let's Encrypt SSL on Ubuntu 20.04][serverspace/letsencrypt]
  for creating a certificate and mentioning the `certbot.timer`

- Create the file `/etc/apache2/sites-available/domain.com.conf` and edit `/etc/apache2/ports.conf` according to

  - [digitalocean][digitalocean/apache2-setup] for apache-setup

  - [ubuntu-wiki][ubuntu/wiki/mod_ssl] for `TLS`

  - [upload.com][upload/enhance_encryption] for enhancing encryption (in `/etc/apache2/mods-available/ssl.conf` and `/etc/apache2/sites-available/defaul-ssl.conf`)

  - Test your ssl-encryption via [this ssl-test][ssllabs/ssltest].

- [Apache-Docs: Directives in config-files][apache/docs/core]

- [Apache-Docs: SSL/TLS Strong Encryption: How-To][apache/docs/ssl-tsl_strong_encryption]

- Source: [serverfault-forum][serverfault/forum/which_apache-conf_is_used]

  ```zsh
  # NICE!
  # check, which config is loaded at runtime
  apache2ctl -V | grep SERVER_CONFIG_FILE
  ```

- Nice snippets:

  ```zsh
  # firewall
  sudo ufw allow http
  sudo ufw allow https
  sudo ufw reload

  # install apache2
  sudo apt install apache2
  # install letsencrypt to create a certificate
  sudo apt install letsencrypt

  # enable needed modules in apache2
  # ssl for encryption (https)
  sudo a2enmod ssl
  # for HSTS
  sudo a2enmod headers

  # this timer will renew your certificates automatically
  sudo systemctl status certbot.timer

  # deactivate apache2
  # such that certbot can use port 80 for creating the certificate
  sudo systemctl stop apache2

  # create your certificate
  sudo certbot certonly --standalone -d domain.com
  # or
  sudo certbot certonly --standalone --preferred-challenges http -d domain.com

  # reactivate apache2
  sudo systemctl stop apache2
  ```


## Install nextcloud <a name="install_nextcloud"></a>

In case you want to use `postgres`, look at the following commands to setup the database for nextcloud.
The source is [marksei - How to install NextCloud 20 on Ubuntu 18.04/19.04/19.10/20.04][marksei/install_nextcloud_on_ubuntu], but these commands are same as in the mariadb-related commands in the nextcloud-documentation.

```zsh
# in shell
sudo apt install postgresql
sudo -u postgres psql
```

```sql
# in psql
CREATE DATABASE nextcloud;
CREATE USER nextcloud_user WITH PASSWORD 'YOUR_PASSWORD_HERE';
GRANT ALL PRIVILEGES ON DATABASE nextcloud TO nextcloud_user;
```

After the postgres-setup, I used the official nextcloud's documentation [Example installation on Ubuntu 20.04 LTS][nextcloud/docs/server/example_ubuntu] to install all required packages and downloaded nextcloud.

```zsh
# install all required packages (excluding mariadb/mysql)
sudo apt update && sudo apt upgrade
sudo apt install libapache2-mod-php7.4
sudo apt install php7.4-pgsql
sudo apt install php7.4-gd
sudo apt install php7.4-curl
sudo apt install php7.4-mbstring
sudo apt install php7.4-intl
sudo apt install php7.4-gmp
sudo apt install php7.4-bcmath
sudo apt install php7.4-imagick
sudo apt install php7.4-xml
sudo apt install php7.4-zip
```

```zsh
# cd into Downloads
cd && mkdir -p 'Downloads/tmp' && cd 'Downloads/tmp'

# download nextcloud from https://nextcloud.com/install/
curl 'https://download.nextcloud.com/server/releases/nextcloud-20.0.3.zip' -o 'nextcloud.zip'
curl 'https://download.nextcloud.com/server/releases/nextcloud-20.0.3.zip.sha256' -o 'nextcloud.zip.sha256'

# Verify download
sha256sum -c 'nextcloud.zip.sha25' < 'nextcloud.zip'

# Verify pgp-signature
wget 'https://download.nextcloud.com/server/releases/nextcloud-20.0.3.zip.asc' -O 'nextcloud.zip.asc'
wget 'https://nextcloud.com/nextcloud.asc' -O 'nextcloud.asc'
gpg --import 'nextcloud.asc'
gpg --verify 'nextcloud.zip.asc' 'nextcloud.zip'
```

```zsh
# finalize
unzip 'nextcloud.zip'
sudo mv 'nextcloud' '/var/www/your.domain.com'

# cleanup
cd ..
rm -r 'tmp'
```

Now, you can follow the [Installation on Linux - Apache Web server configuration][nextcloud/docs/server/installation_on_linux/apache_configuration].


```zsh
# setup /etc/apache2/sites-available/your.domain.com

# needed for nextcloud
sudo a2enmod rewrite
sudo a2enmod env
sudo a2enmod dir
sudo a2enmod mime

# finish installation
sudo chown -R www-data:www-data /var/www/your.domain.com

# restart
sudo systemctl restart apache2
```

TODO: https://docs.nextcloud.com/server/20/admin_manual/issues/general_troubleshooting.html#service-discovery-label

[apache/docs/core]: https://httpd.apache.org/docs/current/mod/core.html
[apache/docs/ssl-tsl_strong_encryption]: https://httpd.apache.org/docs/current/ssl/ssl_howto.html
[devconnected/ssh-server]: https://devconnected.com/how-to-install-and-enable-ssh-server-on-ubuntu-20-04/
[digitalocean/apache2-setup]: https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-20-04
[digitalocean/ssh-encryption]: https://www.digitalocean.com/community/tutorials/understanding-the-ssh-encryption-and-connection-process
[digitalocean/ufw_on_ubuntu_20.04]: https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-20-04
[github/self/vim.md]: https://github.com/dominicparga/howto/blob/nightly/vim.md
[golinuxcloud/ssh-authentication-methods]: https://www.golinuxcloud.com/openssh-authentication-methods-sshd-config/
[howtogeek/best_ways_to_secure_ssh]: https://www.howtogeek.com/443156/the-best-ways-to-secure-your-ssh-server/
[howtogeek/ssh-copy-id]: https://www.howtogeek.com/424510/how-to-create-and-install-ssh-keys-from-the-linux-shell/
[linuxhandbook/has_user_sudo-rights]: https://linuxhandbook.com/check-if-user-has-sudo-rights/
[linuxhandbook/list_users_in_group]: https://linuxhandbook.com/list-users-in-group-linux/
[marksei/install_nextcloud_on_ubuntu]: https://www.marksei.com/how-to-install-nextcloud-20-on-ubuntu/
[namecheap/create_subdomain]: https://www.namecheap.com/support/knowledgebase/article.aspx/9776/2237/how-to-create-a-subdomain-for-my-domain/
[namecheap/dyndns-update-url]: https://www.namecheap.com/support/knowledgebase/article.aspx/29/11/how-do-i-use-a-browser-to-dynamically-update-the-hosts-ip/
[nextcloud/docs/server/example_ubuntu]: https://docs.nextcloud.com/server/20/admin_manual/installation/example_ubuntu.html
[nextcloud/docs/server/installation_on_linux]: https://docs.nextcloud.com/server/20/admin_manual/installation/source_installation.html
[nextcloud/docs/server/installation_on_linux/apache_configuration]: https://docs.nextcloud.com/server/20/admin_manual/installation/source_installation.html#apache-configuration-label
[ostechnix/ssh_access_per_user]: https://ostechnix.com/allow-deny-ssh-access-particular-user-group-linux/
[raspberrypi/security]: https://www.raspberrypi.org/documentation/configuration/security.md
[serverfault/forum/which_apache-conf_is_used]: https://serverfault.com/questions/12968/how-to-find-out-which-httpd-conf-apache-is-using-at-runtime
[serverspace/letsencrypt]: https://serverspace.us/support/help/how-to-get-lets-encrypt-ssl-on-ubuntu/
[ssllabs/ssltest]: https://www.ssllabs.com/ssltest
[stackexchange/which_users_are_allowed_to_log_in_via_ssh]: https://unix.stackexchange.com/questions/36804/which-users-are-allowed-to-log-in-via-ssh-by-default
[superuser/input-keyboard-layout]: https://superuser.com/a/404507
[ubuntu/security-users]: https://ubuntu.com/server/docs/security-users
[ubuntu/wiki/mod_ssl]: https://wiki.ubuntuusers.de/Apache/mod_ssl/
[upload/enhance_encryption]: https://upcloud.com/community/tutorials/install-lets-encrypt-apache/
