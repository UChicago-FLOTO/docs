# Application Overview

Measurement applications can be deployed on FLOTO for the purposes of collecting data from various edge deployments.

Users can deploy applications to FLOTO using the dashboard, CLI, or [API](./api.md).

There are 3 components at play in this API.

1. *Services* represent a container within your application. Right now, all this means is a reference to some container registry. 
2. *Applications* orchestrate services. This is similar to putting services together into a `docker-compose.yml` file or similar. At this step, services are combined, and environment variables are specified.
3. *Jobs* place applications on devices, and specify a schedule. Jobs can override, or add new environment variables to the application.

Each of these components can be made public, meaning all FLOTO users can see the details, or private, meaning only the creator can see the details. For example, if your application requires environment secrets, these can be added to a private job, while the application kept public. Other users will be able to use the public application in their own private jobs.

# Developing your application

FLOTO is not intended as a development environment. At the very least, we recommend that you develop your docker image on your local machine to verify that it is working as expected before submitting it to the FLOTO portal.

If you wish you test out your application on a Raspberry Pi, we've published [this CHI@Edge notebook](https://chameleoncloud.org/experiment/share/c4a7e4ac-1380-4e8f-9fc9-b2a84750739c), which will allow you to launch and interact with a container in a very similar environment to FLOTO. CHI@Edge has the same container runtime architecture as FLOTO, and so if your container runs on CHI@Edge, it likely will work on FLOTO.

# Using the Dashboard

The [FLOTO dashboard](https://portal.floto.science/) requires you to have an approved account to use. After initial login with your institutional account, please contact an admin.

## Creating and deploying a new application

This section will walk you through deploying an entirely new application, all the way through running it on a FLOTO device.

### Create service(s)

First, navigate to the "Services" page via the navigation buttons at the top of the page. At the bottom there is a form to submit your container reference(s), and whether to make this public. 

The container reference should be what you would use when running `docker pull`. If your container is not hosted on Docker Hub, make sure to include the registry url. If your container registry is private, please contact us to set up secret access.

It is important this container be built for ARM architecture, so that it can run on our devices.

If your service requires any types of peripherals, you can select these from a list when creating your service. Your service will only be able to run on devices with this peripheral type configured.

If your service needs to expose ports on the local network, you can configure this. For each port, select the protocol (TCP/UDP), the port for the device to listen on (limited to 30000-32768), and the port inside your service to forward traffic to. Then click "Add Port"

### Create an application

Next, navigate to the "Application" page. At the bottom, you'll first need to enter a name and a description for you application. By default, your application will be "single-tenant", meaning no other applications can be scheduled on the same device while your application is running. This can be unchecked on this page. Then, click "continue" to move onto the next step. 

Select at least one service for your application to run. On the next step, you'll be asked to define environment variables. These will be set the same in all services, on all devices.

In addition to any environment variables defined here, and later in the job, we will also set the following in your containers:

* `FLOTO_DEVICE_UUID`: The UUID of the device the container is running on.
* `FLOTO_JOB_UUID`: The UUID of the job that created your container.
* `FLOTO_`: Other variables prefixed with `FLOTO_` may appear, depending on the specific device configuration, as defined by the device's operator.

If your application requires secret values, such as S3 secrets, consider leaving them as blank. They can be overrode in a private job.

A shared ephemeral volume will be mounted to `/share` inside each of the services in your application at runtime, per device. The underlying filesystem in this volume is located on the host device, and supports simultaneous read/write from multiple conatiners, sutiable for locking. When your job ends, the data in this path is deleted.

A perisistant volume will be mounted to `/public` inside each of the services in your application at runtime, per device. This directory is shared across all jobs on the device, from all users, not just the ones in your application. The intend use of this volume is a place to store data, which is later consumed by another application, for example a periodic data uploader. The data in this volume is not deleted when your job ends, can be read or overwritten by other applications, due to the multi-tenant environment.

### Create a job

Lastly, navigate to the "Job" page. Select the application you wish to run from the input box.

Next, you'll need to specify the job's schedule. Scroll to the "New timing" section. Currently, the only type of timing on FLOTO is "On-demand," meaning the job will run starting now, until some expiration time. This type of timing is automatically selected. Click on the calendar and clock icons to pick the expiration date. On the right of the click, you'll see the computed duration of this job. Click "Add +" to confirm the timing, and you'll see it appear in the list above.

On the next step, select the devices you wish to run the application on.

Lastly, you may override or add any new environment variables.

Once your job is created, you can inspect it by selecting the "eye" symbol for it in the table at the top of the page. You may need to scroll the table to the right to see this column. 

## Debugging a job

In the job overview page, you'll see all of the information for a job. You'll also a list of events for the job, which can indicate it's status. You can also see on this page the computed timeslots for your job, indicating during what times your application is running per device.
