# FLOTO Docs

Floto is currently using openBalena for device and application management, which is based on Balena. The openBalena site describes the system at a high level.

The API reference can be found here: https://docs.balena.io/reference/api/overview/

The CLI describes interacting with the system, which perhaps is more useful than the API to see various uses: https://docs.balena.io/reference/balena-cli/

openBalena has some limitations, and does not have 100% parity with the features described in the documentation
- openBalena only has a single root user for authentication. We are working to wrap the relevant portions of the balena API.
- You must build a docker image locally, and then you can deploy it to the fleet. This basically just is pushing the image layers, and then all the devices that are tracking that version automatically upgrade their application.
- There is no GUI for openBalena, we are using a community project.

See design documents [here](https://docs.google.com/document/d/1DJxb6h85yIabu2j3ZOlZpOkw75D49J9qSb1xut9EiDw/edit#heading=h.cb7ogod5jgpc)