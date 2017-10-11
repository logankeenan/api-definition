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

#### Single Resource
When making a `POST` request a client is creating a single resource and only one single resource. A `POST` is not intended to create multiple resources. A client should make multiple `POST` requests to create multiple entities which share a relationship.

#### Example
We will be creating a product resource to represent skim milk.  Each product has a relationship to a product category. In our case, the skim milk product category can be identified by '/product-categories/445' where 445 is the productCategoryId.  The product category resource must exist before we create the skim milk resource, otherwise, a 400 response would be returned.


##### Sample POST body
```JSON
{
  "name": "Skim Milk",
  "price": "1.59",
  "productCategoryId": "445",
  "description": "The greatest milk in the world!"
}
```

##### Sample Response
Http Status Code: 201
Headers:
```
Location: '/products/1234'
```
