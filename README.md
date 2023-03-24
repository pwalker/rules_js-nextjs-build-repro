# Bazel + Next.js Build Reproduction

I tried to keep this app pretty close to the [official next.js example](https://github.com/aspect-build/bazel-examples/tree/main/next.js), but I needed to make a few minor tweaks. I moved the `alpha` app dependencies to `apps/alpha/package.json` instead of the repo root `package.json`. Also, I got rid of the nested `BUILD` files in the differect alpha app directories.

## Reproducing

The error shows up when I install a package that has a peer dependency on React, like `react-hook-form` or in this example `@tanstack/react-query`. To reproduce the error, try to run the app in from a production build:

```sh
bazel run //apps/alpha:next_start
```

This should fail with an error like:

```
TypeError: Cannot read properties of null (reading 'useEffect')
    at Module.exports.useEffect (/path/to/app/.cache/bazel/_bazel_pwalker/cebc07f6ad5b195871f76e4d1e59e83e/execroot/__main__/bazel-out/k8-fastbuild/bin/node_modules/.aspect_rules_js/react@18.2.0/node_modules/react/cjs/react.production.min.js:24:292)
    at QueryClientProvider (file:///path/to/app/.cache/bazel/_bazel_pwalker/cebc07f6ad5b195871f76e4d1e59e83e/execroot/__main__/bazel-out/k8-fastbuild/bin/node_modules/.aspect_rules_js/@tanstack+react-query@4.28.0_biqbaboplfbrettd7655fr4n2y/node_modules/@tanstack/react-query/build/lib/QueryClientProvider.mjs:46:9)
    at Wc (/path/to/app/.cache/bazel/_bazel_pwalker/cebc07f6ad5b195871f76e4d1e59e83e/sandbox/linux-sandbox/116/execroot/__main__/bazel-out/k8-fastbuild/bin/node_modules/.aspect_rules_js/react-dom@18.2.0_react@18.2.0/node_modules/react-dom/cjs/react-dom-server.browser.production.min.js:68:44)
    # and on and on
```

However, the build succeeds if you don't involve bazel:

```sh
bazel build //packages/... # this builds @nextjs-example/one
pnpm next build
pnpm next start
```

I've got `hoist=false` in my `.npmrc` file, so I think I'm getting my local node modules as close as possible to how they are laid out in bazel-land.

## Workaround

It's possible to work around this issue by using the [transpilePackages](https://nextjs.org/blog/next-13-1#built-in-module-transpilation-stable) configuration option:

```js
// file: next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  //...

  transpilePackages: ["@tanstack/react-query"],
};

module.exports = nextConfig;
```

With this workaround, the next.js build succeeds both in and out of bazel.
