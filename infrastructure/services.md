# Infrastructure services

This document is meant to describe the VMs we use, and how they mange the services we run. 

## staff (staff.floto.science)

Used to SSH to other nodes. As the `ubuntu` user, you can SSH to all other VM's `ubuntu` user. This functions as a deploy node for other services (see the openbalena section).

### openbalena

All services run in a GCP kubernetes cluster This is orchestrated via Helm, under the `/opt/open-balena` directory on `staff.floto.science`.

See [open_balena](open_balena.md) for more details about these services.

## auth (auth.floto.science)

The production authentication server. Services run as containers configured by the `ubuntu` user.

### Services
- `keycloak` - Authentication web server.
- `nginx-proxy` - Configures SSL and exposes keycloak service.

## floto-dashboard (portal.floto.science)

The home-grown dashboard & API for floto. These services run as docker containers, configured by the `ubuntu` user, and exposed via the traefik reverse proxy container

### Services
- `traefik` - reverse proxy container that automatically configures SSL
- `floto_web` - the Django application which runs the our floto dashboard and API.

## k3s-server0{1,2,3}.floto.science

k3s control plane nodes, running behind an AWS load balancer for high availability. The k3s is configured locally on each node.

From staff, we deploy `/opt/undercloud` as a balena application to the k3s cluster.