# Device Operator Overview

Device operators manage the deployment of FLOTO devices. They can use the [FLOTO dashboard](https://portal.floto.science) to do so.

## Enrolling a new device.

Contact us to enroll a new device. We will provide details of how to flash an OS on it.

Once your device is networking, please coordinate with us to assign it to your project to ensure proper authorization is set up for its usage.

### Network traffic
FLOTO devices don't need to accept any inbound connections, but do need to send egress traffic to the following destinations in order for the platform to work.

|Port|Protocol|Destination|Description|
|---|---|---|---|
|443|TCP|{api,auth,k3s-ipv6,portal,registry,s3,tunnel,vpn}.floto.science|HTTPS traffic to FLOTO services|
|3128|TCP|vpn.floto.science|Remote command/control capabilities of device OS|
|6443|TCP|k3s-server{01,02,03}.floto.science|HTTPS traffic to application control plane|
|51820|UDP|k3s-server{01,02,03}.floto.science|VPN connection for application deployment|

Additionally, in order for applications to start your device will need to pull the image from a docker registry. Applications running on your device likely will need to access the internet, for example to upload experiment results to S3. We recommend allowing all traffic to public internet, though if you are the only application user for a device, you can restrict this egress without breaking anything.

## Using the Dashboard

On the dashboard, first navigate to the "Devices" page to see all devices. Select the "wrench" icon next to a device to see its overview. This will let you see various properties about the device, including its MAC and IP addresses. It also shows the device environment variables. There are two additional tabs, the first is the "Command" tab, which lets you run one off commands on the device. The next is the "Logs" tab, which shows the last 1000 lines of logs from the device.

### Configuring peripherals

On the management page for a device, on the right side, you'll see a section for "Peripherals." This allows you to register attached peripherals on your device, and if necessary add further configuration about how the device is installed. This configuration will allow applications to use your device.

To add a device, select "Add new peripheral". In the popup, you can either select an existing peripheral if one has been configured on FLOTO. Otherwise, you can create one from scratch by selecting the peripheral type. This will require you to enter the name of the peripheral (such as Raspberry Pi Camera Module) and a link to documentation for the peripheral. If the peripheral type selected requires further configuration options, you'll be asked to enter them.

Before configuring a peripheral, you'll also see a list of detected peripheral types on your device. By default, you'll most likely see "gpio" as detected. If you have a peripheral plugged in, but you don't see it here, please contact us for help.