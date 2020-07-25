---
title: "Using Caddy v2's reverse proxy feature with a domain managed with Cloudflare!"
date: 2020-07-21T21:08:07+01:00
description: "How to use Caddy v2 as a reverse proxy with a Cloudflare managed domain."
draft: false
tags: ["Caddy", "Reverse-Proxy", "Cloudflare", "Docker", "Containers"]
math: false
toc: false
---

[Caddy](https://caddyserver.com/) version [2.0.0](https://github.com/caddyserver/caddy/releases/tag/v2.0.0) released on May 4th 2020. Caddy brands itself as "The Ultimate Server" with [functionality](https://caddyserver.com/docs/) including a web server, reverse proxy, automatic certificate renewal, automatic HTTPS and more!

I have to say, it's pretty awesome! :thumbsup:

This blog post will show you how to use the Caddy v2 reverse proxy feature with a domain managed with Cloudflare. So, lets jump in! :smile:

## Caddyfile

Caddy uses a [Caddyfile](https://caddyserver.com/docs/caddyfile) for it's configuration.

Two main Caddyfile concepts to understand are [**blocks**](https://caddyserver.com/docs/caddyfile/concepts#blocks) and [**directives**](https://caddyserver.com/docs/caddyfile/concepts#directives).

### Blocks

A Caddyfile block contains configuration for a site. Blocks are declared using curly braces:

```
example.com {
    ...
}
```

One unique block worth mentioning is the [global options block](https://caddyserver.com/docs/caddyfile/options). This block **must** be defined at the top of the Caddyfile and allows you to modify options which apply globally to Caddy. Two usage examples for this block are altering the **acme_ca** (ACME CA's directory) to the [Let's Encrypt staging endpoint](https://letsencrypt.org/docs/staging-environment/) or the **email** option which is used when creating the ACME account.

To use the global options block along with a site block, your Caddyfile would look similar to the following:

```caddyfile
{
    email example@example.com
    # This is a valid comment in a Caddyfile :-)
    # This acme_ca option tells Caddy to use the Let's Encrypt staging endpoint
    # Remove when transitioning to a production environment
    acme_ca https://acme-staging-v02.api.letsencrypt.org/directory
}
# Site block below
example.com {
    ...
}
```

### Directives and Subdirectives

Caddyfile directives customise how a site is served by Caddy and **must** be declared within a site block. A subdirective provides additional configuration for a directive. See the Caddy [documentation](https://caddyserver.com/docs/caddyfile/directives) for a full list of directives and their respective subdirectives.

The directive we are interested in this blog post is the [reverse_proxy](https://caddyserver.com/docs/caddyfile/directives/reverse_proxy) directive.

Adding to the previous example, usage of the reverse_proxy directive is as follows:

```caddyfile
{
    email example@example.com
    # This acme_ca option tells Caddy to use the Let's Encrypt staging endpoint
    # Remove when transitioning to a production environment
    acme_ca https://acme-staging-v02.api.letsencrypt.org/directory
}
# Site block below
example.com {
    # Using the reverse_proxy directive within a site block
    reverse_proxy example:80
}
```

## Using the Caddy v2 Docker container with a Cloudflare managed domain

Right, enough of the boring theory! Onto an actual example.

I have created a [Github repository](https://github.com/dbrennand/caddy-cloudflare-docker-compose) which uses docker-compose to deploy the Caddy v2 container (including the [Cloudflare module](https://github.com/caddy-dns/cloudflare)) and [freshrss](https://www.freshrss.org/) as an example application.

We will be using the [DNS-01 challenge](https://caddyserver.com/docs/automatic-https#dns-challenge) type to request a certificate for the subdomain `freshrss` under your Cloudflare managed apex domain.

### Prerequisites

1. A domain managed with Cloudflare

2. **SSL/TLS encryption mode** set the **Full (strict)**

   - This ensures that clients don't encounter [infinite redirects](https://caddy.community/t/infinite-redirection/3230/5), they connect via HTTPS and ensures the Let's Encrypt certificate on the server is valid.

3. A server with ports 80 and 443 accessible to the internet

   - **NOTE**: The DNS-01 challenge type doesn't require any open ports however, in order to access any applications from the internet, we need these ports exposed.

4. A server with Docker, docker-compose and git installed

5. An A record configured in Cloudflare with the following values:

   - **Type**: A

   - **Name**: freshrss

   - **Content**: The IPv4 address of your server

   - **TTL**: Auto

   - **Proxy Status**: Proxied

6. A [Cloudflare API token](https://github.com/libdns/cloudflare#authenticating) (**NOT** an API key!) with the following permissions:

   - Zone / Zone / Read

   - Zone / DNS / Edit

### Deployment

1. Clone the caddy-cloudflare-docker-compose repository and change directory: `git clone https://github.com/dbrennand/caddy-cloudflare-docker-compose.git; cd caddy-cloudflare-docker-compose`.

2. Make a copy of the ExampleCaddyfile called Caddyfile: `cp ExampleCaddyfile Caddyfile`.

3. Modify the following lines in the Caddyfile:

    - `email example@example.com`: Alter `example@example.com` to your email address.

    - `subdomain.example.com`: Modify this to be your domain. For example: `freshrss.yourdomain.com`.

**NOTE**: I have provided some snippets and comments in the ExampleCaddyfile which I hope you will find useful.

4. Modify the `CLOUDFLARE_API_TOKEN` environment variable in the docker-compose.yaml file with your Cloudflare API token:

```dockerfile
environment:
      CLOUDFLARE_API_TOKEN: "Insert your Cloudflare API token here."
```

5. Modify the `PUID`, `PGID` and `TZ` environment variables in the docker-compose.yaml file (if required).

6. Start the Caddy and freshrss containers using: `docker-compose up -d`.

View the Caddy container logs using: `docker logs caddy`. If everything is configured correctly, you will see something similar to the following output in the Caddy container logs:

```
[INFO] [freshrss.yourdomain.com] acme: use dns-01 solver
[INFO] [freshrss.yourdomain.com] acme: Preparing to solve DNS-01
[INFO] [freshrss.yourdomain.com] acme: Trying to solve DNS-01
[INFO] [freshrss.yourdomain.com] acme: Checking DNS record propagation using [127.0.0.11:53]
[INFO] Wait for propagation [timeout: 1m0s, interval: 2s]
[INFO] [freshrss.yourdomain.com] acme: Waiting for DNS record propagation.
[INFO] [freshrss.yourdomain.com] The server validated our request
[INFO] [freshrss.yourdomain.com] acme: Cleaning DNS-01 challenge
[INFO] [freshrss.yourdomain.com] acme: Validations succeeded; requesting certificates
[INFO] [freshrss.yourdomain.com] Server responded with a certificate.
[INFO][freshrss.yourdomain.com] Certificate obtained successfully
[INFO][freshrss.yourdomain.com] Obtain: Releasing lock
```

:star: Congratulations! You have successfully configured Caddy to obtain a valid certificate from Let's Encrypt for the subdomain `freshrss` under your Cloudflare apex domain :smile:

Navigate to `freshrss.yourdomain.com` in your browser and you will see the setup wizard for freshrss!

In my next blog post, I will cover how you can use Cloudflare as a Dynamic DNS (DDNS) provider.
