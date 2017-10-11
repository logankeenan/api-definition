# api-definition

### Create

#### HTTP Methods
The http method associated with create is `POST`.

 `POST` is neither safe or idempotent because making two identical `POST` requests will result in two resources being created with identical content.  

The result of a `POST` request is a http response with a header where the Location is the resource identifier of the newly created resource. Example: `Location: '/products/123'`

#### Status Codes
[201 (CREATED)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/201): The request has succeeded, the resource will be created before a response is returned, and the response will contain a Location header containing an identifier for the created resource
[400 (Bad request)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/400): The request could not be understood by the server due to an invalid syntax.  A bad request could be caused by invalid state, invalid domain validation errors, missing data, etc.
[401 (Unauthorized)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/401): The client is not authenticated -- the resource **may** potentially be created at this resource identifier, but they must authenticate first.
[403 (Forbidden)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/403): The client is not authorized to create this type of resource.
[404 (Not found)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/404): The resource cannot be created at this identifier.
