# Troubleshooting
For troubleshooting, use the user `balena@floto.science`.

## 503 server error
It seems KVM might have issues with long running docker processes, or something
crashed. Things came back up with: `cd ~/open-balena && ./scripts/compose up -d`

# Enrolling devices into both FLOTO and Balena

We may want to be able to access devices from both FLOTO and balena hub. To do this, we need to first configure the devices to support FLOTO so that the root CA is installed to support the VPN.

## Step 1: Configure the device

It is crucial to add the admin ssh key, so that we can access the device OS if
needed for debugging, troubleshooting, or future features. This step must be
done each time to give each device a new API key.

```
BALENARC_BALENA_URL=$FLOTO_DOMAIN balena login --credentials --email $FLOTO_USER --password $FLOTO_PASS

# Version here will depend on $IMAGE
BALENARC_BALENA_URL=$FLOTO_DOMAIN balena config generate \
	--fleet floto \
	--version 2.105.14 \
	--network ethernet \
	--appUpdatePollInterval 10 \
	--output /tmp/floto_config.json

cat /tmp/floto_config.json | jq '.os = {"sshKeys": ["ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDI4sH+RRiGngG9eQUql7nLv9Qa5b5UHSGxARnfMVeiK1UpPOKNAcLiwl6N08sKymYKaY539kto8rxf4k+RIFwfQgFhxOOGJjeeUYmxyp++Gym0iG7iO9dTzJVeTJ+5T24NjRYfrj1OLX2XUqtS78q4q1EUCet7eJv27GPs9LXRep3xIVVToWrlxijp9+DmD752aL8cP7pMshRwcq7dGL+bG+tY8FYVmOQzcQW5l9H627L4QYeawbvB9aq2W6gTENetUFycPtVGRg4+Of/2s3bprUOIKZTkKbNM8Z2m1GYfz1OAD3A865I0n61Xs/qeVCphWoyEKe1XVbRS262SipdF floto@openbalena"]}' > config.json

BALENARC_BALENA_URL=$FLOTO_DOMAIN balena os configure --fleet floto --config-network ethernet --config config.json $IMAGE
```
## Step 2: Flash the image and boot

Flash the image onto the device and then boot the device.

## Step 3: Check for FLOTO connectivity

Check that the new device is online with `floto devices`

> :exclamation: **Note!** You can stop here

## Step 4: Reconfigure the device for Balena hub

Tunnel into the new device over ssh. Run `cp /mnt/boot/config.json
/mnt/boot/floto_config.json`. Download a new config for the balena hub from the
balena dashboard, copy it, and insert it into the command
`os-config join 'COPIED_CONFIG'`. This command will restart all balena services,
and move the device to balena hub.

## Step 5: Reconfigure the device back to FLOTO

Now with the balena dashboard terminal, run `cp /mnt/boot/config.json
/mnt/boot/hub_config.json`. Now copy the contents from 
`/mnt/boot/floto_config.json` into the `os-config join 'COPIED_CONFIG'` command
again to move the device back to FLOTO.

## Step 6: Check for FLOTO connectivity again

Check that the device comes back online and can be tunneled to.
