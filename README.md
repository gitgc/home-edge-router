home-edge-router
================

This repository contains the configuration for my home edge/ingress load balancer, built on [Caddy](https://caddyserver.com/) and [`caddy-security`](https://github.com/greenpau/caddy-security). This provides HTTPS/SSL termination with valid letsencrypt.org certificate for private IP. Additionally, it protects access to its upstream services via oauth using the `caddy-security` Google SSO integration. Services can only be accessed if user can sign-in successfully via Google SSO using an email on my home domain. If user authenticates successfully, a `JWT` is issued granting access across the load balancer.

Takes advantage of my own [`home-proxy-webserver`](https://github.com/gitgc/home-proxy-webserver) Caddy container that has the Digital Ocean DNS and `caddy-security` extensions pre-installed.

Requirements
------------
* Docker

Steps
-----
Before deploying, all the environment variables declared in the `.env` file with values of `CHANGE_ME` must be replaced with valid values:

| Environment Variable           | Description                                                 |
| ------------------------------ | ----------------------------------------------------------- |
| `CADDY_DOMAIN`                 | The domain (must be under control in Digital Ocean DNS)     |
| `CADDY_HEALTH_CHECK_URI`       | The endpoint to use for Caddy health checking               |
| `CADDY_ADMIN`                  | The email address of the authentication admin user          |
| `DO_AUTH_TOKEN`                | The Digital Ocean API Key                                   |
| `GOOGLE_CLIENT_ID`             | The Google OAuth client ID                                  |
| `GOOGLE_CLIENT_SECRET`         | The Google OAuth client secret                              |
| `JWT_SHARED_KEY`               | A random unique value for the authentication JWT shared key |
| `FRIGATE_UPSTREAM_HOST`        | The upstream hostname for the Frigate server                |
| `SCRYPTED_UPSTREAM_HOST`       | The upstream hostname for the Scrypted server               |
| `HOME_ASSISTANT_UPSTREAM_HOST` | The upstream hostname for the Home Assistant server         |
| `UNIFI_UPSTREAM_HOST`          | The upstream IP for the Unifi OS Console                    |

Naturally, the `_UPSTREAM_HOST` machines should be live and accessible first!

Then:

```console
    $ docker-compose up -d
```

After deploying, the following OAuth/SSL protected end points should be available:
 
| Endpoint                       | Description                                                 |
| ------------------------------ | ----------------------------------------------------------- |
| https://auth.$CADDY_DOMAIN     | OAuth user management console                               |
| https://network.$CADDY_DOMAIN  | Unifi OS console                                            |
| https://home.$CADDY_DOMAIN     | Home Assistant dashboard                                    |
| https://nvr.$CADDY_DOMAIN      | Frigate NVR                                                 |
| https://scrypted.$CADDY_DOMAIN | Scrypted console                                            |
