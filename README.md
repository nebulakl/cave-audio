**Update**

Broken as of kernel 5.0.10. Old kernel seems to be fine.

---

# cave-internal-audio
Getting audio to work on ASUS Chromebook C302CA

This guide assumes you are using up-to-date Archlinux

## To get audio working (with pulseaudio)
1. [Download this repo](https://github.com/nebulakl/cave-internal-audio/archive/master.zip) and extract the files
2. move `dfw_sst.bin` to `/lib/firmware/`
    ```bash
    sudo mv dfw_sst.bin /lib/firmware/
    ```
3. move `Google-Cave-1.0-Cave` to `/usr/share/alsa/ucm/`
   ```bash
   sudo mv Google-Cave-1.0-Cave /usr/share/alsa/ucm/
   ```
4. create a symbolic link
    ```bash
    sudo ln -s /usr/share/alsa/ucm/Google-Cave-1.0-Cave/ /usr/share/alsa/ucm/sklnau8825max
    ```
5. blacklist `snd_hda_intel` module by putting the following in `/etc/modprobe.d/c302ca-audio.conf`
   ```
   blacklist snd_hda_intel
   ```
6. reboot and audio should be working now

## To switch between the speaker and the headphone automatically with jack detection
The following method is just a **workaround** because it only works with one user. Ideally `pulseaudio` should take care of this. (Need more research on `alsa ucm`)

1. install the `acpid` package
2. put the following in `/etc/acpi/events/plugheadphone`
   ```
   event=jack/headphone HEADPHONE plug
   action=sudo -u YOUR_USER_NAME XDG_RUNTIME_DIR=/run/user/YOUR_USER_ID pactl set-card-profile 0 Headphone
   ```
   and put the following in `/etc/acpi/events/unplugheadphone`
   ```
   event=jack/headphone HEADPHONE unplug
   action=sudo -u YOUR_USER_NAME XDG_RUNTIME_DIR=/run/user/YOUR_USER_ID pactl set-card-profile 0 Speaker
   ```
   replace `YOUR_USER_NAME` and `YOUR_USER_ID` accordingly.
3. enable and start `acpid` service
   ```
   sudo systemctl enable --now acpid
   ```
