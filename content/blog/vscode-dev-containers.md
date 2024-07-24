---
title: "Getting started with Visual Studio Code Dev Containers"
date: 2024-07-24T10:46:45+01:00
draft: false
tags: ["Dev Containers", "Visual Studio Code"]
showToc: true
---

Hello internet! :wave: In this blog post I will cover how to get started with [Visual Studio Code Dev Containers](https://code.visualstudio.com/docs/devcontainers/containers).

## What are Dev Containers and why use them?

Dev Containers provide a containerised development environment within Visual Studio Code. With Dev Containers, you and your team can have a consistent development environment, regardless of the host OS. Moreover, it helps ease the onboarding process, getting new team members up and running quickly, and helps avoid common issues such as "it works on my machine" or "oh, you've got a different version of x".

So without further ado, let's get started! :rocket:

## Getting Started

To use Dev Containers, you will need the following:

- [Visual Studio Code](https://code.visualstudio.com/) *obviously* :wink:
- [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)
- A container runtime such as Docker, Podman or OrbStack (uses Docker engine under the hood - MacOS only).

My container runtime of choice is [OrbStack](https://orbstack.dev/). I've been using it for a while now and it's been great! :tada: It's fast and very lightweight. Podman is another great alternative, especially if you're looking for a Docker alternative. [Podman Desktop](https://podman-desktop.io/) was released not long ago too, so definitely check it out if GUIs are more your thing :slightly_smiling_face:

## Creating a Dev Container

Dev Containers are configured using a [`devcontainer.json`](https://code.visualstudio.com/docs/devcontainers/containers#_create-a-devcontainerjson-file) file which resides in a `.devcontainer` directory in your repository. This file defines the entire configuration for the containerised environment including the base image, extensions, settings and more.

For this blog post, I will create a Dev Container for developing Ansible content. We will need Ansible core, Python and the [Ansible extension](https://marketplace.visualstudio.com/items?itemName=redhat.ansible) for Visual Studio Code.

Let's start by creating the needed directories for the Ansible project:

```bash
mkdir -pv ansible-dev-containers-demo/.devcontainer
```

Next, we will create the `devcontainer.json` file:

```bash
touch ansible-dev-containers-demo/.devcontainer/devcontainer.json
```

Now, let's add the following configuration to the `devcontainer.json` file:

```json
{
    "name": "Ansible Dev Containers Demo",
    "build": {
        "dockerfile": "Dockerfile"
    },
    "customizations": {
        "vscode": {
            "settings": {
                "redhat.telemetry.enabled": false
            },
            "extensions": [
                "redhat.ansible"
            ]
        }
    }
}
```

Let's break down the configuration. We will be building the base image using a simple `Dockerfile`. The Ansible extension will be installed in the Dev Container and Red Hat telemetry in the extension will be disabled.

Next, we'll create the `Dockerfile` with the following content:

```bash
touch ansible-dev-containers-demo/.devcontainer/Dockerfile
```

```Dockerfile
FROM python:3.12-slim
# Install ansible-core
RUN pip install ansible-core
```

Now, let's open the project in Visual Studio Code and we will see a toast notification that a Dev Container configuration file has been detected:

![Dev Container toast notification](../images/dev-container-toast-notification.png)

Click `Reopen in Container` to start building the Dev Container. Once built, open a terminal in Visual Studio Code, and you'll see that we are running inside the Dev Container with Ansible and the extension installed:

```bash
root@b6c0de05320f:/workspaces/ansible-dev-containers-demo# ansible --version
ansible [core 2.17.2]
  config file = None
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.12/site-packages/ansible
  ansible collection location = /root/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/local/bin/ansible
  python version = 3.12.4 (main, Jul  2 2024, 20:57:30) [GCC 12.2.0] (/usr/local/bin/python)
  jinja version = 3.1.4
  libyaml = True
```

![Dev Container Ansible extension](../images/dev-container-ansible-extension.png)

You now have a containerised environment for Ansible development! :tada:

## Rebuilding the Dev Container

If you need to rebuild the Dev Container, you can do so by opening the command palette (`Ctrl|Command+Shift+P`) and running the `Dev Containers: Rebuild and Reopen in Container` command.

## Conclusion

Dev Containers are a great way for you and your team to have a consistent development environment. We only scratched the surface of what's possible with Dev Containers in this post. In the future, I'll cover running commands or scripts pre and post startup, 1Password SSH integration, environment variables, port forwarding and more! :slightly_smiling_face:

If you'd like to learn more about Dev Containers, check out the [official documentation](https://code.visualstudio.com/docs/devcontainers/containers).

Until next time! :wave:
