---
title: "Petrichorjs"
layout: hextra-home
---

{{< hextra/hero-badge >}}

  <div class="hx-w-2 hx-h-2 hx-rounded-full hx-bg-primary-400"></div>
  <span>Free, open source</span>
  {{< icon name="arrow-circle-right" attributes="height=14" >}}
{{< /hextra/hero-badge >}}

<div class="hx-mt-6 hx-mb-6">
{{< hextra/hero-headline >}}
  Build modern web servers&nbsp;<br class="sm:hx-block hx-hidden" />with full type safety
{{< /hextra/hero-headline >}}
</div>

<div class="hx-mb-12">
{{< hextra/hero-subtitle >}}
  Easy to use, fast to write, fully type safe &nbsp;<br class="sm:hx-block hx-hidden" />web server framework for TypeScript
{{< /hextra/hero-subtitle >}}
</div>

<div class="hx-mb-6">
{{< hextra/hero-button text="Get Started" link="docs" >}}
</div>

<div class="hx-mt-6"></div>

<div class="hx-w-full">
{{< tabs items="Creating routes,Parsing params,Validating body,Using middleware" >}}
{{< tab >}}

```ts
import { Router } from "petrichorjs";

const router = new Router();

router.get("/").handle(({ request, response }) => {
    response.ok().json({
        message: "Hello World!",
    });
});

const usersRouter = router.group("/users").handle();
usersGroup.get("/").handle(async ({ request, response }) => {
    response.ok().json(await getUsers());
});

router.listen(3332);
```

{{< /tab >}}
{{< tab >}}

```ts
import { intParser } from "petrichorjs";

router
    .get("/users/:id")
    .parse({ id: intParser })
    .handle(({ request, response }) => {
        response.ok().json(await getUser(request.params.id));
    });
```

{{< /tab >}}
{{< tab >}}

```ts
import vine from "@vinejs/vine";
import { validator } from "@petrichorjs/validator-vinejs";

router
    .post("/users")
    .validate({
        body: validator(
            vine.object({
                id: vine.number().withoutDecimals(),
                email: vine.string().email(),
                profile: vine.object({
                    username: vine.string(),
                }),
            })
        ),
    })
    .handle(async ({ request, response }) => {
        const data = await request.json();
        await createUser(data);

        response.created().json({
            data: {
                id: data.id,
            },
        });
    });
```

{{< /tab >}}
{{< tab >}}

```ts
import { trailingSlash } from "petrichorjs";

router.use(trailingSlash());
router.use(async ({ request, response }, next) => {
    console.log(`${request.method} ${request.url}`);
    await next();
});

router
    .get("/users/:id")
    .parse({ id: intParser })
    .before(async (request) => {
        return {
            user: await getUser(request.params.id),
        };
    })
    .handle(({ request, response }) => {
        response.ok().json(request.locals.user);
    });
```

{{< /tab >}}
{{< /tabs >}}

</div>
