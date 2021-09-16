---
title: "Using Cloudflare as a Dynamic DNS (DDNS) provider"
date: 2021-09-16T18:13:30+01:00
description: "How to use Cloudflare as a DDNS provider."
draft: false
tags: ["Dynamic DNS", "DDNS", "Cloudflare", "Docker", "Containers"]
showToc: true
---

Hey there! :wave:

In this blog post, I will be showing you how to use Cloudflare as a Dynamic DNS (DDNS) provider.

## What is DDNS?

> Dynamic DNS (DDNS) is a service that keeps the Domain Name System (DNS) updated with a web property’s correct IP address.
>
> <cite>Cloudflare glossary: Dynamic DNS [^1]</cite>

Essentially, DDNS allows you to automatically update your DNS records when a change is detected to your public IP address.

## DDNS Use Case

Many Internet Service Providers (ISPs) do not provide a static IP address with their regular consumer plans. This often causes issues for those of us that enjoy self-hosting applications from home, and have DNS records for a domain pointing to your public IP address.

Say you're self-hosting some applications from home and have DNS records pointing to your home's public IP address. What if your router suddenly rebooted due to a software or hardware issue?

Your ISP **may** assign you with a new public IP address causing your applications to be inaccessible. This is an example of where DDNS could help.

With DDNS, you can ensure your DNS records are automatically kept up to date when your home's public IP address changes :thumbsup:

## Other DDNS providers

Cloudflare is not the only option for DDNS. There are many others including:

* [DuckDNS](https://www.duckdns.org/)

* [NoIP](https://www.noip.com/)

* [FreeDNS](https://freedns.afraid.org/)

However, for this blog post, we are obviously focusing on using Cloudflare :smile:

## Using Cloudflare as a DDNS provider

We will be using joshuaavalon's [docker-cloudflare](https://github.com/joshuaavalon/docker-cloudflare) container to use Cloudflare as a DDNS provider.

There are many Cloudflare DDNS containers out there. The reason I use this one is because:

- It's [multi-architecture](https://github.com/joshuaavalon/docker-cloudflare/pkgs/container/cloudflare-ddns)

  - ARM FTW! 🥧

- It has a minimal [configuration file](https://github.com/joshuaavalon/docker-cloudflare#file) supporting YAML, JSON or a Javascript file. Configuration via environment variables is also supported but is considered *"legacy"*.

- It provides more advanced configuration options such as using an IPV(4|6) lookup service of your choice, and Webhooks to notify when DNS record updates run, succeed or fail.

### Prerequisites

1. [A Cloudflare account and domain added to Cloudflare](https://support.cloudflare.com/hc/en-us/articles/201720164-Creating-a-Cloudflare-account-and-adding-a-website).

2. Docker installed on the device which can reach the Cloudflare API: `https://api.cloudflare.com/client/v4/`

3. A Cloudflare API token. Please follow the instructions [here](https://github.com/joshuaavalon/docker-cloudflare#api-token).

  > Make sure you generate an **API Token** and **not** a Global API Key!

### Using the docker-cloudflare container

The minimal configuration file required for the docker-cloudflare container is the following:

```yaml
# config.yaml
auth:
  # Provide your API token here!
  scopedToken: QPExdfoNLwndJPDbt4nK9-yF1z_srC8D0m6-Gv_h
domains:
  # Remember to change this to your domain!
  - name: ddns.yourdomain.com
    # The type of record that is created
    type: A
    # Determines the proxy status for the record
    proxied: true
    # If the record does not exist, create it
    create: true
    # zoneId could also be yourdomain.com if the Cloudflare API token is granted #zone:read permissions
    zoneId: JBFRZWzhTKtRFWgu3X7f4YLX
```

The container will update your DNS record(s) with your public IP address every 5 minutes.

Once you have your configuration file, use the following command to start the docker-cloudflare container:

`docker run --name cloudflare-ddns -d -v ./config.yaml:/app/config.yaml joshava/cloudflare-ddns`

Once started, you should see something similar to the following when running `docker logs cloudflare-ddns`:

```
[cont-init.d] executing container initialization scripts...
[cont-init.d] 10-adduser: executing...
usermod: no changes

Initializing container

User uid: 1001
User gid: 1001

[cont-init.d] 10-adduser: exited 0.
[cont-init.d] 11-cron: executing...
Setting crontab to */5 * * * *
[cont-init.d] 11-cron: exited 0.
[cont-init.d] done.
[services.d] starting services
[services.d] done.
2021-09-16T18:02:57.903Z [info] Cloudflare DDNS start
2021-09-16T18:03:00.026Z [info] Skipped updating.
2021-09-16T18:03:00.027Z [info] Updated ddns.yourdomain.com with <your public IP>
2021-09-16T18:03:00.028Z [info] Cloudflare DDNS end
```

Congratulations, you're now using Cloudflare as a DDNS provider! :smile: :star:

### Bonus - Using a Discord Webhook for updates

A few months ago I looked into using Discord webhooks to receive updates for when my DNS record(s) were updated by the container.

I submitted a [PR](https://github.com/joshuaavalon/docker-cloudflare/pull/51) because I wanted to send a specific message along with the webhook payload. After discussion with the author, a webhook `formatter` was added which I tested using a Discord webhook.

Firstly, you need to [create a Discord webhook](https://support.discord.com/hc/en-us/articles/228383668-Intro-to-Webhooks) for a channel on your server. Once created, provide the webhook URL in the config below.

To use the `formatter`, you need to use a Javascript configuration file similar to the following:

```javascript
// config.js
const formatter = (status, data) => {
  if (status === "run") {
    return { content: "Updating DNS record." };
  } else {
    return { content: JSON.stringify(data) };
  }
};

const config = {
  auth: {
    scopedToken: "QPExdfoNLwndJPDbt4nK9-yF1z_srC8D0m6-Gv_h"
  },
  domains: [
    {
      // Remember to change to your domain!
      name: "ddns.yourdomain.com",
      type: "A",
      proxied: true,
      create: true,
      // Remember to change the zone ID!
      zoneId: "JBFRZWzhTKtRFWgu3X7f4YLX",
      webhook: {
        // Make sure you edit the webhook URL below to your own!
        run: "https://discord.com/api/webhooks/111111233445566678/py-D6zAc4IolXBoA7gslLAJc0WKO3KPU1eOxSNzX6qlkCBsqIP8EGILj-ALraivIbs6n",
        success: "https://discord.com/api/webhooks/111111233445566678/py-D6zAc4IolXBoA7gslLAJc0WKO3KPU1eOxSNzX6qlkCBsqIP8EGILj-ALraivIbs6n",
        failure: "https://discord.com/api/webhooks/111111233445566678/py-D6zAc4IolXBoA7gslLAJc0WKO3KPU1eOxSNzX6qlkCBsqIP8EGILj-ALraivIbs6n",
        formatter
      }
    }
  ]
};

module.exports = config;
```

As the configuration file has changed to a Javascript file, the docker run command used before is slightly different: `docker run --name cloudflare-ddns -d -v ./config.js:/app/config.js joshava/cloudflare-ddns`

Once running, you should see messages appearing in the Discord channel when a DNS record update occurs!

I hope you found this blog post useful! Until next time! :smile:

## References

[^1]: https://www.cloudflare.com/learning/dns/glossary/dynamic-dns/
