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

