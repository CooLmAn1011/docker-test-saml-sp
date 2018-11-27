# Docker Test SAML 2.0 Identity Provider (IdP)

[![DockerHub Pulls](https://img.shields.io/docker/pulls/kristophjunge/test-saml-idp.svg)](https://hub.docker.com/r/kristophjunge/test-saml-idp/) [![DockerHub Stars](https://img.shields.io/docker/stars/kristophjunge/test-saml-idp.svg)](https://hub.docker.com/r/kristophjunge/test-saml-idp/) [![GitHub Stars](https://img.shields.io/github/stars/kristophjunge/docker-test-saml-idp.svg?label=github%20stars)](https://github.com/kristophjunge/docker-test-saml-idp) [![GitHub Forks](https://img.shields.io/github/forks/kristophjunge/docker-test-saml-idp.svg?label=github%20forks)](https://github.com/kristophjunge/docker-test-saml-idp) [![GitHub License](https://img.shields.io/github/license/kristophjunge/docker-test-saml-idp.svg)](https://github.com/kristophjunge/docker-test-saml-idp)

![Seal of Approval](https://raw.githubusercontent.com/kristophjunge/docker-test-saml-idp/master/seal.jpg)

Docker container with a plug and play SAML 2.0 Identity Provider (IdP) and Service provider (SP) for development and testing.

Built with [SimpleSAMLphp](https://simplesamlphp.org). Based on official PHP7 Apache [images](https://hub.docker.com/_/php/).

**Warning!**: Do not use this container in production! The container is not configured for security and contains static user credentials and SSL keys.

SimpleSAMLphp is logging to stdout on debug log level. Apache is logging error and access log to stdout.

The contained version of SimpleSAMLphp is 1.16.2.

## Usage

Build the Container:

```
docker build -t chrisusick/test-saml .
```

## As an IDP 

For running SimpleSAMLPHP as an IDP to test a SP: 

```
docker run --name=some-test-saml-idp \
-p 8080:80 \
-p 8443:443 \
-e SIMPLESAMLPHP_SP_ENTITY_ID=http://app.example.com \
-e SIMPLESAMLPHP_SP_ASSERTION_CONSUMER_SERVICE=http://localhost/simplesaml/module.php/saml/sp/saml2-acs.php/test-sp \
-e SIMPLESAMLPHP_SP_SINGLE_LOGOUT_SERVICE=http://localhost/simplesaml/module.php/saml/sp/saml2-logout.php/test-sp \
-e SIMPLESAMLPHP_TRUSTED_DOMAINS='localhost:3000'
-d chrisUsick/test-saml
```

There are two static users configured in the IdP with the following data:

| UID | Username | Password | Group | Email |
|---|---|---|---|---|
| 1 | user1 | user1pass | group1 | user1@example.com |
| 2 | user2 | user2pass | group2 | user2@example.com |

However you can define your own users by mounting a configuration file:

```
-v /users.php:/var/www/simplesamlphp/config/authsources.php
```

You can access the SimpleSAMLphp web interface of the IdP under `http://localhost:8080/simplesaml`. The admin password is `secret`.

## As an SP

For running SimpleSAMLPHP as a SP to test an IDP: 

```
docker run --name=some-test-saml-sp \
-p 8080:80 \
-p 8443:443 \
-e SIMPLESAMLPHP_SP_ENTITY_ID=http://app.example.com \
-e SIMPLESAMLPHP_IDP_METADATA_URL=http://my-saml-idp.com/saml/metadata
-e SIMPLESAMLPHP_IDP_SSO_URL=http://my-saml-idp.com/saml/SSOService
-e SIMPLESAMLPHP_IDP_SLO_URL=http://my-saml-idp.com/saml/SLOService
-e SIMPLESAMLPHP_IDP_CERT_FINGERPRINT=1234123412341234
-d chrisUsick/test-saml
```

## Test the Identity Provider (IdP) with the SP

To ensure that the IdP works you can use SimpleSAMLphp as test SP.

For this test the following is assumed:
- The entity id of the SP is `http://app.example.com` (`SIMPLESAMLPHP_SP_ENTITY_ID`).
- The local development URL of the SP is `http://localhost:3000` (when running the SP use port 3000).
- The local developemnt URL of the IdP is `http://localhost:8080` (when running the idP use port 8080).

Set the rest of the environment variables as follows:  
Then start the 2 containers:

```
docker run --name=simplesaml-idp \
-p 8080:80 \
-p 8443:443 \
-e SIMPLESAMLPHP_SP_ENTITY_ID=http://app.example.com \
-e SIMPLESAMLPHP_SP_ASSERTION_CONSUMER_SERVICE=http://localhost:3000/simplesaml/module.php/saml/sp/saml2-acs.php/test-sp \
-e SIMPLESAMLPHP_SP_SINGLE_LOGOUT_SERVICE=http://localhost:3000/simplesaml/module.php/saml/sp/saml2-logout.php/test-sp \
-e SIMPLESAMLPHP_TRUSTED_DOMAINS=localhost:3000,localhost:8080 \
-e SIMPLESAMLPHP_BASEURLPATH=http://localhost:8080/simplesaml/ \
-d chrisusick/test-saml
```

```
docker run --name=simplesaml-sp \
-p 3000:80 \
-e SIMPLESAMLPHP_SP_ENTITY_ID=http://app.example.com \
-e SIMPLESAMLPHP_IDP_METADATA_URL=http://localhost:8080/simplesaml/saml2/idp/metadata.php \
-e SIMPLESAMLPHP_IDP_SSO_URL=http://localhost:8080/simplesaml/saml2/idp/SSOService.php \
-e SIMPLESAMLPHP_IDP_SLO_URL=http://localhost:8080/simplesaml/saml2/idp/SingleLogoutService.php \
-e SIMPLESAMLPHP_IDP_CERT_FINGERPRINT=119b9e027959cdb7c662cfd075d9e2ef384e445f \
-e SIMPLESAMLPHP_TRUSTED_DOMAINS=localhost:3000,localhost:8080 \
-e SIMPLESAMLPHP_BASEURLPATH=http://localhost:3000/simplesaml/ \
-d chrisusick/test-saml
```

The entity id is only the name of SP and the contained URL wont be used as part of the auth mechanism.

Initiate the login from the development SP under `http://localhost:3000/simplesaml`.

Click under `Authentication` > `Test configured authentication sources` > `test-sp` and login with one of the test credentials.

## License

This project is licensed under the MIT license by Kristoph Junge.
