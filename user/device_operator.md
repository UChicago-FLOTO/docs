# Device Operator

Documentation for device operators

Currently, access is done via a root user, with credentials stored on the
floto.science server. To request access to this, contact Mark Powers.

## CLI Access

To login using the balena CLI, `source` the following script.

```
#!/bin/bash
FLOTO_DOMAIN=floto.science
FLOTO_USER=root@floto.science
FLOTO_PASS=$(ssh floto@floto.science cat passwd)
alias floto="BALENARC_BALENA_URL=$FLOTO_DOMAIN balena"

floto login --credentials --email $FLOTO_USER --password $FLOTO_PASS
```

After, you can use the `floto` command as you would normally run `balena`.

## Dashboard Access

The dashboard is hosted [here](https://admin.floto.science:8080/).
To use the dashboard, first extract the password via 
`ssh floto@floto.science cat passwd`.
Then you can login with this password and the username `root@floto.science` to
on the dashboard login page.

## SSH Access to a Device

To gain access to a device first, you need to `tunnel` using the device ID.
This command, will create a tunnel from the
device to the `floto.science` host, and then use SSH to forward this to your
local machine. Here, the port 8029 is arbitrary, and can be changed to anything.
`ssh -L 8029:127.0.0.1:8029 floto@floto.science "floto tunnel $DEVICE_ID --port 22222:8029"`

After running this, you can SSH through the tunnel:
`ssh root@localhost -p 8029`

### Container Access

After getting SSH access to a device you can run `balena ps` to see the
containers on the device. This will show the container name. Then you can run a
command in the container, like you would with `docker exec`. For example:
`balena exec -it $CONTAINER_NAME bash`


