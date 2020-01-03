# Access beyond the stage

By default, Hass.io doesn't allow one to see the world outside it's Docker container.

To get a prompt to the HassOS, see SSH access to the host > [HassOS based Hass.io](https://developers.home-assistant.io/docs/en/hassio_debugging.html#hassos-based-hassio):

- prepare the USB stick with `authorized_keys`
- ```
  ssh root@hassio.local -p 22222
  ```

---

This would be nice. My Raspberry Pi is "faceless" (no monitor) and this might allow for seeing things on the console.

It is also required, if I plan to run other Docker-based software on the same hardware.

---

Unfortunately, this fails for me:

```
$ ssh root@hassio.local -p 22222
ssh: connect to host hassio.local port 22222: Connection refused
```

Not sure, why. #help tbd.

