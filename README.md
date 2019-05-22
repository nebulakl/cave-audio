# Getting audio to work on ASUS Chromebook C302CA

This guide assumes you are using up-to-date ArchLinux

## Preparation
1. On an **up-to-date** ChromeOS, open the terminal and copy the firmware files to somewhere convenient to use. e.g.
   ```bash
   sudo mkdir /mnt/stateful_partition/cros-firmware-75.0.3770.19
   sudo cp -r /lib/firmware/ /mnt/stateful_partition/cros-firmware-75.0.3770.19/
   ```
2. [Download this repo](https://github.com/nebulakl/cave-audio/archive/master.zip) and extract the files

## To get audio working (with pulseaudio)

1. Copy the firmware files from ChromeOS to `/lib/firmware/intel` in ArchLinux. **Make sure the symlinks are pointing to the correct files**
   ```bash
   sudo cp -r cros-firmware-75.0.3770.19/lib/firmware/intel /lib/firmware/intel
   ```
   **Note:** as of ChromeOS 75.0.3770.19, `/lib/firmware/intel/dsp_fw_release.bin` should point to `dsp_fw_release_v969.bin`.

2. Copy `dfw_sst.bin` to `/lib/firmware/`
    ```bash
    sudo cp dfw_sst.bin /lib/firmware/
    ```

3. Copy `Google-Cave-1.0-Cave` to `/usr/share/alsa/ucm/`
   ```bash
   sudo cp -r Google-Cave-1.0-Cave /usr/share/alsa/ucm/
   ```

4. Create a symbolic link for `Google-Cave-1.0-Cave` folder.
    ```bash
    sudo ln -s /usr/share/alsa/ucm/Google-Cave-1.0-Cave/ /usr/share/alsa/ucm/sklnau8825max
    ```

5. Blacklist `snd_hda_intel` module by putting the following in `/etc/modprobe.d/c302ca-audio.conf`
   ```
   blacklist snd_hda_intel
   ```

6. The following steps may reduce popping sound (YMMV)
   
   https://wiki.archlinux.org/index.php/PulseAudio/Troubleshooting#Glitches,_skips_or_crackling
   https://wiki.archlinux.org/index.php/PulseAudio/Troubleshooting#Pops_when_starting_and_stopping_playback

7. Reboot and audio should be working now

**Note:** You may need to repeat step 1 every time the package `linux-firmware` gets updated.

## To switch between the speaker and the headphone automatically with jack detection
The following method is just a **workaround** because it only works with one user. Ideally `pulseaudio` should take care of this. (Need more research on `alsa ucm`)

1. Install the `acpid` package
2. Put the following in `/etc/acpi/events/plugheadphone`
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

3. Enable and start `acpid` service
   ```
   sudo systemctl enable --now acpid
   ```
