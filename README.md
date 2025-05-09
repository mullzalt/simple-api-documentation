# Response Structure

Response and request should be in written in JSON

## Response

```typescript
type Response<Error, Data, Metadata, Pagination> =
  | ResponseSuccess<Data, Metadata, Pagination>
  | ResponseFail<Error>;

type ResponseSuccess<Data, Metadata, Pagination> = {
  success: true;
  message: string;
  status: HTTPSuccessCode; // example: 200, 201
  data: Data;
  metadata: Metadata;
  pagination: Pagination;
};

type ResponseFail<Error> = {
  success: false;
  message: string;
  status: HTTPFailCode; // example: 200, 201
  error: Error;
};
```

> Note: The generic could be set as never (not undefined)

> The top level message should provide general ideas of what happened

## Example

Consider the following data structure:

```typescript
type Pagination = {
  current_page: number;
  total_page: number;
  total_item: number;
  page_size: number;
  prev_page: number | null;
  next_page: number | null;
};

// this just give general idea on how your error should be formatted, feel free to modified
type HTTPErrorResponse = {
  message: string; // the message could be left same as the throw error/general error
  name: string;
  cause?: unknown;
  details?: unknown;
};

type Post = {
  id: string; // it could be number or string, it's up to you
  title: string;
  body: string;
  author: string;
};

type PostMetadata = {
  is_owner: boolean;
};
```

The CRUD operation request and responses should be like following:

### Reading many data

> GET yourdomain.com/api/v1/posts

response:

the typed response is:

```typescript
// for many response
type PostsResponse =
  | ResponseSuccess<Post[], never, Pagination>
  | ResponseFail<HTTPErrorResponse>;
// for single response
type PostResponse =
  | ResponseSuccess<Post, never, Pagination>
  | ResponseFail<HTTPErrorResponse>;
```

```json
{
  "success": true,
  "message": "OK",
  "status": 200,
  "data": [
    {
      "id": "1",
      "title": "REST API for dummies part 1",
      "body": "Some text",
      "author": "John Doe"
    },
    {
      "id": "2",
      "title": "REST API for dummies part 2",
      "body": "Some text",
      "author": "John Doe"
    }
  ],
  "pagination": {
    "current_page": 1,
    "total_page": 2,
    "total_item": 4,
    "page_size": 2,
    "prev_page": null,
    "next_page": 2
  }
}
```

empty list should not treated as error, so you could give notice that list is empty in client rather than handling error
example of empty list:

```json
{
  "success": true,
  "message": "OK",
  "status": 200,
  "data": [],
  "pagination": {
    "current_page": 1,
    "total_page": 1,
    "total_item": 0,
    "page_size": 1
    "prev_page": null,
    "next_page": null,
  }
}
```

### Reading Single Response

> GET yourdomain.com/posts/:post_id

> GET yourdomain.com/posts/2

if post exists:

```json
{
  "success": true,
  "message": "OK",
  "status": 200,
  "data": {
    "id": "2",
    "title": "REST API for dummies part 2",
    "body": "Some text",
    "author": "John Doe"
  },
  "metadata": {
    "is_owner": true
  }
}
```

if post not exists:

```json
{
  "success": false,
  "message": "Post not found",
  "status": 404,
  "error": {
    "message": "Not Found",
    "name": "not_found_error"
  }
}
```

### Creating Data

> POST yourdomain.com/posts

request body in json:

```json
{
  "title": "REST API for dummies part 3",
  "body": "Some more text",
  "author": "John Dumb"
}
```

response if data created:

```json
{
  "success": true,
  "message": "Post successfully created",
  "status": 201
}
```

response if user give invalid input:

```json
{
  "success": false,
  "message": "Invalid Post input",
  "status": 400,
  "error": {
    "message": "Schema Error",
    "name": "post_schema_error",
    "detail": [
      {
        "field": "title",
        "message": "Title could not be empty!",
        "expected": "string",
        "received": "undefined"
      }
    ]
  }
}
```

response if something is wrong with the server:

```json
{
  "success": false,
  "message": "Something went wrong",
  "status": 500,
  "error": {
    "message": "Internal server error",
    "name": "internal_server_error"
  }
}
```

### Updating data

> PUT yourdomain.com/posts/id

> PUT yourdomain.com/posts/3

request body in json:

```json
{
  "title": "REST API for dummies part 3 (revision 1)",
  "body": "Some updated text",
  "author": "John Dumb"
}
```

response if data updated:

```json
{
  "success": true,
  "message": "Post successfully updated",
  "status": 200
}
```

### Deleting data

> DELETE yourdomain.com/posts/id

> DELETE yourdomain.com/posts/3

response if data successfully deleted:

```json
{
  "success": true,
  "message": "Post deleted successfully",
  "status": 200
}
```

The error response for both update and delete should pretty much the same as the example above,
notice that this could also return not found if the request ID is non existsent.

> Note: After requesting create, update or delete in client, always invalidate the data in cache or request a new READ operation
