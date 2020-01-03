# Running Home Assistant with Docker

Aim:

Getting four Aeotec Z-Wave devices to work, via Z-Stick Gen 5. 

Using Docker.

On Mac, at least at first.

## Steps

### 1. Find out the Z-Stick device name

Attach the device and:

```
$ ls  -1 /dev/cu.usbmodem*
/dev/cu.usbmodem1422401
```

Check that the `docker-compose.yml` file has the right device mapping:

```
    devices:
      - /dev/cu.usbmodem1422401:/dev/zwave
```

```
$ docker-compose up
```

NOTE: 

[Docker FAQ](https://docs.docker.com/docker-for-mac/faqs/#can-i-pass-through-a-usb-device-to-a-container) mentions:

>*"Unfortunately, it is not possible to pass through a USB device (or a serial port) to a container as it requires support at the hypervisor level."*  

That pretty much does it for us. Cannot pass the Aeotec USB dongle to be seen by Home Assistant, within Docker (on Mac).  Likely this works on Linux, but I'd need to get such a box.


## References

- [Install the Aeotec Z-Stick Gen5 with Home Assistant and Docker](https://algorithmdude.com/2019/02/24/install-the-aeotec-z-stick-gen5-with-home-assistant/) (blog, Feb 2019)

- [Home Assistant installation in Mac mini 2014: docker vs direct install](https://www.reddit.com/r/homeassistant/comments/90doci/home_assistant_installation_in_mac_mini_2014/) (Reddit, ~2018-19)
  - identifies Docker on Mac issues with Home Assistant that essentially turn people away

<!-- disabled
- [Can I pass through a USB device to a container?](https://docs.docker.com/docker-for-mac/faqs/#can-i-pass-through-a-usb-device-to-a-container) (Docker FAQ)  
-->
  
