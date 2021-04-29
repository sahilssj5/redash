You can use this folder to add scripts and configurations to customize Redash build and development loop.

## How to customize Webpack

### Configurable parameters

You can override the values of configurable parameters by exporting a `CONFIG` object from the module located at `scripts/config`.

Currently the following parameters are supported:

- **staticPath**: Override the location of Redash static files (default = `/static/`).

#### Example Configuration (`scripts/config.js`):

```javascript
module.exports = {
  staticPath: "my/redash/static/path"
};
```

### Programmatically

For advanced customization, you can provide a script to apply any kind of overrides to the default config as provided by `webpack.config.js`.

The override module must be located under `scripts/webpack/overrides`. It should export a `function` that receives the Webpack configuration object and returns the overridden version.

#### Example Override Script (`scripts/webpack/overrides.js`):

This is an example of an override that enables Webpack stats.

```javascript
function applyOverrides(webpackConfig) {
  return {
    ...webpackConfig,
    stats: {
      children: true,
      modules: true,
      chunkModules: true
    }
  };
}

module.exports = applyOverrides;
```

#### Using JWT Auth
Set environment variables
```
REDASH_JWT_AUTH_ISSUER - jwt issuer
REDASH_JWT_AUTH_PUBLIC_CERTS_URL - url hosting jwt public key
REDASH_JWT_AUTH_AUDIENCE - jwt audience
REDASH_JWT_AUTH_ALGORITHMS - HS256,RS256,ES256
REDASH_JWT_AUTH_PARAM_NAME - url param for jwt token
```

Generating JWT public and private keys
```
ssh-keygen -t rsa -b 4096 -m PEM -f jwtRS256.key
openssl rsa -in jwtRS256.key -pubout -outform PEM -out jwtRS256.key.pub
```

Creating JWT token via private key
```javascript
const jwt = require('jsonwebtoken');
const fs = require('fs');
const privateKey = fs.readFileSync('<path to private_key>');
const token = jwt.sign(
    {email: '<email>'}, //email is a necessary field
    {key: privateKey, passphrase: <passphrase>},
    {algorithm: 'RS256', audience:<audience>', issuer:'<issuer>', expiresIn:"7d" });
console.log(token);
```

Hosting jwt public key
```javascript
const http = require('http');

const hostname = '0.0.0.0';
const port = 3000;

const server = http.createServer((req, res) => {
    res.statusCode = 200;
    res.setHeader('Content-Type', 'application/json');
    const key = ['<public_key>']
    res.end(JSON.stringify(
        key
    ));
});

server.listen(port, hostname, () => {
    console.log(`Server running at http://${hostname}:${port}/`);
});
```

To add support for viewer group, run the query
```
INSERT into groups(org_id, type, name, permissions, created_at) VALUES (1, 'regular', 'viewer', '{list_dashboards}', NOW());
```
