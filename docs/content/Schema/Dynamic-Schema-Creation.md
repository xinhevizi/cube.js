---
title: Dynamic Schema Creation
permalink: /schema/dynamic-schema-creation
category: Data Schema
menuOrder: 2
---

Cube.js allows schemas to be created on-the-fly using a special
[`asyncModule()`][ref-async-module] function only available in the [schema
execution environment][ref-schema-env]. `asyncModule()` allows registering an
async function to be executed at the end of the data schema compile phase so
additional definitions can be added.

[ref-schema-env]: /schema-execution-environment
[ref-async-module]: /schema-execution-environment#asyncmodule

[comment]: <> (In multi-tenant configurations, there is often a need to
dynamically re-compute schemas.)

<!-- prettier-ignore-start -->
[[warning | Note]]
| Each `asyncModule` call will be invoked only once per schema compilation.
<!-- prettier-ignore-end -->

## Generation

In the following example, we retrieve a JSON object representing all our cubes
using `fetch()` and then use the [`cube()` global function][ref-globals] to
generate schemas from that data:

[ref-globals]:
  https://cube.dev/docs/schema-execution-environment#cube-js-globals-cube-and-others

```javascript
const fetch = require('node-fetch');

asyncModule(async () => {
  const dynamicCubes = await (
    await fetch('http://your-api-endpoint/dynamicCubes')
  ).json();

  dynamicCubes.forEach((dynamicCube) => {
    cube(dynamicCube.title, {
      dimensions: dynamicCube.dimensions,
      measures: dynamicCube.measures,
      preAggregations: {
        main: {
          type: `originalSql`,
        },
      },
    });
  });
});
```

## Recompilation

It is also useful to be able to recompile the schema when there are changes in
the underlying input data. For this purpose, the [`schemaVersion`
][link-config-schema-version] value in the `cube.js` configuration options can
be specified as an asynchronous function:

```javascript
module.exports = {
  schemaVersion: async ({ authInfo }) => {
    const schemaVersions = await (
      await fetch('http://your-api-endpoint/schemaVersion')
    ).json();

    return schemaVersions[authInfo.tenantId];
  },
};
```

[link-config-schema-version]: /config#options-reference-schema-version
