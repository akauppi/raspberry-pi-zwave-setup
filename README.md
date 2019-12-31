# Raspberry Pi / Z-Wave Setup

Instructions for setting up RaspberryPi 4 for thermostat control.

These instructions are based on:

- [Install Home Assistant](https://www.home-assistant.io/getting-started/) (Home Assistant docs)

## Required

- Raspberry Pi single board computer
- Z-Wave adapter (Aeotec Z-Stick Gen5)
- Z-Wave peripherals (Aeotec Radiator Thermostat, Aeotec Multisensor 6)
- Micro-SD card[^1][^2][^3] ("32 GB or larger", "A2" application class)

>*⚠️ WARNING: A Raspberry Pi 4 package we bought (Dec 2019) contained an A1, 16 GB Micro-SD instead of the recommended 32 GB, A2. Check the details at ordering!*

[^1]: Read e.g. [this review](https://www.androidcentral.com/best-sd-cards-raspberry-pi-4) for tips on suitable cards. TL;DR Buy Samsung EVO+ 32 GB for ~10 Eur

[^2]: [Home Assistant Getting Started](https://www.home-assistant.io/getting-started/): *"Ideally get one that is Application Class 2 as they handle small I/O much more consistently than cards not optimized to host applications. A 32 GB or bigger card is recommended."*

[^3]: Home Assistant > [Hardware](https://www.home-assistant.io/docs/installation/#hardware) section recommends 1GB RAM, 32 GB storage

Computers for running the setup:

- Computer with SD adapter (macOS used)
- SD / MicroSD adapter
- HDMI display and adapter (any TV will do)
- USB keyboard

Models used:

- RPi 4 (4GB), with enclosure

Scalability: 

RPi 3/3+/4 are mentioned as *"a good starting point, and depending on the amount of devices you integrate this can be enough"* <sub>[source](https://www.home-assistant.io/docs/installation/#performance-expectations)</sub>. 

Z-Wave has a limit of >200 devices / network. Based on the above mention it can be that a Raspberry Pi would have challenges running such? Just need to test and see.


## Downloads

- The right [installation image](https://www.home-assistant.io/hassio/installation/) (Home Assistant)[^3image] 
  - used `hassos_rpi4-3.7.img.gz`
- [balenaEtcher](https://www.balena.io/etcher/); an Electron app (Windows/Linux/macOS) for flashing SD cards

[^3image]: Chose "Raspberry Pi Model B 32bit" since it was marked "recommended".


## Steps

### 1. Flash the card

![](.images/balenaEtcher.png)

### 2. WLAN (optional)

We skip this. If you cannot use Ethernet cable, see the [instructions](https://www.home-assistant.io/hassio/installation/) and [here](https://github.com/home-assistant/hassos/blob/dev/Documentation/network.md#wireless-wpapsk).

### 3. Power on!

Connect the RPi to a display, just to see what's happening.

In any case, [http://hassio.local:8123](http://hassio.local:8123) should soon show this:

>![](.images/hassio-loading.png)

*Note: ONLY use the Raspberry Pi power source for RPi4 (it didn't show any signs of life with a Mac USB-C source, so it's not only about current).*

When ready... you'll see this:

>![](.images/hassio-signup-fin.png)

See [here](https://www.home-assistant.io/docs/authentication/).

Register. 

*Note: It's unclear to Asko (30-Dec-19) how much of this is local, whether some of it is in the cloud.*

When asked for devices, skip it for now. You should see something like this:

>![](.images/hassio-up.png)

---

### Where are we now?

Hass.io is running. It starts at `http://hassio.local:8123` each time we turn the Raspberry Pi on.

We don't:

- have console (command line) access to Linux
- have devices installed

---

This would be a good time to explore the UI in the browser.

### 4. Ssh access

It would be nice to have console access to the Raspberry Pi, from my own Mac. Here's how to do it:

- Hass.io > Add-on Store > (install "SSH Server" plugin)[^5ssh]
- Hass.io > Dashboard > SSH Server

  ![](.images/ssh-plugin.png)

  - Have an SSH key pair available, or [generate one](https://help.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) (GitHub docs)
  - Enter the public key into "Config" section, as instructed on the page.
  - Make sure the plugin is started.

- Hass.io > System > Host system > Reboot
  - Seems to need this, otherwise Web UI gave "502".

- (after reboot) Hass.io > SSH Server > Open Web UI
  - You should get a command prompt
  - Try also from the PC/Mac: `$ ssh root@hassio.local`.

Click `Terminal` and you are able to e.g. check that the Home Assistant configuration is healthy:

![](.images/homeassistant-check.png)

>NOTE: You can now edit Hass.io config remotely. SSH also allows one to map the file system if you wish so (see sshfs); you don't need Samba. However: *"This add-on will not enable you to install packages or do anything as root.”*, i.e. you are SSH'ing to within Home Assistant, NOT within the Raspberry Pi itself.

[^5ssh]: I also tried "SSH & Web Terminal" but ended up using this on.


## Read the manuals

This might be a good time to read the [manuals](https://www.home-assistant.io/docs/). They are really rather good. :)

*...(hours later)...*


## Adding the Z-Stick Gen5

Z-Wave devices need an adapter to be used from a Raspberry Pi. The one we have is an Aeotec [Z-Stick Gen5](https://aeotec.com/z-wave-usb-stick/) USB accessory.

Adding this to Home Assistant:

<!-- disabled (wasn't easy :( )
>*"You do not need to install any software to use Z-Wave."* <sub>[source](https://www.home-assistant.io/docs/z-wave/installation/)</sub>

Yayyy! 😊
-->

1. Add a section to `configuration.yaml`:

  ```
  zwave:
    usb_path: /dev/ttyACM0
    network_key: "0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08, 0x09, 0x0A, 0x0B, 0x0C, 0x0D, 0x0E, 0x0F, 0x10"
  ```

  Note: edit some of the `.network_key` values, to make it unique (secret) to you. **Keep a backup** - if you were to need to start from scratch, you'd need this to communicate with devices if they've been tied with secrecy.
  
  I also created that `.yaml` file so it's there if I plan to use it.
  
2. Attach the Aeotec Z-Stick Gen5 to the Raspberry Pi

3. Hass.io > System > Reboot (not sure if this was needed)

<!-- disabled
You'll see the Stick in Configuration > Integrations:

![](.images/stick-visible.png)
-->

You'll see the Stick in Configuration > Z-Wave:

![](.images/config-zwave.png)

<font color="red" size="+2">Q: Is this normal??? I don't have "add node" or any such buttons, as are implied here:

>![](.images/error-no-add-node-button.png)
</font>

---

This may be relevant:

[SOLVED: Can’t get Aoetec Z-Stick to work with HassOS](https://community.home-assistant.io/t/solved-cant-get-aoetec-z-stick-to-work-with-hassos/141227) (Oct 2019)

>*"Under Configuration>Devices, do you see AEON Labs ZW090 Z-Stick Gen5 US? If you click on it, and click again on the Z-Wave icon, you should see the status card for the device. Toward the top of the card, it should say ready."*

Nothing shows under Configuration > Devices:

>![](.images/configuration-devices-empty.png)

---

This is what it's all about:

- [AEOTEC USB Zwave module not creating /dev/ttyACM0](https://github.com/raspberrypi/linux/issues/3027) (raspberrypi/linux GitHub Issues)
  - [a thread at Raspberry Pi forums](https://www.raspberrypi.org/forums/viewtopic.php?f=28&t=245031#p1502030), linked from the above issue

It's an electric mismatch, looks like the fault would be on Aeotec's side, or just the combination of them and Raspberry Pi 4. 

The only work-arounds currently available (Dec 2019) involve USB 2.0 hubs. That won't do it for me - returning the Z-Stick Gen 5.

---


## Access management

<font color=red>tbd. The intention is to allow the Raspberry Pi to be visible only from within our housing compound network.

tbd. describe the steps</font>


<!-- tbd....
## Console access

Now that Raspberry Pi is running, you can open a terminal there:

```
-->


## Security

>*One major advantage of Home Assistant is that it’s not dependent on cloud services. Even if you’re only using Home Assistant on a local network, you should take steps to secure your instance.* <sub>[source](https://www.home-assistant.io/docs/configuration/securing/)</sub>

Check out [https://www.home-assistant.io/docs/configuration/securing/](https://www.home-assistant.io/docs/configuration/securing/).

### Hidden from the world

The approach taken at Katajaharjuntie 22 (housing compound) is to limit access to the Raspberry Pi to from-within IP's only.

<font color=red>tbd. Describe</font>


## Notes

### Raspberry Pi 4 power

The power supply needs to be "at least 2.5A" (instructions). Raspberry Pi's own is 3.1A. This is probably what you'll get from USB-C power supplies anyhow, but if you intend to run off a USB-A brick with a suitable cable, be aware.

In particular:

>⚠️ *If you are using a Raspberry Pi please remember to ensure you’re using an appropriate power supply with your Pi. Mobile chargers may not be suitable since some were only designed to provide just enough power to the device it was designed for by the manufacturer. Do not try to power the Pi from the USB port on a TV, computer, or similar.*

### Z-Wave mesh nature

>*"Any device that’s permanently powered (not battery powered) will help build the mesh"* <sub>[source](https://www.home-assistant.io/docs/z-wave/)</sub>

This means while Z-Wave is a mesh network, it's not *automatically* that. E.g. thermostats won't relay data - they need to hear the main hub.

The Aeotec Multisensors can be powered over USB. If you can do this, they will likely work as mesh relays.[^6notes]

[^6notes]: Needs to be checked. Our initial pilot is small enough that all devices likely hear the hub?


### Z-Wave Plus

Be aware that the benefits of Z-Wave Plus (see [here](https://www.home-assistant.io/docs/z-wave/devices/#z-wave-plus)) are only available if *all* the devices are Z-Wave Plus.


### Instant Status

>*"As long as your device lists Hail or Association in its Controlled Command Classes, then you’ll get instant status updates."* <sub>[source](https://www.home-assistant.io/docs/z-wave/devices/#instant-status)</sub>

### Aeotec Z-Stick Gen5

Home Assistant [Z-Wave Device Specific Settings](https://www.home-assistant.io/docs/z-wave/device-specific/#aeotec-z-stick) have good stuff e.g. about the Aeotec stick.

#### Disabling the "disco lights" (FAILED!)

Unfortunately, the instructions don't apply when set up with Hass.io (we don't see the actual Linux dev's in the Hass.io terminal).

More instructions [here](echo -e -n "\x01\x08\x00\xF2\x51\x01\x00\x05\x01\x51" > /dev/serial/by-id/usb-0658_0200-if00).

I <strike>did</strike> tried to do it on macOS:

1. Detach the dongle, attach to the Mac
2. Follow instructions:

  ```
  $ echo -e -n "\x01\x08\x00\xF2\x51\x01\x00\x05\x01\x51" | cu -l /dev/cu.usbmodem14532401 -s 115200
cu: creat during lock (/var/spool/uucp/TMP000000c602 in /Users/asko/Git/kht22-rasppie-zwave as uid 501): Permission denied
cu: /dev/cu.usbmodem14532401: Line in use
  ```

  Don't know how to switch the LEDs off - may cover by a tape.


### Pairing by clicking the buttons (Aeotec Z-Stick Gen 5)

>*"Although possibly not recommended as mentioned by firstof9, I have successfully taken my z-stick out and used the ‘standard’ Aeotec pairing method for many devices. Once you plug it back into your HA server you need to do a reboot and then the new devices should come up in the z-wave config panel"* <sub>[source](https://community.home-assistant.io/t/solved-cant-get-aoetec-z-stick-to-work-with-hassos/141227/5)</sub>





