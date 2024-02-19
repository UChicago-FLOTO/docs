Floto is currently using openBalena for device and application management, which is based on Balena. The openBalena site describes the system at a high level.

The API reference can be found here: https://docs.balena.io/reference/api/overview/

The CLI describes interacting with the system, which perhaps is more useful than the API to see various uses: https://docs.balena.io/reference/balena-cli/

openBalena has some limitations, and does not have 100% parity with the features described in the documentation
- openBalena only has a single root user for authentication. We are working to wrap the relevant portions of the balena API.
- You must build a docker image locally, and then you can deploy it to the fleet. This basically just is pushing the image layers, and then all the devices that are tracking that version automatically upgrade their application.
- There is no GUI for openBalena, we are using a community project.

See design documents [here](https://docs.google.com/document/d/1DJxb6h85yIabu2j3ZOlZpOkw75D49J9qSb1xut9EiDw/edit#heading=h.cb7ogod5jgpc)

# Services

- `api` - Runs the openBalena API
- `db` - The datbase for the API
- `vpn` - The VPN that connects to each device
- `s3` - We use AWS for s3, rather than running our own service.
- `registry` - The registry of fleet application releases, which stores data on s3.
- `haproxy` - The HAProxy instance that the other services run behind.

Cert-manager is used to automatically manage certificates.

# OpenBalena Infrastructure
OpenBalena is running on a kubernetes cluster. This section describes how it was set up and it's usage.

NOTE: For prod, the helm chart and configuration is installed on 
`staff.floto.science` under `/opt/open-balena`.

- Set up a standard kubernetes cluster on GCP.
- Set up kube credentials
    `gcloud container clusters get-credentials floto-prod`
- Clone open-balena fork somewhere that can access the cluster.
    `git clone https://github.com/bartversluijs/open-balena.git`
- Install the helm chart and set up openbalena following this readme
    https://github.com/bartversluijs/open-balena/tree/master/kubernetes/helm
- You'll also need to install cert-manager and then the cert-manager issuer. For prod: 
    after installing https://cert-manager.io/docs/, run
    `kubectl apply -f /opt/open-balena/kubernetes/cert-manager.yaml`

## Migration from docker deployment to k8s
This section describes how migration was done from a docker deployment to a k8s cluster

- migrate s3 data
    - create amazon s3 bucket and set credentials in the kubernetes.yaml file.
        GCP s3 did not work on first try, not sure what is required.
    - Set up rclone with credentials for both s3 endpoints, and copy the data with
    `rclone copy openbalena_s3:registry-data aws_s3:floto-prod` if config is
    ```
    [openbalena_s3]
    type = s3
    provider = Minio
    env_auth = false
    access_key_id = 
    secret_access_key = 
    region = us-east-1
    endpoint = http://s3.floto.science

    [aws_s3]
    type = s3
    provider = AWS
    env_auth = false
    access_key_id = 
    secret_access_key = 
    region = us-east-2
    ```
- migrate psql
    - export prod db
        `docker exec -it openbalena_db_1 pg_dump -U docker resin > prod.sql`
    - copy file to into db pod with kubectl
    - clear init data from db and exec into database
        ```
        $ psql -U docker resin
        drop schema public cascade;
        create schema public;
        ```
    -  Import data `psql -U docker resin -f prod.sql`
- migrate cert/vpn data
    - This is all done during `quickstart` step. The trick is to copy over the old
    cert directory, and then run `quickstart`. It'll skip generating the certs, since
    they already exist, and just insert them into the `kubernetes.yml` file.
    - Make sure to take down the old VPN service, otherwise the devices will not disconnect.
    I had to restart the VPN and API pods once or twice in order for them to eventually reconnect.

# Troubleshooting

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
