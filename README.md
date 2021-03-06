# Mobile Flows connectors common libraries

## Overview
The project is a suite of commonly used utility functions for anyone developing a Mobile Flows connector on Express Node.js. 
As a connector developer you can use the functions and considerably reduce the overall development time. In addition to that, 
it helps to maintain a similar pattern in the connector code and avoid making mistakes. 

You can load this module wherever its required and use necessary functions. 
```
const mfCommons = require('@vmw/mobile-flows-connector-commons')
```

For a detailed, language-neutral, specification for how to develop connectors, 
please see the [Card Connectors Guide](https://github.com/vmware-samples/card-connectors-guide).

## Installation
This module is available through the NPM registry.
```
$ npm install @vmw/mobile-flows-connector-commons
```

## Functions available for development

### validateAuth(mfPublicKeyUrl)
This validates Authorization JWT from Mobile Flows. 
Function takes public key URL of Mobile Flows server. Returns a function to be used as a middleware 
for all protected APIs.

If the validation fails, request is rejected as 401 with a message.

If the validation succeeded, it adds some local variables at `res.locals.mfJwt`.
```$xslt
res.locals.mfJwt.tenantId = 'Mobile Flows tenant id'
res.locals.mfJwt.username = 'Username'
res.locals.mfJwt.email = 'User email id'
res.locals.mfJwt.idmDomain = 'User IDM domain id'
res.locals.mfJwt.decoded = 'All decoded params of the JWT'
```

Example
```
app.use(['/api/*'], mfCommons.validateAuth('https://prod.hero.vmwservices.com/security/public-key'))
```

### getConnectorBaseUrl(req)
It takes Express request object and returns the connector's base URL. In most cases its only used to 
generate URLs in the connector discovery, for Mobile Flows server to call into the connector.

The function identifies the original host requested by the caller using x-forwarded headers.

Example
```
const baseUrl = mfCommons.getConnectorBaseUrl(req)
```

### mfRouting.addContextPath(req, res, next)
Use this function as a middleware for all object and action APIs. 
It reads Mobile Flows routing headers and resolves it to the correct value when the connector is hosted behind a path based proxy. 
This is necessary for most objects that produce actions and for actions that produce other actions. We recommend always adding this.
Properties in res.locals will be updated based on the type of request.

For object requests - `res.locals.mfRoutingPrefix`

For action requests - `res.locals.mfRoutingTemplate`

Example
```
app.use(['/api/*'], mfCommons.mfRouting.addContextPath)
```

### handleXRequestId(req, res, next)
Use this function as a middleware for all connector APIs to generate better logs and help in debugging a request. 
If the http call comes in with an `x-request-id` header, then this middleware will set it in `res.locals.xRequestId` 
to be put in logs and for the connector developer to send out in further http calls as needed. 
If there isn't an `x-request-id` header value, then this middleware will generate one for you.

The xRequestId in the logs allows the person debugging to correlate all the logs with the original http request.

Example
```
app.use('/*', mfCommons.handleXRequestId)
```

### readBackendBaseUrl(req, res, next)
This function can be used as a middleware for all connector APIs. If Mobile Flows admin configures a 
backend base URL for the connector, then this function sets it in `res.locals.baseUrl`.

Adding the middleware allows you to read the base URL from `res.locals.baseUrl`, whenever you need to
make an API call to backend.

Example
```
app.use('/*', mfCommons.readBackendBaseUrl)
```

### logReq(res, format, ...args)
It can be used to log a message along with some useful properties related to the current request.

Example
```
mfCommons.logReq(res, 'Created ticket: %s', ticketId)

// [req: req-id-1] [t: tenant123] [u: shree] [base: https://backend.com] Created ticket: TKT-5
```

### log(format, ...args)
It can be used to log a message outside the context of a request.

Example
```
mfCommons.log('Sent samples for analytics.')
```

## Functions available for testing

## mockMfServer
This is a dummy server to mimic some properties of the Mobile Flows server. It will be useful for unit testing purposes.
Its public key is available at `/security/public-key`.

### start(port)
Call the method to start the dummy server to listen at the specified port on the localhost. You can have it running
for a suite of unit tests, instead of starting it once per test.

Example
```
mfCommons.mockMfServer.start(5000)
```

### stop(function)
Stop the dummy server after your unit tests. It optionally takes a callback function that will be invoked by this library.

Example
```
mfCommons.mockMfServer.stop()
```
```
mfCommons.mockMfServer.stop(function)
```

### getMfToken({ username, audience })
This function returns a JWT for the connector authorization.  
It is similar to the one generated by actual Mobile Flows server. 
You can use this function for specific user and connector URL being tested (audience URL).

Example
```
const mfToken = mfCommons.getMfToken({ username: 'shree', audience: `${CONNECTOR_URL}/api/actions/file-ticket`})
```


## Contributing

The mobile-flows-connectors-common-libraries project team welcomes contributions from the community. 
Before you start working with mobile-flows-connectors-common-libraries, please read 
our [Developer Certificate of Origin](https://cla.vmware.com/dco). All contributions to this repository must be
signed as described on that page. Your signature certifies that you wrote the patch or have the right to pass it on
as an open-source patch. For more detailed information, refer to [CONTRIBUTING.md](CONTRIBUTING.md).

## License

Mobile Flows connectors common libraries under the [BSD 2 license](LICENSE.txt)

