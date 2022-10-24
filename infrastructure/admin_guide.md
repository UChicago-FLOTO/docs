# Services

All services are running from the `balena` user on `floto.science`.

## openBalena services

These services are those that come with openBalena.

Configued in `/home/balena/open-balena/`.These services run in docker
containers.

The services are:
- `api` - Runs the openBalena API
- `db` - The datbase for the API
- `vpn` - The VPN that connects to each device
- `s3` - Object store for balena, mainly to be used by the registry
- `registry` - The registry of fleet application releases
- `haproxy` - The HAProxy instance that the other services run behind.

openBalena runs a `cert-provider` conatiner, that self signs certs.

### Customizations

The only differences between our services and the upstream (openBalena) services
are changes in the `~/open-balena/compose/services.yaml` compose file to get
around KVM docker build issues (i.e. set to build with `network=host`).

### Important scripts

Use `~/open-balena/scripts/compose [up|down] -d` to start/stop openBalena
services.

To view service logs, run
`~/open-balena/scripts/compose exec $SERVICE journalctl -fn100`
where `$SERVICE` is `api`, `vpn`, `s3`, `registry`.

## open-balena-dashboard services

These services are a collection of [community
projects](https://github.com/dcaputo-harmoni/open-balena-admin/) that run a
dashboard for openBalena.

These services are
- `open-balena-admin_ui` - Runs a react dev server. This should be udpated to
    mount to an nginx directory.
- `open-balena-admin_postgrest` - A http proxy for the belena DB. This should be
    updated so there is no reason to expose it. Instead, we should be using the
    balena api.
- `open-balena-admin_remote` - Theoretically allows for remote connection to
    devices, but doesn't seem to work.

These are exposed via `nginx` reverse proxy set up under
`/etc/nginx/sites-enabled/` configs, using a Let's Encrypt wildcard cert.

### Customizations

Ports are changed from default in `~/open-balena-admin/compose/services.yaml`.

### Important scripts

Use `~/open-balena-admin/scripts/compose [up|down] -d` to start/stop services.

# Troubleshooting
For troubleshooting, use the user `balena@floto.science`.

## 503 server error
It seems KVM might have issues with long running docker processes, or something
crashed. Things came back up with: `cd ~/open-balena && ./scripts/compose up -d`
