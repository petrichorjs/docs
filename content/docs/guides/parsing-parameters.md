---
title: "Parsing Parameters"
weight: 3
---

Route parameters are parameters that exist in the url/path the route should handle requests for. They can be parsed meaning that you can for example try to convert the string param to an integer. If the parser says the param is invalid the router will keep looking until it finds another suitable route handler.

```ts
router
    .get("/:char")
    .parse({
        char: ({ param, unparseable }) => {
            if (param.length !== 1 || !param.match(/[a-z]/i)) {
                unparseable();
            }
            return param.toLowerCase();
        },
    })
    .handle(({ request, response }) => {
        request.params; // { char: string }
    });

// GET /a 200 Ok
// GET /abc 404 Not Found
```

## With Route Groups

Route params also work with route groups. Note that you don't have to parse the param in the same group that it was declared in, meaning that in the example bellow you could have had the parser function on the route (after the `get`) function instead.

```ts
const userGroup = router
    .group("/users/:id")
    .parse({
        id: intParser,
    })
    .handle();

userGroup.get("/").handle(({ request, response }) => {
    request.params; // { id: number }
});
```

## Pre- and Suffixes

Some routers and web server frameworks allow for both pre- and suffixes. That is something Petrichorjs does not allow directly. Instead you should use a parser function to do so.

```ts
router
    .get("/:id")
    .parse({
        id: ({ param, unparseable }) => {
            if (!param.startsWith("@")) unparseable();
            return param.slice(1);
        },
    })
    .handle(({ request, response }) => {
        request.params; // { id: string }
    });

// GET /@abc 200 Ok { id: "abc" }
// GET /abc 404 Not Found
```

## Built-in Parsers

Petrichorjs offers some built-in parsers for parsing params.

```ts
import { intParser } from "petrichorjs";

router
    .get("/:id")
    .parse({ id: intParser })
    .handle(({ request, response }) => {
        request.params; // { id: number }
    });
```

Currently there exists:

-   `intParser` for parsing integers
