---
title: "Getting Started"
weight: 1
---

{{% steps %}}

### Step 1

Petrichorjs does not have a starter template nor a quick start command like other web server frameworks. Instead you have to install it manually with npm, and to do so use the command bellow.

```shell
$ npm install petrichorjs
```

### Step 2

Then import the package into your source code.

```ts {filename="index.ts"}
import { Router } from "petrichorjs";

const router = new Router();

router.get("/").handle(({ request, response }) => {
    response.ok().json({
        message: "Hello World!",
    });
});

router.listen(3332);
```

### Step 3

Now Petrichorjs has successfully been installed and is ready to run. To test the code you just wrote just execute it and the website should be visible on [http://localhost:3332](http://localhost:3332).

{{< cards >}}
{{< card link="/docs/guides" title="Next steps" icon="arrow-right" >}}
{{< /cards >}}

{{% /steps %}}
