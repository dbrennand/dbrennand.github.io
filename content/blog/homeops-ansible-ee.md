---
title: "Home-Ops | AWX and Ansible Execution Environments"
date: 2025-05-16T15:08:18+01:00
draft: false
tags: ["Home-Ops", "Ansible", "AWX", "Execution Environment"]
showToc: true
---

Hey there folks! 👋

Back again with another blog post! This time I wanted to discuss some cool changes I've made in my Homelab to explore AWX and Ansible Execution Environments.

## What is AWX?

[AWX](https://github.com/ansible/awx) provides a web interface and REST API for managing and running Ansible content. AWX is the upstream project which Red Hat's commerical offering Ansible Automation Platform's Automation Controller is based on. AWX uses Execution Environments to run Ansible content and is deployed on Kubernetes using the [awx-operator](https://github.com/ansible/awx-operator).

![AWX](../images/awx.png)

## What is an Execution Environment (EE)?

An EE is an [OCI compliant container image](https://github.com/opencontainers/image-spec/) which contains Python, [`ansible-core`](https://pypi.org/project/ansible-core/), [`ansible-runner`](https://pypi.org/project/ansible-runner/) and other dependencies (Ansible content such as roles, collections and any dependent Python modules). Using EEs helps create a consistent environment every time we run our Ansible content.

## Motivations

Before making these changes in my Homelab, I was aware of AWX and EEs but I hadn't used either of them much, or built my own EE before. I've always wanted to use AWX and recent changes in my job have also motivated my use of AWX too.

## Building Execution Environments

The Ansible development tooling ecosystem has a very handy tool for building EEs called [`ansible-builder`](https://github.com/ansible/ansible-builder). Ansible builder helps simplify the EE creation process by handling tasks such as including Python modules, Ansible roles and collections, and configuring a certain user and permissions; just to name a few.

Ansible builder uses a definition file named [`execution-environment.yml`](https://ansible.readthedocs.io/projects/builder/en/stable/definition/) to define the base container image, dependencies and additional build steps for the EE.

### Multi-Arch Execution Environment builds in GitHub Actions

In my [Home-Ops](https://github.com/dbrennand/home-ops) repository, I use `ansible-builder` in a [GitHub Action Workflow](https://github.com/dbrennand/home-ops/blob/dev/.github/workflows/build-ee.yml) to build my EE and upload it to the [GitHub Container Registry](https://github.com/dbrennand/home-ops/pkgs/container/home-ops). You can pull it now using the command:

```console
docker pull ghcr.io/dbrennand/home-ops:latest
```

The image is multi-arch supporting both `linux/amd64` and `linux/arm64` as I use my EE on devices with these CPU architectures. Getting the EE to build for multiple architectures inside a GitHub Action was very trial and error. I searched around online but couldn't find anyone else building a multi-arch EE with GitHub Actions and `ansible-builder`.

The first error I encountered was:

```
ERROR: Multi-platform build is not supported for the docker driver.
Switch to a different driver, or turn on the containerd image store, and try again.
Learn more at https://docs.docker.com/go/build-multi-platform/
```

I got this error even though I had initialised the workflow with the `docker/setup-qemu-action@v3` and `docker/setup-buildx-action@v3` actions for multi-arch builds, which I'd done before without any issues. It turns out that I needed to tell `ansible-builder` to use the name of the builder context created by the `docker/setup-buildx-action@v3` action. Furthermore, whilst testing there was another issue where the image built by `ansible-builder` wasn't loaded into the Docker daemon.

To resolve both these issues I used the `--extra-build-cli-args` argument with `ansible-builder` like so:

```console
ansible-builder ... --extra-build-cli-args "--load --builder ${{ steps.docker_buildx.outputs.name }} --platform linux/amd64,linux/arm64"
```

Once I did the above, the next error I encountered was:

```
ERROR: docker exporter does not currently support exporting manifest lists
```

From a recently active GitHub [issue](https://github.com/docker/buildx/issues/59#issuecomment-2612971318) this wasn't supported until very recently via an experimental containerd-snapshotter feature on the `docker/setup-docker-action@v4` action; so I enabled it:

```yaml
- name: Set up Docker
  uses: docker/setup-docker-action@v4
  with:
    daemon-config: |
    {
        "features": {
        "containerd-snapshotter": true
        }
    }
```

Finally, I had multi-arch builds of my EE in GitHub Actions working! Woo! :tada: :smile:

## Using the Execution Environment

I'm using the EE on my Macbook with [`ansible-navigator`](https://ansible.readthedocs.io/projects/navigator/) and AWX. I've written a little more about this [here](https://homeops.danielbrennand.com/ansible/execution-environment/).

As most of my playbooks use the [`community.general.onepassword`](https://docs.ansible.com/ansible/latest/collections/community/general/onepassword_lookup.html) lookup plugin to pull secrets from a 1Password vault, I've included the [op](https://developer.1password.com/docs/cli/get-started/) CLI in my EE.

To start the EE and authenticate the `op` CLI I run the following commands in my [Home-Ops](https://github.com/dbrennand/home-ops) repository:

```console
cd ansible
export ONEPASSWORD_SERVICE_ACCOUNT_TOKEN=op://Vault/ServiceAccount/token
op run -- ansible-navigator exec -- /bin/bash
```

The 1Password application on my Macbook then prompts for my fingerprint via Touch ID to authorise retrieval of the Service Account token. Pretty :cool: :sunglasses:

In AWX I've registered the EE for use in Job Templates using an [Ansible playbook](https://github.com/dbrennand/home-ops/blob/dev/ansible/playbooks/playbook-awx.yml#L11). To authenticate the `op` CLI in the EE running on AWX, I also create a [custom credential type](https://github.com/dbrennand/home-ops/blob/dev/ansible/playbooks/playbook-awx.yml#L46) which I then use in my Job Templates. I'll probably make another blog post expanding on this a little more soon! So stay tuned for that one! :slightly_smiling_face:

That's all for this one folks! Until next time! :wave:
