# Kong Upstream JWT Plugin
## Overview
This plugin will add a signed JWT into the HTTP Header `JWT` of proxied requests through the Kong gateway. The purpose of this, is to provide means of _Authentication_, _Authorization_ and _Non-Repudiation_ to API providers (APIs for which Kong is a gateway).

In short, API Providers need a means of cryptographically validating that requests they receive were A. proxied by Kong, and B. not tampered with during transmission from Kong -> API Provider. This token accomplishes both as follows:
1. **Authentication** & **Authorization** - Provided by means of JWT signature validation. The API Provider will validate the signature on the JWT token (which is generating using Kong's RSA x509 private key), using Kong's public key. This public key can be maintained in a keystore, or sent with the token - provided API providers validate the signature chain against their truststore.
2. **Non-Repudiation** - SHA256 is used to hash the body of the HTTP Request Body, and the resulting digest is included in the `payloadhash` element of the JWT body. API Providers will take the SHA256 hash of the HTTP Request Body, and compare the digest to that found in the JWT. If they are identical, the request remained intact during transmission.

## Supported Kong Releases
Kong >= 0.12.x 

## Installation
Recommended:
```
$ luarocks install kong-upstream-jwt
```
Other:
```
$ git clone https://github.com/Optum/kong-upstream-jwt.git /path/to/kong/plugins/kong-upstream-jwt
$ cd /path/to/kong/plugins/kong-upstream-jwt
$ luarocks make *.rockspec
```

## Configuration
The plugin requires that Kong's private key be accessible in order to sign the JWT. [We also include the x509 cert in the `x5c` JWT Header for use by API providers to validate the JWT](https://tools.ietf.org/html/rfc7515#section-4.1.6). We access these via Kong's overriding environment variables `KONG_SSL_CERT_KEY` for the private key as well as `KONG_SSL_CERT_DER` for the public key. The first contains the path to your .key file, the second specifies the path to your public key in DER format .cer file.

If not already set, these can be done so as follows:
```
$ export KONG_SSL_CERT_KEY="/path/to/kong/ssl/privatekey.key"
$ export KONG_SSL_CERT_DER="/path/to/kong/ssl/kongpublickey.cer"
```

**One last step** is to make the environment variables accessible by an nginx worker. To do this, simply add these line to your _nginx.conf_
```
env KONG_SSL_CERT_KEY;
env KONG_SSL_CERT_DER;
```


Feel free to open issues, or refer to our [Contribution Guidelines](https://github.com/Optum/kong-upstream-jwt/blob/master/CONTRIBUTING.md) if you have any questions.
