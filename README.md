# Webpack & Vue — remote microfrontend template

Starter template for the [MFE Orchestrator](https://github.com/mfe-orchestrator), listed in the
marketplace as `webpack-remote-vue`. Webpack 5 + Vue 3, wired as a **remote**.

## Requirements

- Node.js 22.15 or newer (CI builds on Node 24, the current LTS)
- [pnpm](https://pnpm.io) 12 (pinned in `packageManager`; `corepack enable` picks it up)

## Getting started

```bash
pnpm install
pnpm dev        # http://localhost:3001
pnpm build      # production build into dist/
```

## Project structure

```
.
├── .github/workflows/build-and-deploy.yml   # build + upload to the orchestrator
├── public/index.html                        # standalone dev page
├── src/
│   ├── App.vue                              # dev only shell
│   ├── components/Button.vue                # the exposed component
│   ├── bootstrap.ts
│   └── index.ts                             # webpack entry, defers to bootstrap
├── package.json
└── webpack.config.ts                        # federation config
```

## What this remote exposes

| module | source |
| --- | --- |
| `./Button` | `src/components/Button.vue` |

Federation name `exampleremote`, entry file `dist/remoteEntry.js`.

### The federation name

```ts
name: 'exampleremote', // mfe-orchestrator:federation-name
```

A webpack host does not import this remote by its URL: it injects `remoteEntry.js` and then looks
the container up **by this name**. When the orchestrator console creates a microfrontend from this
template, it rewrites the string on the line carrying the `mfe-orchestrator:federation-name` marker
to the federation name of the microfrontend's slug: slashes become underscores and dashes go, so
`checkout-new` becomes `checkoutnew`. That is the same name the console writes into the hosts it
configures, and the `nameToIntegrate` it serves in the manifest. Keep the marker on that line; if
you rename the container by hand, use that same derived name.

Add more in the `exposes` map of `webpack.config.ts`. Anything you expose there becomes importable from a
host as `<remote-name>/<key>`.

A host does not hardcode this remote's URL: it asks the orchestrator for it by slug, through the
client SDK (`@mfe-orchestrator-hub/client` 0.3.0 or later):

```js
remotes: {
  // key: what the host imports from; 'checkout-new': the slug in the orchestrator;
  // 'checkoutnew': this remote's federation name
  checkoutnew: "promise globalThis.__MFE_ORCHESTRATOR__.loadWebpackRemote('checkout-new', 'checkoutnew')",
},
```

See the host templates for the other side of the wiring.

## Build output

`pnpm build` writes to `dist/`. The federation entry lands at **`dist/remoteEntry.js`**, which is the `entryPoint` the marketplace entry declares.

Check it after any change to `webpack.config.ts`: the orchestrator serves exactly that path, so a build that
puts the entry somewhere else is broken.

## Deploying

### Upload to the orchestrator

`.github/workflows/build-and-deploy.yml` builds the app and uploads `dist/` with
[`mfe-orchestrator/github-action`](https://github.com/mfe-orchestrator/github-action). It runs on
any pushed tag, or manually via *Run workflow*.

Configure these once, in the repository settings:

| kind | name | value |
| --- | --- | --- |
| secret | `MICROFRONTEND_ORCHESTRATOR_API_KEY` | your orchestrator API key |
| variable | `MICROFRONTEND_SLUG` | the slug of this remote in the orchestrator |
| variable | `MICROFRONTEND_ORCHESTRATOR_DOMAIN` | your console URL, optional, defaults to `https://console.mfe-orchestrator.dev` |

The API key is a **secret**, never a variable and never a literal in the workflow file. If you
prefer hardcoding the two per project values instead of using repository variables, replace the
expressions in the `env:` block at the top of the workflow.

## Notes

- The entry point only does `import('./bootstrap')`. That indirection lets Module Federation negotiate the shared scope before any Vue code runs — keep it.

## License

MIT
