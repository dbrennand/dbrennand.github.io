---
title: "Becoming an Ansible Collection Maintainer: community.beszel"
date: 2025-10-02T20:34:15+01:00
draft: false
tags: ["Ansible", "Collection", "Community", "Beszel"]
showToc: true
---

In this blog post I wanted to talk about how I became a maintainer for the [`community.beszel`](https://github.com/ansible-collections/community.beszel) Ansible Collection and my learnings.

# What is Beszel?

[Beszel](https://beszel.dev/) is a simple, lightweight server monitoring software which can be self-hosted on your own infrastructure. I first heard about Beszel whilst reading the [`/r/selfhosted`](https://www.reddit.com/r/selfhosted/comments/1eb4bi5/i_just_released_beszel_a_server_monitoring_hub/) subreddit about a year ago, and a couple of months ago I had time to deploy it in my [Home-Ops](https://github.com/dbrennand/home-ops/tree/main/docker/beszel) project.

Beszel consists of two components: the hub and the agent. The hub provides a web interface for visualising the data sent from agents which run on your servers. My Beszel Hub is currently receiving data from 10 systems in my Homelab:

![Beszel Hub](../images/beszel-hub.png)

If you want to read more about my setup I've written a little about it [here](https://homeops.dbren.uk/infrastructure/beszel/).

I thought to myself, I want an automated way of deploying and managing the lifecycle of the Beszel agent on my Homelab devices, so I looked to Ansible to help me with this.

# Humble Beginnings - `dbrennand.beszel` Ansible Role

I couldn't find anything developed on GitHub for automated Beszel Agent deployment via Ansible, so I created an Ansible Role [`dbrennand.beszel`](https://github.com/dbrennand/ansible-role-beszel) on GitHub and published it to [Ansible Galaxy](https://galaxy.ansible.com/ui/standalone/roles/dbrennand/beszel/). This standalone role worked really well and I knew the role could be useful for others, and boy was I right! :grin:

It wasn't long before [pull requests](https://github.com/dbrennand/ansible-role-beszel/pulls?q=is%3Apr+is%3Aclosed) were coming in to implement features outside the original agent deployment scope of the role. It was clear to me that we were going into [Ansible Collection](https://docs.ansible.com/ansible/latest/collections_guide/index.html) territory. If you're new to Ansible and want to learn more about collections then check out my previous [blog post](https://dbren.uk/blog/getting-started-ansible/#collections-) where I discuss them.

# Birth of `community.beszel`

Initially I reached out to the Beszel [maintainer](https://github.com/henrygd/beszel/discussions/969) to propose the idea of an Ansible Collection, but after some more research I found that there is a process for requesting Ansible community collections and this seemed like a great fit. The Beszel [maintainer](https://github.com/henrygd/beszel/discussions/969#discussioncomment-13839515) thought so too, so I requested the new collection be created via the [Ansible forum](https://forum.ansible.com/t/community-beszel-ansible-collection/43999).

The initial `0.1.0` release of [`community.beszel`](https://github.com/ansible-collections/community.beszel) implemented the same Ansible Role as [`dbrennand.beszel`](https://github.com/dbrennand/ansible-role-beszel) but fast forward to today, we're on version `0.3.0` and have Ansible Roles for deploying Beszel Hub on baremetal and Ansible Modules for managing Beszel systems using the Beszel Hub REST API. There is a lot more functionality we can add to [`community.beszel`](https://github.com/ansible-collections/community.beszel) and the collection has started to receive contributions from the community which is awesome! :smiley:

# Learnings from becoming an Ansible Collection Maintainer

Becoming a maintainer for [`community.beszel`](https://github.com/ansible-collections/community.beszel) has taught me a lot about creating and testing Ansible Modules. I learned how to use [`ansible-test`](https://docs.ansible.com/ansible/latest/dev_guide/testing_running_locally.html) to create and run [integration, sanity and unit tests](https://github.com/ansible-collections/community.beszel/tree/main/tests). I documented these steps in the collection's [CONTRUBUTING.md](https://github.com/ansible-collections/community.beszel/blob/main/CONTRIBUTING.md#running-integration-tests) file. Furthermore, I recently gave some [feedback](https://forum.ansible.com/t/your-feedback-wanted-the-ansible-collection-developer-and-maintainer-journey/44592/3?u=dbrennand) on the Ansible forum about my new maintainer journey, things that I found really helpful and opportunities for improvement in documentation and tooling.

# Conclusion

Overall I've found it very satisfying seeing how my little Ansible role has turned into something bigger, and others are getting benefit from it too. I've enjoyed getting more involved in the Ansible community and I've learned a lot about maintaining and testing content in an Ansible collection. I'm sure there is much more still to learn too!

If you've made it this far then thanks for reading and I hope you have a great day!
