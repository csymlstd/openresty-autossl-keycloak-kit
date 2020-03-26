# openresty-autossl-keycloak-kit
This docker-compose stack includes a base config to start an openresty (nginx) server with PHP that will auto generate SSL certificates on the first request. A default nginx site is also enabled to proxy Keycloak, an open-source authentication system.

# How it works

**Containers**

- Keycloak (auth system to protect by username/password with role-based authorization)
- Postgres (stores data from keycloak, can be swapped with any SQL db of your choice)
- PHP (a simple implementation of php)
- Openresty (web server built on nginx and lua, allows auto-ssl and openid)

**NGINX Conf**

Since openresty is built on nginx, you can bring any existing conf in with simple adjustments to enable auto-ssl and openid login capabilities.

There are three examples in the conf folder:

- `default.conf.example` includes a basic HTTPS server that forwards php files to the php container.

- `default-protected.conf.example` includes the same HTTPs server with a lua access block that will force users to login through Keycloak to access. This is similar to basic auth, but backed by a database.


- `default-role-based.conf.example` includes the same HTTPs server with a lua access block. They will only be permitted access if they have the set Realm Role otherwise they will see a 403 error page.

- `keycloak.conf` is a default config to access keycloak when logging into sites

```
NOTE: You will need to replace mydomain.com with your domain in nginx.conf and site confs.
```
**Auto SSL**

`nginx.conf` includes the default server to process certificate requests.

Starting at line 48, is a function that allows you to use Lua to decide if the requesting domain is permitted or not. You can use a redis to http connector in this block to check external db's. Resources are linked below.

**Keycloak**

Documentation for Keycloak is slim when it comes to implementing role based authentication. I have found a few community sites with some examples that I have spliced together to get this implementation.

There are a few settings you will need to change to get this to work.

Realm

- For basic login protection
  - Your realm should be all set by default

- For role-based protection
  - Go to Roles in the sidebar
  - Click Add Role under the Realm Roles tab
  - Save Role
  - Go to Users, and edit a user
  - Go to Role Mappings tab
  - Select your new role from the Available Roles column and Add selected.
  - This user should now be able to access your client using the `default-role-based.conf.example` nginx conf.

Clients

- For basic login protection
  - Client Protocol: openid-connect
  - Access Type: Confidential
  - Standard Flow Enabled: ON
  - Save, and go to Credentials tab:
  - Copy Client Secret and paste it in your nginx conf along with your Client ID from the settings tab.

- For role-based protection
  - Go to Mappers tab
  - Click Create
  - Name: Roles
  - Mapper Type: User Realm Role
  - Multivalued: ON
  - Add to ID token: ON
  - Add to access token: ON
  - Add to userinfo: ON
  - Save

# Resources
- [docker-openresty](https://hub.docker.com/r/openresty/openresty)
- [docker-keycloak](https://hub.docker.com/r/jboss/keycloak)
- [lua-resty-auto-ssl](https://github.com/auto-ssl/lua-resty-auto-ssl)
- [lua-resty-openidc](https://github.com/zmartzone/lua-resty-openidc)
- [Securing Applications and Services Guide - Keycloak](https://www.keycloak.org/docs/latest/securing_apps/index.html)
- [How we generate and renew SSL certs for arbitrary custom domains using LetsEncrypt! - sandeep.dev](https://sandeep.dev/how-we-generate-and-renew-ssl-certs-for-arbitrary-custom-domains-using-letsencrypt-cjtk0utui000c1cs1f7y9ua5n) - How to authorize domains from http
- [How i setup custom domain SSL with Let's Encrypt for changelog pages - changelogfy.com](https://changelogfy.com/blog/how-i-setup-custom-domain-ssl-with-lets-encrypt-for-changelog-pages/) - How to authorize domains from redis