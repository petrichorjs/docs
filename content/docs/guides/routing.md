---
title: "Routing"
weight: 1
---

Routing in Petrichorjs is made to be as simple as possible. To create a route you just have to specify the method and path and then handle the request to that endpoint.

## Routes

Routes can be created on both the router and on [route groups](#route-groups). Either use a http verb method (`.get` or `.post`) or the `.all` method to catch all methods to that endpoint.

```ts
router.get("/users").handle(...); // Handles GET requests to /users
router.post("/users").handle(...); // Handles POST requests to /users
router.put("/users").handle(...); // Handles PUT requests to /users
router.delete("/users").handle(...); // Handles DELETE requests to /users

router.on("MY_METHOD", "/users").handle(...) // Handles MY_METHOD requests to /users

router.all("/users").handle(...) // Handles any request to /users no matter the method
```

Route builder methods can also be chained as follows.

```ts
router.get("/users").post().delete().on("MY_METHOD").handle(...)
// Handles GET, POST, DELETE and MY_METHOD requests to /users
```

### Route Priority

Certain routes have certain priority over others. This means you can do route overloading (like you can to function overloading in TypeScript). The order of priority is as follows:

1. Static routes
2. Required dynamic parameters (Parser must not throw)
3. Optional dynamic parameters (Parser must not throw)
4. Required wildcards
5. Optional wildcards

```ts
router.get("/users/:id").parse({ id: intParser }).handle(...)
router.get("/users/:id").handle(...)
router.get("/:slug").handle(...)
router.get("/*?").handle(...)

// GET /users/1 will be handled in the first route
// GET /users/john will be handled in the second route
// GET /users/ will be handled in the third route
// GET / or /abc/123 will be handled in the fourth route
```

### Dynamic Routes

A dynamic route is a route that has a dynamic part to it, meaning it can handle requests to different endpoints and then use the data from the requested url as parameters. Check out how to [handle requests](/docs/guides/handling-requests#request-parameters) to learn more about how to use the params when handling the request.

{{< callout type="warning" >}}
Dynamic routes cannot have pre- or suffixes in the same slug. `/@:userId` is therefor not allowed. To learn how to combat this check out [parsing params](/docs/guides/parsing-parameters#pre--and-suffixes).
{{< /callout >}}

Required route params can be created using the syntax: `/:<name>`.

```ts
// Will only handle GET requests to /users/<anything> and not to /users
router.get("/users/:id").handle(({ request, response }) => {
    request.params; // { id: string }
});
```

Optional route params can be created in the same way as required ones except that optional parameters can only be followed by other optional ones.

```ts
// Will handle GET requests to both /users/<anything> and /users
router.get("/users/:id?").handle(({ request, response }) => {
    request.params; // { id?: string | undefined }
});
```

Wildcard route params behave like any regular route parameter except for the fact that it catches all routes after it. There can therefor only exist one wildcard param per route. Wildcard routes can also be optional.

```ts
// Will handle GET requests to /users/...
router.get("/users/*").handle(({ request, response }) => {
    request.wildcard; // { wildcard: string }
});

// Will handle GET requests to both /users/... and /users
router.get("/users/*?").handle(({ request, response }) => {
    request.params; // { wildcard?: string | undefined }
});
```

## Route Groups

Route groups are used to group routes together like the name implies. Route groups can also have their own middleware, validators and parameter parsers. All child routes and child route groups to the route group will inherit the middleware, validators and parameter parsers. The middleware will keep its order. The paths to the nested routes will also build on the route groups path.

{{< callout type="info" >}}
The `.handle` method on route group builders always has to be called before assigning the group to a variable. It doesn't take any arguments and is only used to build the route group.
{{< /callout >}}

```ts
const usersGroup = router.group("/users").handle()
usersGroup.get("/").handle(...) // GET /users

// Creating nested route groups is also possible
const userGroup = usersGroup.group("/:id").parse({
    id: intParser
}).handle()

// The parse function from the route group is inherited
userGroup.get("/").handle(...) // GET /users/123
userGroup.post("/follow").handle(...) // POST /users/123/follow
```

### Separating into different files

Separating the route groups is also possible. It can be done by just exporting the route group.

{{< callout type="info" >}}
The need to separate the index and router files is to avoid circular imports and because the `listen` method on the router has to be called after all routes has been registered.
{{< /callout >}}

```ts {filename="index.ts"}
import { router } from "./router.js";
import "./users.js";

router.listen(3332);
```

```ts {filename="router.ts"}
import { Router } from "petrichorjs";

export const router = new Router();

router.get("/").handle(({ request, response }) => {
    response.ok().json({
        message: "Hello World!",
    });
});
```

```ts {filename="users.ts"}
import { router } from "./router.js";

const usersGroup = router.group("/users").handle();

usersGroup.get("/").handle(({ request, response }) => {
    response.ok().json([
        {
            id: 1,
            username: "John Doe",
        },
    ]);
});
```
