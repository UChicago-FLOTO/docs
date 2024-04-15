# API

FLOTO proves a REST API that may be used. [The API schema can be browsed here](https://portal.floto.science/api/schema/swagger-ui/). This documentation lays out the endpoints, and request and response schema.

The base API endpoint is `https://portal.floto.science/api/`. Your API key can be found on [your profile](https://portal.floto.science/dashboard/user), and copied via the "Copy Key" button. This token should be passed in via an authorization header. 

For example, the following bash command assumes your API key is set to a variable named `FLOTO_API_KEY`, and will retrieve all services public and private that your user can view.
```bash
curl -L -X GET \
    -H "Authorization: Token $FLOTO_API_KEY" \
    https://portal.floto.science/api/services
```

Similarly, your user can create a service by POSTing the service schema JSON.
```bash
curl -v -L \
        -X POST \
        -H "Authorization: Token $FLOTO_API_TOKEN" \
        -H "Content-Type: application/json" \
        -d '{"created_by_project": "ebfdcb54-7fd9-416f-a443-e74489cbd663", "peripheral_schemas": [], "ports": [], "resources": [], "is_public": false, "container_ref": "ubuntu"}' \
        http://localhost:8080/api/services/
```

## Use case: Getting device metadata

[./application_user.md#create-an-application](During application runtime), your service can know the device it is running on via the `FLOTO_DEVICE_UUID` environment variable. If you need more metadata regarding the device, you can get it by using the [device retrieve API endpoint](https://portal.floto.science/api/schema/swagger-ui/#/devices/devices_retrieve). For example, if you want to fetch the approximate geocoordinates of the device, the following command parses this information:

```
curl -sL -X GET \
    https://portal.floto.science/api/devices/e7bcc58048c42bfbbd42b7c85a7ac479 |\
    jq -r '"\(.latitude), \(.longitude)"'
```

For the full schema of the device, see the [API endpoint documentation](https://portal.floto.science/api/schema/swagger-ui/#/devices/devices_retrieve).