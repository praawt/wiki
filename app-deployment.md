# Application deployment

The goal of this document is to describe how we deploy applications
on the Winding Tree platform. This is a result of the
[wt-demo-app](https://github.com/windingtree/wt-demo-app/pull/96)
deployment work. There might be something a little different in your
application. Feel free to further improve this guide.

## Deployment process

- Deploy is triggered only on a tagged commit.
- Deployment process is run on Travis CI.
- Applications are running from [now](https://zeit.co/now).

## Configuration

- Read the guide for Travis on [now](https://zeit.co/docs/examples/travis)
- We need `now` token - every application has its own. Ask on
[Rocket Chat](https://windingtree.rocket.chat) for that one.
- Then we need to get the application ready. `now` works as a CLI provided by
an *npm package*. But it doesn't really make sense to have it as an application
dependency.
- After each deployment is triggered, `now` automatically runs `npm run build` and
`npm start` (That can be overloaded by `now-build` and `now-start` npm scripts).
- It's good to know, that by default, the now environment installs the dev dependencies
where we should keep everything required for building the app.
- It's good to know, that free accounts on now are limited by 1MB per file. So for testing,
make sure your app is small or well-split.
- DNS alias is configured separately, ask on [Rocket Chat](https://windingtree.rocket.chat).

### `package.json`

- Required scripts:
  - `build` (or `now build`) - That's where we compile/configure our application
  for the deployed environment. It can probably be omitted for very simple apps.
  - `start` (or `now start`) - That's how the app is served in production. It can
  be something like `serve --single ./dist` or `node ./dist/index.js`.
  - `deploy` - That's what is run on Travis CI and actually triggers the deployment.
  It should be something like `now deploy -e NODE_ENV=production --token $NOW_TOKEN --npm --public`
  Note that the `NODE_ENV` specification might result into non-installed `devDependencies`.
  - `alias` - That's useful when we are pointing a custom domain to the deployment.
  It should be `now alias --token=$NOW_TOKEN`.
- New property `now`
  - It basically configures the deployment and absolutely necessary is only
    - `name` - Basename of the app, such as `wt-demo-app`
  - Other useful fields
    - `alias` - The actual domain name that points to this deployment.
- It is wise to specify the `engines` field to enforce a node version that actually works.

### `.travis.yml`

- As we want to release only tagged versions, we need to do certain changes to
the default Travis configuration from now manual.
- If we already have the travis file, just add the `build and deploy` stage.
- Don't forget to add the encrypted `NPM_TOKEN`.

```yaml
language: node_js
node_js:
- node
cache: npm
# This skips the installation phase on Travis. Since the app is built
# directly on now, we don't need to spend time with this on CI server.
install: true 
jobs:
  include:
  - stage: build and deploy
    # This ensures that build is triggered only on tags
    if: tag IS present
    script:
    - npm i -g now
    - npm run deploy
    - npm run alias
env:
  global:
    secret: <automatically added by travis encrypt command described in now guide>
```