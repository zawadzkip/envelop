# @envelop/response-cache

## 3.0.0

### Major Changes

- 887fc07: **BREAKING** Require the user to provide a `session` function by default.

  Previously, using the response cache automatically used a global cache. For security reasons there is no longer a default value for the `session` config property. If you did not set the `session` function before and want a global cache that is shared by all users, you need to update your code to the following:

  ```ts
  import { envelop } from '@envelop/core'
  import { useResponseCache } from '@envelop/response-cache'

  const getEnveloped = envelop({
    plugins: [
      // ... other plugins ...
      useResponseCache({
        // use global cache for all operations
        session: () => null
      })
    ]
  })
  ```

  Otherwise, you should return from your cache function a value that uniquely identifies the viewer.

  ```ts
  import { envelop } from '@envelop/core'
  import { useResponseCache } from '@envelop/response-cache'

  const getEnveloped = envelop({
    plugins: [
      // ... other plugins ...
      useResponseCache({
        // return null as a fallback for caching the result globally
        session: context => context.user?.id ?? null
      })
    ]
  })
  ```

- a5d8dcb: **Better default document string storage**

  Previously non parsed operation document was stored in the context with a symbol to be used "documentString" in the later. But this can be solved with a "WeakMap" so the modification in the context is no longer needed.

  **BREAKING CHANGE**: Replace `getDocumentStringFromContext` with `getDocumentString`

  However, some users might provide document directly to the execution without parsing it via `parse`. So in that case, we replaced the context parameter with the execution args including `document`, `variableValues` and `contextValue` to the new `getDocumentString`.

  Now a valid document string should be returned from the new `getDocumentString`.

  **Custom document string caching example.**

  ```ts
  const myCache = new WeakMap<DocumentNode, string>()

  // Let's say you keep parse results in somewhere else like below
  function parseDocument(document: string): DocumentNode {
    const parsedDocument = parse(document)
    myCache.set(parsedDocument, document)
    return parsedDocument
  }

  // Then you can interact with your existing caching solution inside the response cache plugin like below
  useResponseCache({
    getDocumentString(document: DocumentNode): string {
      // You can also add a fallback to `graphql-js`'s print function
      // to let the plugin works
      const possibleDocumentStr = myCache.get(document)
      if (!possibleDocumentStr) {
        console.warn(`Something might be wrong with my cache setup`)
        return print(document)
      }
      return possibleDocumentStr
    }
  })
  ```

  **Migration from `getDocumentStringFromContext`.**

  So if you use `getDocumentStringFromContext` like below before;

  ```ts
  function getDocumentStringFromContext(contextValue: any) {
    return contextValue.myDocumentString
  }
  ```

  You have to change it to the following;

  ```ts
  import { print } from 'graphql'
  function getDocumentString(executionArgs: ExecutionArgs) {
    // We need to fallback to `graphql`'s print to return a value no matter what.
    return executionArgs.contextValue.myDocumentString ?? print(executionArgs.document)
  }
  ```

## 2.4.0

### Minor Changes

- 8bb2738: Support TypeScript module resolution.
- Updated dependencies [8bb2738]
  - @envelop/core@2.4.0

## 2.3.3

### Patch Changes

- fbf6155: update package.json repository links to point to the new home
- Updated dependencies [fbf6155]
  - @envelop/core@2.3.3

## 2.3.2

### Patch Changes

- Updated dependencies [07d029b]
  - @envelop/core@2.3.2

## 2.3.1

### Patch Changes

- Updated dependencies [d5c2c9a]
  - @envelop/core@2.3.1

## 2.3.0

### Minor Changes

- Updated dependencies [af23408]
  - @envelop/core@2.3.0

## 2.2.0

### Minor Changes

- Updated dependencies [ada7fb0]
- Updated dependencies [d5115b4]
- Updated dependencies [d5115b4]
  - @envelop/core@2.2.0

## 2.1.1

### Patch Changes

- 5400c3f: fix infinite loop while applying schema transforms

## 2.1.0

### Minor Changes

- Updated dependencies [78b3db2]
- Updated dependencies [f5eb436]
  - @envelop/core@2.1.0

## 2.0.0

### Patch Changes

- Updated dependencies [4106e08]
- Updated dependencies [aac65ef]
- Updated dependencies [4106e08]
  - @envelop/core@2.0.0

## 1.0.1

### Patch Changes

- 3dfddb5: Bump graphql-tools/utils to v8.6.1 to address a bug in getArgumentsValues
- Updated dependencies [3dfddb5]
  - @envelop/core@1.7.1

## 1.0.0

### Patch Changes

- Updated dependencies [d9cfb7c]
  - @envelop/core@1.7.0

## 0.6.0

### Minor Changes

- b919b21: Add cross-platform support for platforms that do not have the `Node.js` `crypto` module available by using the `WebCrypto` API. This adds support for deno, cloudflare workers and the browser.

  **BREAKING**: The `BuildResponseCacheKeyFunction` function type now returns `Promise<string>` instead of `string.`. The function `defaultBuildResponseCacheKey` now returns a `Promise`. The `UseResponseCacheParameter.buildResponseCacheKey` config option must return a `Promise`.
  **BREAKING**: The `defaultBuildResponseCacheKey` now uses the hash algorithm `SHA256` instead of `SHA1`.

## 0.5.1

### Patch Changes

- b1a0331: Properly list `@envelop/core` as a `peerDependency` in plugins.

  This resolves issues where the bundled envelop plugins published to npm had logic inlined from the `@envelop/core` package, causing `instanceof` check of `EnvelopError` to fail.

- Updated dependencies [b1a0331]
  - @envelop/core@1.6.1

## 0.5.0

### Minor Changes

- 090cae4: GraphQL v16 support

## 0.4.0

### Minor Changes

- 04120de: add support for GraphQL.js 16

## 0.3.0

### Minor Changes

- 0623cf7: Introspection query operations are no longer cached by default.

  In case you still want to cache query operations you can set the `ttlPerSchemaCoordinate` parameter to `{ "Query.__schema": undefined }` for caching introspection forever or `{ "Query.__schema": 100 }` for caching introspection for a specific time. We do not recommend caching introspection.

  Query operation execution results that contain errors are no longer cached.

## 0.2.1

### Patch Changes

- 9688945: Allow to set ttl=0 to disable caching, and use ttlPerType to maintain a whitelist
- a749ec0: Include operationName for building the cache key hash. Previously, sending the same operation document with a different operationName value could result in the wrong response being served from the cache.

  Use `fast-json-stable-stringify` for stringifying the variableValues. This will ensure that the cache is hit more often as the variable value serialization is now more stable.

## 0.2.0

### Minor Changes

- 075fc77: Expose metadata by setting the `includeExtensionMetadata` option.

  - `extension.responseCache.hit` - Whether the result was served form the cache or not
  - `extension.responseCache.invalidatedEntities` - Entities that got invalidated by a mutation operation

  Take a look at the README for mor information and examples.

## 0.1.0

### Minor Changes

- 823b335: initial release
