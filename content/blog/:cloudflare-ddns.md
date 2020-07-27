---
title: "Using Cloudflare as a Dynamic DNS (DDNS) provider"
date: 2020-07-26T17:46:07+01:00
description: "How to use Cloudflare as a DDNS provider."
draft: false
tags: ["Dynamic DNS", "DDNS", "Cloudflare", "Docker", "Containers"]
math: false
toc: false
---

This blog post will show you how to use Cloudflare as a Dynamic DNS (DDNS) provider.

## What is DDNS?

> Dynamic DNS (DDNS) is a service that keeps the Domain Name System (DNS) updated with a web property’s correct IP address.
> <cite>Cloudflare glossary: Dynamic DNS [^1]</cite>

Essentially, DDNS allows you to automatically update your DNS records when a change is detected to your public IP address.

## DDNS Common Use Case

Many Internet Service Providers (ISPs) do not provide their customers with a static public IP address. This can cause issues for people who self-host applications from home and have DNS records pointing to their ISP provided public IP address.

Say you are hosting some applications from home and have DNS records pointing to your home's public IP address. What if you router suddenly rebooted due to a software or hardware issue? 

Your ISP **may** assign you with a new public IP address causing your applications to be inaccessible. This is an example of where DDNS could help.

With DDNS, you can ensure your DNS records are automatically kept up to date when your home's public IP address changes :thumbsup:

## Other DDNS providers

Cloudflare is not the only option for DDNS. There are many others including:

* [DuckDNS](https://www.duckdns.org/)

* [NoIP](https://www.noip.com/)

* [FreeDNS](https://freedns.afraid.org/)

However, for this blog post, we are obviously focusing on using Cloudflare :smile:

## Using Cloudflare as a DDNS provider

We will be using the [docker-cloudflare-ddns](https://github.com/oznu/docker-cloudflare-ddns) docker container created by [oznu](https://github.com/oznu) to use Cloudflare as a DDNS provider.

### Prerequisites

1. A domain managed with Cloudflare.

2. Docker installed on a server which can reach the Cloudflare API: `https://api.cloudflare.com/client/v4/`.

3. A Cloudflare API token (see below).

#### Creating a Cloudflare API Token

To obtain a Cloudflare API token:

1. Log in to the [Cloudflare dashboard](https://dash.cloudflare.com/profile/api-tokens) to manage your API tokens.

2. Click **Create Token**.

3. Under *Custom Token*, click **Get started**.

4. Provide a *Name* for the API token.

5. Under *Permissions*, grant the following to the API token:

    - Zone / Zone Settings / Read

    - Zone / Zone / Read

    - Zone / DNS / Edit

    ![Cloudflare API token permissions](/images/api-token-permissions.png)

6. Under *Zone Resources*, include the zone (domain) you want to grant the API token permissions to edit.

![Cloudflare API token summary](/images/full-api-token.png)

7. Click **Continue to summary**.

8. Review the API token configuration and click **Create Token**.

### Using the docker-cloudflare-ddns container

The docker-cloudflare-ddns container **requires** the following environment variables:

- `API_KEY`: The API token (created above) used to update a DNS record for the granted zone.
    
- `ZONE`: The DNS zone which updates should be applied to.

    - Example: `danielbrennand.com`.

Notable optional environment variables:

- `SUBDOMAIN`: The subdomain in the `ZONE` to update a DNS record for.

    - **NOTE**: If absent, the root zone (Example: `danielbrennand.com`) DNS record will be updated.

- `PROXIED`: If `true`, the proxy status on the DNS record will be set to proxy traffic through Cloudflare.

- `RRTYPE`: Defaults to: `A` for IPv4. To use IPv6, you can alter the value to `AAAA`. 

    - **NOTE**: To use IPv6 (`AAAA`), Docker must have [IPv6 enabled](https://docs.docker.com/config/daemon/ipv6/).

- `CRON`: The interval to check if the DNS record needs to be updated. Defaults to: `*/5 * * * *` (5 minutes).

    - For useful cron examples, see [here](https://crontab.guru/examples.html).

Other environment variables can be found [here](https://github.com/oznu/docker-cloudflare-ddns#optional-parameters).

**NOTE**: If the DNS record to be updated does not exist, it will be created automatically.

#### Deployment Examples

I will be using my domain in the examples below. Make sure you alter it to your domain! :laughing:

##### Example 1:

Update the DNS record for the subdomain `freshrss` in the zone `danielbrennand.com` with traffic proxied through Cloudflare.

```bash
docker run -d --name cloudflare \
  -e API_KEY="Insert your Cloudflare API token here." \
  -e ZONE="danielbrennand.com" \
  -e SUBDOMAIN="freshrss" \
  -e PROXIED="true" \
  oznu/cloudflare-ddns
```

##### Example 2:

Update the DNS record for the root zone `danielbrennand.com`.

```bash
docker run -d --name cloudflare \
  -e API_KEY="Insert your Cloudflare API token here." \
  -e ZONE="danielbrennand.com" \
  oznu/cloudflare-ddns
```

##### Example 3:

Update the DNS record with the public IPv6 address for the subdomain `freshrss` in the zone `danielbrennand.com` with traffic proxied through Cloudflare.

**NOTE**: Remember, for this example to work, you must have [IPv6 enabled](https://docs.docker.com/config/daemon/ipv6/) in Docker.

```bash
docker run -d --name cloudflare \
  -e API_KEY="Insert your Cloudflare API token here." \
  -e ZONE="danielbrennand.com" \
  -e SUBDOMAIN="freshrss" \
  -e RRTYPE="AAAA" \
  -e PROXIED="true" \
  oznu/cloudflare-ddns
```

##### Example 4:

Update the DNS record for the subdomain `freshrss` in the zone `danielbrennand.com` on an interval of 1 hour.

```bash
docker run -d --name cloudflare \
  -e API_KEY="Insert your Cloudflare API token here." \
  -e ZONE="danielbrennand.com" \
  -e SUBDOMAIN="freshrss" \
  -e CRON="0 * * * *" \
  oznu/cloudflare-ddns
```

If successful, observing the container logs using `docker logs cloudflare` will look similar to the following output:

The following logs are from Example 1.

```
[cont-init.d] executing container initialization scripts...
[cont-init.d] 30-cloudflare-setup: executing...
DNS Zone: danielbrennand.com (619du4059a9354492r54e6702834ao62)
DNS record for 'freshrss.danielbrennand.com' was not found in danielbrennand.com zone. Creating now...
DNS Record: freshrss.danielbrennand.com (bde8a4987aj8ede16d62d7479d967f3l)
[cont-init.d] 30-cloudflare-setup: exited 0.
[cont-init.d] 50-ddns: executing...
No DNS update required for freshrss.danielbrennand.com (Public IPv4 address here).
```

### Multiple Subdomains

Your probably thinking, what if I wanted multiple subdomains pointing to my public IP address. Do I have to run this container for each of them!?

Nope! :smile:

You can create CNAME records pointing to the A record referencing your public IP address.

Continuing with the example subdomain above (`freshrss`), if I had a new application which I wanted to host at the subdomain `foo.danielbrennand.com`. I would create the following CNAME record:

- **Type**: CNAME

- **Name**: foo

- **Target**: freshrss.@

- **TTL**: Auto

- **Proxy Status**: Proxied

I hope you found this blog post useful in showing you how Cloudflare can be used as a DDNS provider :smiley: Until next time! :wave:

## References

[^1]: https://www.cloudflare.com/learning/dns/glossary/dynamic-dns/
