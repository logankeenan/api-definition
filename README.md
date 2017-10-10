# api-definition


## CRUD

### Read
#### HTTP methods
The HTTP verb associated with reading data is `GET`. A `GET` request is safe, idempotent, and cacheable.

Safe means that the a `GET` request does not affect the state of the server. If we draw an analogy to reading a book, this makes a lot of sense -- the book doesn't change just because you read a page. A `GET` is also idempotent, which means that it can be performed many times in a row and the client will always get the same result -- you can read the same page in a book as many times as you like. Because nothing should change on either the server or the resource when doing a `GET`, the client can store (cache) the result. 

The result of a `GET` is some representation of a resource, usually JSON. You can read more about [`GET` on MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/GET)

#### Responses
There are several status codes involved with reading information from an API:

- [200 (OK)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/200): The resource exists and the client is authorized to view it. The response will include the requested resource.
- [400 (Bad request)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/400): The client's request was invalid (e.g., the URL was malformed).
- [401 (Unauthorized)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/401): The client is not authenticated -- the resource **may** exist and the client **may** have access to it, but they must authenticate first.
- [403 (Forbidden)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/403): The resource may exist, but the client is not authorized to view it. If the existence of the resource is sensitive, then the API can return 404 instead.
- [404 (Not found)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/404): The resource does not exist. 

#### Single Item
If we want to identity a product with an `id` of 123, the URI might look like this: `/product/123`. We prefix the URI with `product` so that it is clear which resource we're talking about, as an API will likely have many different resources. Also, we like to optimize URIs for human readability (machines don't care), so we use `product` rather than a numeric id or UUID.

##### Sample response
Here's what a request to`GET /products/123` might return as the body: 
```json
{
  "_link": "https://api.hy-vee.com/products/123",
  "product": {
      "id":123,
      "name": "Hy-Vee Authentic Salsa (Medium)",
      "upc": "089463115732",
      "size": {
        "volume": 12,
        "uom": "oz"
       }
  }
}
```
Since this was a successful request, the response status code is `200`
#### Collections
If we want all products, we simplify don't specify the `id`, so the URI becomes `/products`.

##### Pagination
If there are many products, the API should split up the response into pages. See GitHub. 
##### Filtering

##### Subset

You may have a requirement that clients be allowed to specify a subset of a collection when no possible filtering fits their needs. 
For example say you're expecting that clients will interact with a small collection of ids, so the resulting URI might look like this:  `/api/resource/1,2,3. However, this is not very RESTful -- the URI is not resource-based. We already have a URI for a collection and a URI for accessing a single item in that collection, this is sufficient for the client's needs. Remember, the API should be fast, such that the client could make separate requests for each item, or simply grab the entire list and filter.
