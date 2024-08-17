---
title: "Handling Requests"
weight: 2
---

Petrichorjs has tried making it as easy as possible to handle requests. This library also allows for returning both simple and [streaming responses](#streaming-responses).

## The handle function

The `handle` method on routes is the method that takes a function which responsible for taking the request and generating and returning a suitable response based on the request. This function has access to both the request and response objects.

```ts
router.get("/").handle(({ request, response }) => {
    response.ok().json({
        message: "Hello World!",
    });
});
```

## Response Body

The response body is the content the client will get back from the request. In Petrichorjs there are some helpers for returning a body since the `Content-Type` header on the response has to be set correctly depending on whats returned.

```ts
response.json({ message: "Hello World!" }); // Content-Type: application/json
response.html("<h1>Hello World!</h1>"); // Content-Type: text/html
response.text("Hello World!"); // Content-Type: text/plain
response.body("Hello World!"); // Doesn't set the Content-Type header
```

Calling one of these functions will overwrite any previous call to them before the response is sent to the client.

```ts
// GET / Returns { message: "Bye!" }
router.get("/").handle(({ request, response }) => {
    response.json({ message: "Hello!" });
    response.json({ message: "Bye!" }); // Only this will be sent to the client
});
```

## Response Status Codes

The response status code is used by the client to tell if the request was successful or not. Petrichorjs provides some helper functions for the most common response codes.

```ts
response.ok(); // Sets response status code to 200
response.created(); // Sets response status code to 201
response.badRequest(); // Sets response status code to 400
response.unauthorized(); // Sets response status code to 401
response.forbidden(); // Sets response status code to 403
response.notFound(); // Sets response status code to 404
response.unprocessableContent(); // Sets response status code to 422
response.tooManyRequests(); // Sets response status code to 429
response.internalServerError(); // Sets response status code to 500
response.notImplemented(); // Sets response status code to 501
```

Status codes can also be set manually if a helper function doesn't exist.

```ts
import { statusCodes } from "petrichorjs";

response.status(statusCodes.Ok);
response.status(200);
```

## Redirects

Redirecting the client can be useful, therefor the response has a helper function for it.

{{< callout type="info" >}}
Response status codes for responses with redirects should not be set because the `redirect` method sets its own status code.  
{{< /callout >}}

```ts
router.get("/").handle(({ request, response }) => {
    response.redirect(statusCodes.SeeOther, "/abc");
});
```

## Request Body

Similar to the response body, the request body can be of multiple different types and has to be parsed. Therefor if your expecting a `json` body you should use the appropriate method on the request object.

```ts
await request.body(); // Text body
await request.json(); // Parsed json body
await request.text(); // Text body
```

The request body can be validated using a [validator](/docs/guides/validating-requests#validating-the-body). More information about validators can be found [here](/docs/guides/validating-requests).

```ts
router
    .get("/login")
    .validate({
        body: (data) => {
            if (
                !data ||
                typeof data !== "object" ||
                !("email" in data) ||
                !("password" in data) ||
                typeof data.email !== "string" ||
                typeof data.password !== "string"
            ) {
                return {
                    success: false,
                    errors: [{ message: "Invalid body!" }],
                };
            }

            return { success: true, data: data };
        },
    })
    .handle(async ({ request, response }) => {
        await request.json(); // { email: string, password: string }
    });
```

### Files

Files can be sent with the request body. For the body parser to know files might be present have the `Content-Type` header set to `multipart/form-data`. Then you can access the files using the `files` method or the `json` method on the request object.

```ts
await request.files(); // Returns the files in the same format as how query params are getting parsed.
await request.filesFlat(); // Returns all files from the request flat.

await request.json(); // Returns everything in the body sent with the request, including files.
```

## Request Parameters

Request parameters are the dynamic parameters from the route path. They can be parsed or left as is. You can read more about parsing parameters [here](/docs/guides/parsing-parameters). Read more about how path params work [here](/docs/guides/routing#dynamic-routes).

```ts
import { intParser } from "petrichorjs";

request
    .get("/users/:userId/comments/:commentId/*")
    .parse({
        userId: intParser,
    })
    .handle(({ request, response }) => {
        request.params; // { userId: number, commentId: string, wildcard: string }
    });
```

## Request Locals

Locals in Petrichorjs is a way to send data from middleware upstream to the handler. It can be used to for example get a user from the database before every request to a specific rout group. Read more about locals and before functions [here](/docs/guides/middleware#before-functions). Locals from parent route groups will also be sent to the handler.

```ts
router
    .get("/users/:id")
    .parse({
        // Before functions also work with parser functions
        id: intParser,
    })
    // Before functions are middleware that only run before the request comes to the handler
    .before(async (request) => {
        const user = await getUser(request.params.id);
        if (!user) throw new HttpError(statusCodes.NotFound, "User not found!");

        return {
            user: user,
        };
    })
    .handle(({ request, response }) => {
        request.locals; // { user: User }

        response.ok().json(request.locals.user);
    });
```

## Request Query Params

The query params, also known as search params, are a part of a request is the part after the question mark in the url (`?a=1&b=2&c`). These can also be validated like the [request body](#request-body), which you can read more in-depth about in the [validating requests](/docs/guides/validating-requests#query-parameters) guide.

```ts
request.query.get("name"); // Get a single query param
request.query.getAndParse("id", intParser);
// Get a single query param and parse it,
// or throw an http error if it doesn't exist or fails the parser.
request.query.all(); // Returns all query params in a map
```

You might also have a scenario where you want to be able to have multiple query parameters with the same name. It is also possible, but in that case you have to use the `toObject` method on the query params object.

```ts
request.query.toObject();
// "name=John&pet=cat" => { name: "John", pet: "cat" }
// "user.name=John&user.pet=cat" => {user: { name: "John", pet: "cat" }}
// "pets=cat&pets=dog" => { pets: ["cat", "dog"] }
```

As seen above you can validate single query parameters, however you can also use a validator function like how the request body can be validated.

```ts
router
    .get("/search")
    .validate({
        query: (data) => {
            if (!("q" in data) || typeof data.q !== "string")
                return {
                    success: false,
                    errors: [{ message: "Query query param missing!" }],
                };

            return {
                success: true,
                data: {
                    q: data.q,
                },
            };
        },
    })
    .handle(async ({ request, response }) => {
        request.query.get("q"); // string
        response.ok().json(search(request.query.get("q")));
    });
```

## Cookies

Cookies are a part of both the request and response. Cookies cannot be validated, but you can use before functions or regular middleware to validate them if needed for something like auth or anything else.

```ts
request.cookies.get("name"); // Get a single cookie
request.cookies.all(); // Returns all cookies from the request in a map
request.cookies.size(); // Returns the number of cookies sent with the request

response.cookie("session", "abc123"); // Set a single cookie without options
response.cookie("session", "abc123", { ...options }); // Set a single cookie with options
```

Here's an example of everything working together.

```ts
router.get("/").handle(({ request, response }) => {
    const welcomedBefore = request.cookies.get("welcomed") !== undefined;
    response.cookie("welcomed", "true");

    return response.ok().json({
        message: welcomedBefore ? "Hello again!" : "Welcome to my site!",
    });
});
```

## Headers

Headers can both be sent to the server and returned to the client.

```ts
router.get("/").handle(({ request, response }) => {
    request.headers; // Get all headers sent with the request
    request.contentType; // Shorthand for the request Content-Type header

    response.header("Content-Type", "text/plain"); // Set a single header on the response
    response.headers({ "Content-Type": "text/plain" }); // Set multiple headers
});
```

## Streaming Responses

Streaming responses can be done in Petrichorjs. Streamed responses also work with [middleware](/docs/guides/middleware#streamed-responses). Streamed responses are a way for the handler to send some information to the client at a time.

```ts
router.get("/").handle(({ request, response }) => {
    // Streams are created with this function
    // All streaming logic should be inside this callback function
    response.streamResponse(async (stream) => {
        let i = 0;
        while (i < 10) {
            await stream.write(`${i}`);
            await stream.sleep(10); // 10 ms
            i++;
        }
        await stream.close();
    });
});
```
