# CAS OIDC demo

This overlay enables the CAS OpenID Connect provider and loads its sample client
from `src/main/resources/services/SampleOidc-1000.json`.

| Client setting | Value |
| --- | --- |
| Client ID | `sample-oidc` |
| Client secret | `sample-oidc-secret` |
| Redirect URI | `http://localhost:8080/callback` |
| Grant / response type | `authorization_code` / `code` |
| Scope | `openid` |
| Default issuer | `https://localhost:8443/cas/oidc` |
| Default discovery URL | `https://localhost:8443/cas/oidc/.well-known/openid-configuration` |

Use discovery to configure the client's authorization, token, and userinfo
endpoints. After signing in through CAS and exchanging the authorization code,
call the discovered userinfo endpoint with the access token to obtain the claims.
The callback is a registration for an external sample application; this repository
does not implement that application.

The stub attribute repository in `src/main/resources/application.yml` supplies
the same fixed name, email, username, organization, and roles to every authenticated
user. The client's `ReturnAllAttributeReleasePolicy` releases all resolved
attributes, including custom attributes such as `organization` and `roles`.
Keeping the client's scopes limited to `openid` selects CAS's scope-free release
behavior. Adding standard scopes such as `profile` and `email` changes how CAS
constructs and filters the release policy. See the
[CAS claim release documentation](https://apereo.github.io/cas/development/authentication/OIDC-Authentication-Claims-Release.html).

Build with `./gradlew clean build`. For Heroku, set `CAS_SERVER_NAME` to the public
HTTPS origin of the deployment, without a trailing slash; the issuer then becomes
`https://<app-host>/cas/oidc`. The existing `Procfile` supplies the HTTP port and
disables local TLS termination for Heroku.

CAS generates its signing keys at `./oidc.jwks` by default. Override
`OIDC_JWKS_FILE` with a writable file resource, such as
`file:/etc/cas/config/oidc.jwks`, to use another location. Heroku's local filesystem
is ephemeral, so keys generated there will change when the filesystem is replaced;
use persistent signing-key storage when keys must survive deployments or be shared
between instances.

The checked-in client secret, fixed attributes, and release-all policy are demo
settings. Update the client registration and stub values for your application.
