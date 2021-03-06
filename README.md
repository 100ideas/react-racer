# jfarmer/react-racer

- original: https://github.com/jfarmer/react-racer
- codesandbox fork: https://github.com/100ideas/react-racer/tree/codesandbox
- codesandbox demo (fresh) : http://codesandbox.io/s/github/100ideas/react-racer/tree/codesandbox
- codesandbox demo (9/4/19): https://codesandbox.io/s/38971w83l5

### setup
- should just work.
- problems? 
  - try hardcoding `server` url in `frontend/App.js` - use full url of this codesandbox instance (visible in browser preview) not including trailing slace
  - remove or add syncrepo in `yarn start`
  - separately start server and client using `yarn start:server`, `yarn start:frontend` in separate terminals
  - CORS & websockets? 
    - https://socket.io/docs/server-api/#server-origins-value
    - https://stackoverflow.com/questions/24058157/socket-io-node-js-cross-origin-request-blocked

### how this works [on codesandbox](https://codesandbox.io/s/38971w83l5)

The `start` script in `package.json` first uses `svn` to sync root of github repo `jfarmer/react-racer` to codesandbox at `~/.` aka `/sandbox/`. This should ensure visitors/forkers of this codesandbox get an up-to-date demo.

Ran into CORS pain with `socket.io`, maybe b/c codesandbox only lets us expose one port; or maybe b/c I hate CORS more than webpack in terms of unexpected configuration complexity and don't know how to use it well. Solution was to use `create-react-app`'s `proxy` feature in conjunction with setting a socket.io custom `path` (not `namespace`!): this directs the client to establish its websocket connection at `https://<host>/cra-proxy` vs `https://<host>:4001/socket.io`. I thought this might be better than the vanilla cra proxy feature b/c it by default proxys everything that doesn't match a route. See:
- https://socket.io/docs/client-api/#With-custom-path
  - w/o custom path socket.io directs all ws connections to <origin>/socket.io/<params>
- https://facebook.github.io/create-react-app/docs/proxying-api-requests-in-development
- https://github.com/chimurai/http-proxy-middleware#shorthand for "shorthand" form of `proxy` config string used in package.json (how to set route filter)
- `frontend/package.json`: `proxy: ...`
- `frontend/src/App.js:` `socketPath = ...`
- `api/api.js`: `const io = socketIo(server).of("/cra-proxy");`

If you want to change the port cra uses internally on the codesandbox server, see `./sandbox.config.json: container.port`.

### using svn w/ github

```bash
# sync repo:
svn checkout -r HEAD --force https://github.com/jfarmer/react-racer/trunk .

# svn info
svn help checkout

checkout (co): Check out a working copy from a repository.
usage: checkout URL[@REV]... [PATH]

  If specified, REV determines in which revision the URL is first
  looked up.

  If PATH is omitted, the basename of the URL will be used as
  the destination. If multiple URLs are given each will be checked
  out into a sub-directory of PATH, with the name of the sub-directory
  being the basename of the URL.

  If --force is used, unversioned obstructing paths in the working
  copy destination do not automatically cause the check out to fail.
  If the obstructing path is the same type (file or directory) as the
  corresponding path in the repository it becomes versioned but its
  contents are left \'as-is\' in the working copy. This means that an
  obstructing directory\'s unversioned children may also obstruct and
  become versioned.  For files, any content differences between the
  obstruction and the repository are treated like a local modification
  to the working copy.  All properties from the repository are applied
  to the obstructing path.

Valid options:
  --force                 : force operation to run

  -r [--revision] ARG      : ARG (some commands also take ARG1:ARG2 range)
                            ...
                           'HEAD'       latest in repository
```

### set up shell

```bash
alias l='ls -la'

```
---

archive of `sandbox.config.json`

```json
{
  "infiniteLoopProtection": true,
  "hardReloadOnChange": false,
  "view": "browser",
  "server": "true",
  "container": {
    "port": 3000
  }
}
```

---

### codesandbox advanced config documentation

https://github.com/CompuIves/codesandbox-client/issues/1199#issuecomment-430590886

> You can force the container port that CodeSandbox proxies on https://sandbox-id.sse.codesandbox.io/ by creating a sandbox.config.json file in the root of your sandbox, with the following contents:

```
{
  "container": {
    "port": 8000
  }
}
```

---

https://github.com/CompuIves/codesandbox-client/issues/1465

> We assume the start script (actually, we look for dev, develop, serve, start, in this particular order) starts the main process of the sandbox (the dev server, if you will). So, instead of demo, you can use one of dev, develop or serve.
