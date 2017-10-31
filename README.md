# api-definition

## CRUD

### Create

#### HTTP Methods
The http method associated with create is `POST`.

 `POST` is neither safe or idempotent because making two identical `POST` requests will result in two resources being created with identical content.  

The result of a `POST` request is a http response with a header where the Location is the resource identifier of the newly created resource. Example: `Location: '/products/123'`

#### Status Codes
- [400 (Bad request)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/400): The request could not be understood by the server due to an invalid syntax.  A bad request could be caused by invalid state, invalid domain validation errors, missing data, etc.
- [401 (Unauthorized)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/401): The client is not authenticated -- the resource **may** potentially be created at this resource identifier, but they must authenticate first.
- [403 (Forbidden)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/403): The client is not authorized to create this type of resource.
- [404 (Not found)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/404): The resource cannot be created at this identifier.
- [409 (Conflict)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/409): The resource cannot be created due to a conflict with current state of the server.


#### Single Resource
When making a `POST` request a client is creating a single resource and only one single resource. A `POST` is not intended to create multiple resources. A client should make multiple `POST` requests to create multiple entities which share a relationship.

#### Example
We will be creating a product resource to represent skim milk.  Each product has a relationship to a product category. In our case, the dairy category can be identified by `/product-categories/445` where 445 is the productCategoryId.  The product category resource must exist before we create the skim milk resource, otherwise, a 400 response would be returned.


##### Sample POST body
```JSON
{
  "name": "Skim Milk",
  "price": 1.59,
  "productCategoryId": 445,
  "description": "The greatest milk in the world!"
}
```

##### Sample Response
Http Status Code: 201
Headers:
```
Location: 'https://api.hy-vee.com/products/1234'
```

#### Validations Errors (400 Status Code)
When a 400 status code, bad request, response is received it should contain a body which contains a human readable description of what went know.  Validation errors typically come in two different types. Validation errors that related to particular fields in the resource that was posted and errors that relate to the entire resource.  Errors related to particular field might include an issue with the type, the length of a field, a non-existent id for a relationship, etc. Errors related to the entire resource might be specific to business rules or invalid statue due to a combination of properties.

##### Sample 400 Response
In this example a resource the client attempted to create a credit card resource but the credit card could not be verified (being ambiguous on purpose is important here) and the the nickname property was not provided

```json
{
  "errors": ["The credit card failed verification"],
  "propertyErrors": {
    "nickname": ["Nickname is a required field"]
  }
}
```

### Read

#### HTTP Methods
The HTTP verb associated with reading data is `GET`. A `GET` request is safe, idempotent, and cacheable.

Safe means that a `GET` request does not affect the state of the server. If we draw an analogy to reading a book, this makes a lot of sense -- the book doesn't change just because you read a page. A `GET` is also idempotent, which means that it can be performed many times in a row and the client will always get the same result -- you can read the same page in a book as many times as you like. However, if another client updates the resource you're requesting, you may get a different result between requests. Because nothing should change on either the server or the resource as the result of performing a `GET`, the client can store (cache) the result.

The result of a `GET` is some representation of a resource, usually JSON. You can read more about [`GET` on MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/GET)

#### Status Codes
There are several status codes involved with reading information from an API:

- [200 (OK)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/200): The resource exists and the client is authorized to view it. The response will include the requested resource.
- [400 (Bad request)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/400): The client's request was invalid (e.g., the URL was malformed).
- [401 (Unauthorized)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/401): The client is not authenticated -- the resource **may** exist and the client **may** have access to it, but they must authenticate first.
- [403 (Forbidden)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/403): The resource may exist, but the client is not authorized to view it. If the existence of the resource is sensitive, then the API can return 404 instead.
- [404 (Not found)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/404): The resource does not exist.

#### Single Item
If we want to identify a product with an `id` of 123, the URI might look like this: `/products/123`. We prefix the URI with `products` so that it is clear which resource we're talking about, as an API will likely have many different resources. Also, we like to optimize URIs for human readability (machines don't care), so we use `products` rather than a numeric id or UUID. It's also important that the URI is plural, since we are identifying a single product in a collection of products.

##### Sample Response
Here's what a request to `GET /products/123` might return as the body:
```json
    {
      "category": 42,
      "id": 123,
      "name": "Hy-Vee Authentic Salsa (Medium)",
      "size": {
        "volume": {
          "value": 16,
          "uom": "oz"
        },
        "quantity": {
          "value": 1,
          "uom": "eaches"
        }
      },
      "upc": "089463115732"
    }
```
Since this was a successful request, the response's status code is `200`.

#### Collections
If we want all the members of a collection, we use a URI like this: `/products`. As in the case for a single item, the URI is plural since we are interacting with a collection.

##### Pagination
If there are many items in the collection, the API should split up the response into pages.

##### Filtering
When accessing a collection, a client might want to filter by properties on the resource. The RESTful way to do this is to use a query string made up of individual key-value parameters, which is appended to the URI for the resource collection.

To filter `products` by a category id, the URI might look like this: `/products?categoryId=1`. Multiple query parameters are separated by an ampersand, so to filter products by both category and color, the URI is `/products?categoryId=1&colorId=2`. If we want to filter by multiple values for the same property, we just include the query parameter as many times as necessary (ex: `/products?colorId=1&colorId=2`).

##### Subset
You may have a requirement that clients be allowed to specify a subset of a collection when no filtering meets their needs.
For example, clients may want to interact with a small collection of ids, using a URI like this: `/api/resource/1,2,3`. However, this is not very RESTful -- the URI is not resource-based. REST resources should always have a URI for a collection and a URI for accessing a single item in that collection, which is sufficient for the client's needs. Remember, the API should be fast, such that the client could make separate requests for each item, or simply grab the entire list and filter.

##### Sample Response
A `GET` to `/products` might return this paginated response:
```json
{
    "products": [
      {
        "category": 42,
        "id": 123,
        "name": "Hy-Vee Authentic Salsa (Medium)",
        "size": {
          "volume": {
            "value": 16,
            "uom": "oz"
          },
          "quantity": {
            "value": 1,
            "uom": "eaches"
          }
        },
        "upc": "089463115732"
      },
      {
        "category": 89,
        "id": 456,
        "name": "Hy-Vee Mountain Lightning",
        "size": {
          "volume": {
            "value": 12,
            "uom": "oz"
          },
          "quantity": {
            "value": 1,
            "uom": "casepack"
          }
        },
        "upc": "126908453187"
      }
    ]
}
```
In this response, we can see that `products` contains the array of products. We could've returned just the array, but this approach will allow us to extend the response object in a backwards-compatible way. Because this was a successful request, the response's status code is `200`.

### Update

#### HTTP Methods
The HTTP verb associated with updating a resource is `PUT`. A `PUT` request is not safe, but implemented correctly, it is idempotent. You can read more about [`PUT` on MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/PUT)

#### Status Codes
There are several status codes involved with updating information:
- [202 (Accepted)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/204): The request to update the resource has been receieved, but the update itself has not yet occurred. Usually returned when the update is expensive and processing may take longer than a client cares to wait.  
- [204 (No content)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/204): The resource was successfully updated and the response has no body.
- [400 (Bad request)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/400): The client's request was invalid. The request may be malformed (e.g., invalid JSON) or some validation on the payload may have failed.
- [401 (Unauthorized)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/401): The client is not authenticated, but **may** be able to update the resource if they login.
- [403 (Forbidden)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/403): The resource exists, but the client is not authorized to update it. Depending on how sensitive the resource is, it might be appropriate to return a `404` instead.
- [404 (Not found)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/404): The resource does not exist.
- [405 (Method not allowed)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/405) The resource does not support a PUT (perhaps it is a collection or a static item)
- [409 (Conflict)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/409):
    - multiple clients attempting to update the same resource simultaneously
    - conflicting resource id between the request URI and the payload. 

#### Single Item
If we want to update a product with an id `123`, the URI is simply `/products/123`. We may opt to include the resource id in the request payload, so it should also be checked against the id in the URI.

##### Sample Request
If we want to update the name of a product with id `123`, from "Hy-Vee Authentic Salsa (Medium)" to "Hy-Vee Salsa", we can use a payload like this: 
```json
  {
    "category": 42,
    "id": 123,
    "name": "Hy-Vee Salsa",
    "size": {
      "volume": {
        "value": 16,
        "uom": "oz"
      },
      "quantity": {
        "value": 1,
        "uom": "eaches"
      }
    },
    "upc": "089463115732"
  }
```
Note that all the fields that would be present from a `GET /products/123` are there, but only a single field, `name`, changes. This approach allows clients to fetch a resource, make their change, and call the API to update, without having to do any additional transformation. When designing validation for `PUT` endpoints, keep this in mind.

##### Sample Response
- status code `204`
- no response body

##### Validation
Most likely an API will do some validation against a request to enforce any technical restraints or business logic. If the request fails validation, a `400` status code will be returned, along with details about the errors.

As in a create or `POST`, there will be validation against the payload as a whole (general errors) as well as individual fields (property errors). In the response, general errors should be placed under `errors`, while `propertyErrors` should be an object with keys corresponding to each field with an error (note that there's no need to indicate that a field passed validation, only include errors).

For example, say we have a `/sales` endpoint that represents a sale. More than one sale cannot happen at a time and they must be tied to an individual location. If we were to update a sale to overlap with another and we omitted the location using a request like the following:

`PUT /sales/61801`
```json
{
    "beginDate": "11/01/2017",
    "endDate": "2017-12-01",
    "locationId": null,
    "name": "foo"
}
```

Then the API should respond with:
```json
{
  "errors": ["Multiple sales cannot occur in the same period"],
  "propertyErrors": {
    "beginDate": ["beginDate must be ISO-8601 formatted"],
    "locationId": ["locationId is a required field"],
    "name": ["name must be at least 10 characters long"]
  }
}
```

##### Lengthy Updates

Some resources may take additional processing that exceeds the typical timeout period for a request. Rather than having the client wait for the result, the API should return a `202` response code and a `Location` header that indicates where the status of the request can be monitored. 

From an implementation perspective, we may have a `/tasks` resource that allows clients to monitor long-running tasks, including updates. For example, a `PUT` to `/products/123` would return `202 Accepted` along with these headers:
```json
Location: 'https://api.hy-vee.com/tasks/789'
Expires: Wed, 21 Oct 2017 07:28:00 GMT
```

The headers indicate to the client that the task to update the resource can be monitored at `/queue/789` and that the task will expire at some date in the future, regardless of success or failure.

However, this approach should be a **last** resort; in almost all cases, persistence to a datastore is fast, so resources that require long-running updates should be rare. If you find yourself implementing or reusing a `task` resource for updates, it's a good idea to step back, profile requests, and see what can be done to speed up the API.

#### Collection
Entire collections should not be updated with a `PUT`. Instead, individual items should be updated. Should a client attempt to update a collection, the appropriate response code is a `405`. 

### Delete

#### HTTP Methods
The `DELETE` HTTP verb is used to dipose of resources. A `DELETE` changes the state on the server, either by removing the resource entirely (a hard delete) or by marking the resource as deleted and not exposing it to the client (a soft delete). Because the state of the server is altered, `DELETE` is not safe. If it's properly implemented on the server, a `DELETE`
is idempotent. Note that the status code from subsequent `DELETE`s might differ -- the first will return with a `204` and additional calls will return with `404`. You can read more about [`DELETE` on MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/DELETE).

#### Status Codes
There are several status codes involved with deleting a resource:

- [202 (Accepted)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/202): The request to delete the resource was accepted, but has not yet been processed. The deletion may fail or be denied later. 
- [204 (No content)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/204): The resource was successfully deleted.
- [401 (Unauthorized)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/401): The client is not authenticated -- the resource **may** exist and the client **may** be able to delete it, but they must authenticate first.
- [403 (Forbidden)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/403): The resource may exist, but the client is not authorized to delete it. If the existence of the resource is sensitive, then the API can return 404 instead.
- [404 (Not found)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/404): The resource does not exist. May be returned if the client queries for the resource after deleting it.
- [405 (Method not allowed)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/405): The resource does not support deletion -- perhaps it is a collection or a static item.

#### Individual items
If we want to delete the product with an `id` of 123, we can simply `DELETE /products/123`. The API will respond with a `204` status code and need not include any headers. If we try to `DELETE /products/123` again, the API will respond with a `404`.

##### Cascading deletes
Often deleting a resource requires that the associated resource be removed; this is usually the case for one-to-many relationships where the child resource has a reference to the parent. 

For example, we might have a `cart` resource that represents a user's shopping cart, and a `cartItem` that represents a single product in the cart. Obviously a cart can have none or many items and deleting the cart should remove any `cartItem` resources. 

###### Example
First, we create our cart.

`POST /cart`

```json
{
    "name": "My cart"
}
```
The server responds with a `200` and a `Location` header, indicating that the cart was successfully created with an `id` of `42`. Next we add some Hy-Vee salsa, which has a `productId` of 123, to the cart:

`POST /cart/42/items`

```json
{
    "productId": 123
}
```

Again, the server responds with a `200` and a `Location` header that tells us that the `cartItem` exists as `/cart/42/cartItems/1`.

If we query for the newly added `cartItem`, we can see the full object:

`GET /cart/42/cartItems/1`

```json
{
  "addDate": "2017-10-31",
  "cartId": 42,
  "id": 1,
  "productId": 123
}
```

Now we decide we don't want to shop at all, so we purge our cart with `DELETE /cart/42`. If we try to do a `GET /cart/42`, the server responds with `404`. Because the entire cart is gone, so are all the `cartItem`s associated with it.

#### Long-running deletes

Occasionally, it may not be possible to process a `DELETE` request in a timely manner. In this case, the API can simply return a `202` status code along with a  `Location` header indicating where the status of the `DELETE` can be checked. An example of this approach can be found in the [section on long-running updates](#lengthy-updates).

That said, it is preferable to make the API quick enough that a task queue or other mechanism is not necessary. If a hard delete is not performant (perhaps there are cascading deletions in order to satisfy foreign key constraints), then consider doing a soft delete instead.

#### Collections
The API should disallow the `DELETE` method for collections -- if a client does wish to remove every item in the collection, they can make multiple `DELETE` requests. The appropriate status code for a `DELETE` issued against a collection is `405`.

## HTTP Status Codes

HTTP requests include a status code, which indicate whether the request was successful, or why exactly it failed. The leading number indicates the type or class of the message. 
### 1xx Informational
 - [100 Continue](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/100)
 - [101 Switching protocols](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/101)
### 2xx Success
 - [200 OK](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/200)
 - [201 Created](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/201)
 - [202 Accepted](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/202)
 - [204 No Content](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/204)
### 3xx Redirection
 - [301 Moved Permanently](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/301)
 - [302 Found](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/302)
 - [303 See Other](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/303)
 - [304 Not Modified](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/304)
### 4xx Client errors
 - [400 Bad Request](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/400)
 - [401 Unauthorized](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/401)
 - [402 Payment Required](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/402)
 - [403 Forbidden](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/403)
 - [404 Not Found](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/404)
 - [405 Method Not Allowed](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/405)
 - [406 Not Acceptable](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/406)
 - [409 Conflict](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/409)
### 5xx Server errors
 - [500 Internal Server Error](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/500)
 - [501 Not Implemented](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/501)
