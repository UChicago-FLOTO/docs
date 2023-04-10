# Infrastructure nodes:

The AWS instance name is given, and the DNS records that map to it.

## staff (staff.floto.science)

Used to SSH to other nodes. As the `ubuntu` user, you can SSH to all other VM's `ubuntu` user.

## auth (auth.floto.science)

The production authentication server. Services run as containers configured by the `ubuntu` user.

### Services
- `keycloak` - Authentication web server.
- `nginx-proxy` - Configures SSL and exposes keycloak service.

Services are configured

## openbalena ({,api,registry,s3,tunnel,vpn}.floto.science)

The open balena server. All services run as containers congifured by the `balena` user.

### Services
- `api` - Runs the openBalena API
- `db` - The datbase for the API
- `vpn` - The VPN that connects to each device
- `s3` - Object store for balena, mainly to be used by the registry
- `registry` - The registry of fleet application releases
- `haproxy` - The HAProxy instance that the other services run behind.

Use `~/open-balena/scripts/compose [up|down] -d` to start/stop openBalena
services.

To view service logs, run
`~/open-balena/scripts/compose exec $SERVICE journalctl -fn100`
where `$SERVICE` is `api`, `vpn`, `s3`, `registry`.

These services use a wildcard cert that must be manually renewed. To renew:

1. generate new wildcard cert
	`sudo certbot certonly --manual -d '*.floto.science'`
2. generate haproxy pem file with 
```
DIR=/etc/letsencrypt/live/floto.science
cat $DIR/fullchain.pem > open-balena.pem && cat $DIR/privkey.pem >> open-balena.pem
```
3. Copy file into certs volume
	`docker cp open-balena.pem openbalena_cert-provider_1:/certs/open-balena.pem`
4. Restart openbalena haproxy container
    `docker restart openbalena_haproxy_1`

## dashboard ({admin,dashboard,postgrest,remote}.floto.science, )

A.K.A. the [open-balena-admin community project](https://forums.balena.io/t/open-balena-admin-an-admin-interface-for-openbalena/355324). These services run as docker containers, configured by the `ubuntu` user, and are exposed via an nginx systemd service, configured at `/etc/nginx/sites-enabled/`.

Ports are changed from default in `~/open-balena-admin/compose/services.yaml`.
Use `~/open-balena-admin/scripts/compose [up|down] -d` to start/stop services.

### Services
- `open-balena-admin_ui` - Runs a react dev server. This should be udpated to
    mount to an nginx directory.
- `open-balena-admin_postgrest` - A http proxy for the belena DB. Connects directly to the open balena database via an SSH tunnel.
- `open-balena-admin_remote` - Theoretically allows for remote connection to
    devices, but doesn't seem to work.

## floto-dashboard (portal.floto.science)

The home-grown dashboard & API for floto. These services run as docker containers, configured by the `ubuntu` user, and exposed via the traefik reverse proxy container

### Services
- `traefik` - reverse proxy container that automatically configures SSL
- `floto_web` - the Django application which runs the our floto dashboard and API.