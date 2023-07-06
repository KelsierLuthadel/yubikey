# YubiKey Linux configuration

## Install required udev software
- Verify that `libu2f-udev` has been installed.

    On Debian systems (Ubuntu, Linux Mint, etc.) check to see if `libu2f-udev` has been installed:

    `dpkg -s libu2f-udev`

    `libu2f-dev` can be installed using:

    `sudo apt install libu2f-udev`

## Create udev rules
Download the YubiCo rules from: https://github.com/Yubico/libfido2/blob/main/udev/70-u2f.rules

Copy `70-u2f.rules` to `/etc/udev/rules.d/`

Reboot the system

## Install u2f PAM
This module implements PAM over U2F and FIDO2, providing an easy way to integrate the YubiKey (or other U2F/FIDO2 compliant authenticators) into your existing infrastructure.

`sudo apt-get install libpam-u2f`

## Integrate U2F Key(s) with your account
Open a terminal

run `mkdir -p ~/.config/Yubico`

Insert a U2F key

run `pamu2fcfg > ~/.config/Yubico/u2f_keys`

  You may be prompted to enter a PIN, if so, this is your YubiKey's FIDO2 PIN.
  
Touch the YubiKey to confirm

### Backup keys
If you have a backup device, follow the steps below:

Insert a U2F key

run `pamu2fcfg >> ~/.config/Yubico/u2f_keys`

  You may be prompted to enter a PIN, if so, this is your YubiKey's FIDO2 PIN.
  
Touch the YubiKey to confirm

## Move the config to a secure location
You will want to move the keys to a more secure location than your home directory, to do so run the following commands:

`sudo mv ~/.config/Yubico /etc`

`sudo chown root:root /etc/Yubico/u2f_keys`

`sudo chmod 660 /etc/Yubico/u2f_keys`

## Test sudo 
To make sure that you do not get locked out of your account, you will want to keep terminal windows open.

Edit `/etc/pam.d/sudo` by running `sudo nano /etc/pam.d/sudo`

Immediately below after the “@include common-auth” line, add the following line:

`auth       required   pam_u2f.so authfile=/etc/Yubico/u2f_keys`

Press **CTRL+O** and **ENTER**

Do not close the terminal, or exit from the editor

Remove the YubiKey

In another terminal type `sudo whoami`

Even when the correct password is entered, this will fail as there is no YubiKey inserted

Insert the YubiKey

Type `sudo whoami` and enter the password.

When prompted, touch the YubiKey to confirm#

If all went well, the sudo command will work

## Configure the system for graphical login
If you are using GDM as your login manager edit `/etc/pam.d/gdm-password` as sudo

Immediately below after the “@include common-auth” line, add the following line:

auth       required   pam_u2f.so authfile=/etc/Yubico/u2f_keys`

Press **CTRL+O** and **ENTER**

## Configure the system for TTY login
Edit `/etc/pam.d/login` as sudo

Immediately below after the “@include common-auth” line, add the following line:

`auth       required   pam_u2f.so authfile=/etc/Yubico/u2f_keys`

Press **CTRL+O** and **ENTER**

## SSH Keys
SSH keys can be protected (when using Git for example):

run `ssh-keygen -t ecdsa-sk -O verify-required`

This will write two files into `~/.ssh/` called `id_ecdsa_sk_rk` and `id_ecdsa_sk_rk.pub`

The public key can be uploaded to GitHub / GitLab, so that when you perform a git action over ssh you will be prompted to touch the YubiKey.

## Protecting an ssh server
On the server you will need to install `libpam-yubico`

run: `sudo apt-get install libpam-yubico`

### Save Key IDs for users
Now create a file that will contain the YubiKey IDs

`touch /etc/ssh/authorized_yubikeys`

For each user on the system that will be protected with a YubiKey, you will add the first 12 characters of the key. You can get the ID by opening a text editor and touching the button on the YubiKey, and selecting the first 12 characters.

`sudo nano /etc/ssh/authorized_yubikeys`

```
user1:aahuseafandz
user2:ahgybnjiijha:protocbhgtya:duckfghuythr
user3:fnyynyhjklea:gnhghtyewsda
```

In the example above, user1 has only 1 key, user2 has 3 keys and user3 has 2 keys.

### Create an API Key
You will now need an API key from Yubico, to get one go to: https://upgrade.yubico.com/getapikey/ and follow the prompts to get an ID and secret.

Edit `/etc/pam.d/sshd` to add a PAM authorization, and add `auth required pam_yubico.so id=client_id key=secret_key authfile=/etc/ssh/authorized_yubikeys` to the top of the file (replacing `client_id` and `client_secret` with the ID and secret you created earlier.

`nano /etc/pam.d/sshd`

```
# PAM configuration for the Secure Shell service

auth required pam_yubico.so id=client id key=secret key authfile=/etc/ssh/authorized_yubikeys

# Standard Un*x authentication.
@include common-auth
```

### Updating sshd
Edit `/etc/ssh/sshd_config` and ensure the following lines exist

```
ChallengeResponseAuthentication yes
UsePAM yes
```

Restart the ssh daemon to allow secure login

`sudo systemctl restart sshd`




