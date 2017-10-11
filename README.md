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
