# Install Ubuntu Server
## Create Bootable Install Media
- download Ubuntu Server from [CANONICAL Web site](https://ubuntu.com/download/server "Download Ubuntu Server").
    - Make sure to verify the download by using `shasum`.
- You need to convert `iso` file before putting into USB media. Although [Official tutorial]("https://tutorials.ubuntu.com/tutorial/tutorial-create-a-usb-stick-on-macos" "Create a bootable USB stick on macOS") says install and run a 3rd party application, Etcher, to burn image to USB media, installing a 3rd party application for one specific purpose is an unpleasant method and actually unnecessary. Just do it like below
    - Run the command below to convert `iso` into `img`
    ```bash
    $ hdiutil convert -format UDRW -o ubuntu-19.04-live-server-amd64 ubuntu-19.04-live-server-amd64.iso
    ```

    - Erase USB media.
    ```bash
    # list volumes
    $ diskutil list

    # format volume
    $ sudo diskutil eraseDisk FAT32 UBUNTU GPT /dev/diskn
    ```

    - Write image to USB media.
    ```bash
    $ sudo diskutil unmount /dev/diskn
    $ sudo dd if=/path/to/ubuntu-19.04-live-server-amd64.img of=/dev/diskn bs=1M
    ```
- Done!

## Configuration
- Set timezone

```bash
# get current setting
$ cat /etc/timezones
$ timedatactl
# get available timezones
$ timedatectl list-timezones
# set timezone
$ sudo timedatactl set-timezone `[Timezone]`
```

- Install neovim
```bash
$ sudo add-apt-repository ppa:neovim-ppa/unstable
$ sudo apt-get update
$ sudo apt-get install -y neovim
# Prerequisites for the Python modules
$ sudo apt-get install -y python-dev python-pip python3-dev python3-pip
# set editor
$ sudo update-alternatives --install /usr/bin/vi vi /usr/bin/nvim 60
$ sudo update-alternatives --config vi
$ sudo update-alternatives --install /usr/bin/vim vim /usr/bin/nvim 60
$ sudo update-alternatives --config vim
$ sudo update-alternatives --install /usr/bin/editor editor /usr/bin/nvim 60
$ sudo update-alternatives --config editor
```

## Security
### SSH setting
- Generate the SSH key
    - On client
    <pre>
    $ ssh-keygen -t ed25519 -o -a 100
    Generating public/private ed25519 key pair.
    Enter file in which to save the key ($HOME/.ssh/id_ed25519): $HOME/.ssh/id_ed25519_foo
    Enter passphrase (empty for no passphrase):
    Enter same passphrase again: [<b>DO NOT USE EMPTY PASSPHRASE</b>]
    Your identification has been saved in $HOME/.ssh/id_ed25519_foo.
    Your public key has been saved in $HOME/.ssh/id_ed25519_foo.pub.
    The key fingerprint is:
    SHA256: ***
    The key's randomart image is:
    +--[ED25519 256]--+
    |                 |
    |                 |
    |                 |
    |                 |
    |                 |
    |                 |
    |                 |
    |                 |
    |                 |
    +----[SHA256]-----+
    $ cat ~/.ssh/id_ed25519_foo.pub | pbcopy # or use ssh-copy-id
    </pre>
    - Store the public key on server
    ```bash
    $ touch ~/.ssh/authorized_keys
    $ chmod 600 ~/.ssh/authorized_keys
    ```

- `ssh-agent`

If you have to use bastion servers to connect your server like a figure below. Don't put ssh key on the bastion server. By using `ssh-agent`, you can use key authentication with your server without telling the bastion server your private key.

```
    ,--------.  SSH  ,---------.  SSH  ,-------------.
    | client | ----- | bastion | ----- | your server |
    `--------'       `---------'       `-------------'
```

```bash
# On macOS
$ eval $(ssh-agent) # put this line into the shell profile
$ ssh-add -K ~/.ssh/id_ed25519_foo
$ cat ~/.ssh/config

# macOS specific configuration
Host *
    UseKeychain yes
    AddKeysToAgent yes
    IdentityFile ~/.ssh/id_rsa
...
Host servername
    HostName ip
    User user
    Port num
    IdentityFile ~/.ssh/id_ed25519_foo
    ForwardAgent yes
...

$ ssh -A <user@bastion>
# On the bastion server
$ cat ~/.ssh/config

...
Host servername
    HostName ip
    User user
    Port num
    ForwardAgent yes
# You don't need to put a private key on bastion server!
...

$ ssh servername
```

- Change SSH port from 22
    - edit `/etc/ssh/sshd_config`. Uncomment `port 22` and select random number from 49152-65535 which is in dynamic, private or ephemeral ports.
    ```bash
    # restart ssh
    $ sudo systemctl restart ssh
    ```

- Set firewall
    - `ufw`
    ```bash
    $ sudo ufw status verbose
    $ sudo ufw default DENY
    $ sudo ufw allow [port number]/tcp comment [comment]
    ```

