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

