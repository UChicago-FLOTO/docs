# Operator access

Documentation for FLOTO operators on how to access parts of the infrastructure.

Currently, access is done via a root user, with credentials stored on the
staff.floto.science server. To request access to this, contact Mark Powers.

## CLI Access

To login using the balena CLI, `source` the following script.

```
#!/bin/bash
FLOTO_DOMAIN=floto.science
FLOTO_USER=root@floto.science
FLOTO_PASS=$(ssh ubuntu@staff.floto.science cat passwd)
alias floto="BALENARC_BALENA_URL=$FLOTO_DOMAIN balena"

floto login --credentials --email $FLOTO_USER --password $FLOTO_PASS
```

After, you can use the `floto` command as you would normally run `balena`.
If you are sshed into `floto.science`, the `floto` alias will be set up
automatically.

## Dashboard Access

The dashboard is hosted [here](https://admin.floto.science/).
To use the dashboard, first extract the password via 
`ssh floto@staff.floto.science cat passwd`.
Then you can login with this password and the username `root@floto.science` to
on the dashboard login page.

## SSH Access to a Device

To gain access to a device first, you need to `tunnel` using the device ID.
First ssh to the floto.science host, which has an admin key for the devices.

Then create the ssh tunnel by running `~/tunnel.sh`. This will tunnel the device
ssh port (22222) to the local port 8029.

After running this, you can SSH through the tunnel using the admin key:
`ssh -i ~/.ssh/floto_admin root@localhost -p 8029`

The port 8029 is arbitrary here, any unused port will work. The `tunnel.sh`
script uses 8029 by default.

### Container Access

After getting SSH access to a device you can run `balena ps` to see the
containers on the device. This will show the container name. Then you can run a
command in the container, like you would with `docker exec`. For example:
`balena exec -it $CONTAINER_NAME bash`


