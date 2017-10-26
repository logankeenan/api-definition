# api-definition

## CRUD

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
