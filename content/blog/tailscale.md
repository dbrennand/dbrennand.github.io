---
title: "Remote access with Tailscale"
date: 2024-09-29T13:57:49+01:00
draft: false
tags: ["Tailscale", "VPN", "Remote Access"]
showToc: true
---

Right now, I'm writing this blog post sitting in a coffee shop, securely routing all my traffic through a device on my home network and have access to all my Homelab services and devices as if I was at home too. How? Tailscale! :heart:

## What is Tailscale?

> Tailscale is a secure and private, identity-based, infrastructure agnostic network with a flexible topology, resilient networking, and a streamlined setup.
>
> — <cite>Tailscale Inc[^1]</cite>

[^1]: [Why Tailscale?](https://tailscale.com/why-tailscale)

Tailscale uses the [Wireguard](https://www.wireguard.com/) protocol under the hood, which is modern, fast and secure. Tailscale establishes lightweight encrypted tunnels between devices forming a peer-to-peer mesh network (known as a Tailnet).

You can sign up and login to Tailscale using several providers such as Google, GitHub, Microsoft and more. Once signed up, you can download Tailscale for a range of platforms including: Windows, MacOS, Linux, Android and iOS. It can run on desktops, laptops, servers, VMs, containers, and even your router!

If you want to learn more in depth about how Tailscale works, check out their [documentation](https://tailscale.com/blog/how-tailscale-works).

## Why Tailscale?

So why is Tailscale so great? Why not deploy a traditional VPN like OpenVPN or WireGuard?

Well, I've deployed and used VPN servers like this in the past, one example which comes to mind is [PiVPN](https://www.pivpn.io/). Personally, I don't have the time to maintain a dedicated VPN server, manage configuration, users, keys etc. I just want something that's low maintenance, easy to deploy and use. Tailscale ticks all these boxes ✅

Tailscale is a zero-config VPN, meaning you don't need to worry about IP addresses, ports, or keys. It just works! I don't need to open any ports on my firewall, and Tailscale provides fine grained access controls via [ACLs](https://tailscale.com/kb/1018/acls), so only the devices that I want to talk to one another can do so. One other feature that I really like is [MagicDNS](https://tailscale.com/kb/1081/magicdns), which allows you to resolve hostnames of devices on your Tailnet, so you don't need to remember IP addresses! 🎉

They have a very generous personal (free) plan allowing up to 3 users and 100 devices. This is more than enough for me and most others.

## The Coordination Server and Risk Mitigation

One thing to note is that Tailscale relies on a central ['coordination server'](https://tailscale.com/blog/how-tailscale-works#the-control-plane-key-exchange-and-coordination) accessible at https://login.tailscale.com. This is how you sign in and manage your Tailnet. The coordination server is also responsible for exchanging Wireguard public keys, assigning IP addresses for devices, allowing machine sharing features and setting policies (ACLs) on your Tailnet. However, it's important to stress that no traffic from your devices is routed through the coordination server.

Unlike the Tailscale client which is open source on [GitHub](https://github.com/tailscale/tailscale), the coordination server is not open source. Now for some people this might be a deal breaker, but there are some alternatives, such as [Headscale](https://github.com/juanfont/headscale) which is an open source implementation of the coordination server.

Moreover, part of my threat model was to minimise the impact of the Tailscale coordination server being compromised and malicious devices being added to my Tailnet. Luckily, Tailscale has a feature called [Tailnet lock](https://tailscale.com/kb/1226/tailnet-lock) where new devices need to be signed by trusted nodes before they can join the Tailnet. If you're using Tailscale I highly recommend enabling this feature.

## Deploying Tailscale

Deploying Tailscale on devices is easy. With Tailscale lock enabled there is an extra step but it's not difficult. Here's how I deployed Tailscale on my devices.

I use the [artis3n Tailscale Ansible role](https://github.com/artis3n/ansible-role-tailscale). Below is an example playbook:

```yaml
---
- name: Tailscale | Install & Update
  hosts: tailscale
  roles:
    - role: artis3n.tailscale
```

Role variables are managed in the inventory `group_vars` and `host_vars` because for some devices I override the defaults such as when deploying an [exit node](https://tailscale.com/kb/1103/exit-nodes) or [subnet router](https://tailscale.com/kb/1019/subnets). More on this later!

You can find the example inventory in my [HomeOps repository](https://github.com/dbrennand/home-ops/tree/dev/ansible/inventory). Once I've deployed Tailscale on the device, I login to the Tailscale admin console to get the `tailscale lock sign` command to sign the device using a trusted signing node. Alternatively, if you don't use Ansible you can install Tailscale using the following command on Linux:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

You'll then be directed to a URL to authenticate the device to your Tailnet. Easy as :pie:!

## Subnet Routers

> A subnet router is a device in your tailnet that you use as a gateway to advertise routes to other devices.
>
> — <cite>Tailscale Inc[^2]</cite>

[^2]: [Subnet Routers](https://tailscale.com/kb/1019/subnets)

In my setup I have [two subnet routers](https://github.com/dbrennand/home-ops/blob/dev/ansible/inventory/inventory.yml#L34) for resiliency. Both are running on Pi-hole devices. I advertise [specific addresses](https://github.com/dbrennand/home-ops/blob/dev/ansible/inventory/group_vars/subnet_router.yml#L7) on my Tailnet instead of the entire subnet. Furthermore, these Pi-hole devices are configured as [DNS servers for my Tailnet](https://homeops.danielbrennand.com/ansible/pihole/#post-configuration-setting-the-pi-hole-as-the-tailnet-global-nameserver) so I can resolve internal DNS names from Tailscale and get [Pi-hole ad-blocking](https://tailscale.com/kb/1114/pi-hole) too!

## Exit Nodes

> The exit node feature lets you route all traffic through a specific device on your Tailscale network.
>˚
> — <cite>Tailscale Inc[^3]</cite>

[^3]: [Exit Nodes](https://tailscale.com/kb/1103/exit-nodes)

An exit node is the equivalent of using the default routes `0.0.0.0/0, ::/0` on a traditional VPN configuration. All public internet traffic is routed through the exit node, this is perfect for scenarios where you're on an untrusted network (like the coffee shop Wi-Fi I'm using right now!) and want to route all traffic through a trusted device on your Tailnet. Furthermore, exit nodes are great for bypassing geo-restrictions, for example, if you're travelling and want to watch content that's only available in your home country, or avoid tripping security systems when accessing online banking whilst abroad.

In my setup I have two exit nodes running as unprivileged Proxmox LXC containers. There is a tiny piece of [manual configuration](https://tailscale.com/kb/1130/lxc-unprivileged) needed for this.

## Conclusion

IMHO Tailscale is brilliant and it's pretty mind-blowing how easy it is to set up and use. I'm honestly kind of shocked that I can use all these features for free too. Their free-tier is very generous and even if it did cost money in the future, I'd be happy to pay for it because it's a great service.

That's all for this one folks! Until next time! :wave:
