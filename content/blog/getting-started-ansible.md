---
title: "Getting started with Ansible"
date: 2023-03-05T18:59:57Z
draft: false
tags: ["Ansible", "Getting started"]
showToc: true
---

Hey there! :wave:

In this blog post I'll be writing about Ansible. I've been using it for a little while now and I'd like to share my experience with it and share some knowledge for others starting out! :slight_smile:

Let's begin! :rocket:

# What is Ansible?

Ansible is an open-source automation tool which can be used to automate the management and configuration of servers and other devices. Written in Python, Ansible is agentless and connects over SSH. Some common use-cases for Ansible include (but are not limited to):

- Orchestrating large scale application deployments.
- Prevent configuration drift and enforce consistent configurations.
- Provisioning and deprovisioning tasks.
- Audit server configuration and compliance.

What makes Ansible great is its simplicity and ease of use. By adopting a human friendly language such as [YAML](https://yaml.org/), it lowers the barrier to entry for new users and makes it easy to read and understand.

# Ansible Terminology 📚

Before we get started, lets go over some of the terminology used in Ansible.

## Control Node

The control node is the machine that Ansible is installed and run from. In most cases this will be your local machine however, in some cases, it may be another machine. For example, an organisation may use a Rundeck server as the control node or use [Ansible Automation Platform](https://www.redhat.com/en/technologies/management/ansible) which has the [Ansible Automation Controller](https://www.redhat.com/en/technologies/management/ansible/automation-controller) as the control node.

See the [Ansible control node requirements](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#control-node-requirements) documentation for further information.

## Inventory 📋

The inventory file contains a list of servers that Ansible will manage. Inventory can be in [INI](https://en.wikipedia.org/wiki/INI_file) or YAML format. Furthermore, it can be static or dynamically created via a script or inventory plugin (more on this later). For example, dynamic inventory could be created from a CMDB or cloud provider API.

### Static INI Inventory

```ini
# inventory.ini
[webservers] # Inventory group name
web1.example.com
web2.example.com

[databases]
db1.example.com
```

### Static YAML Inventory

```yaml
# inventory.yaml
---
webservers: # Inventory group name
  hosts:
    web1.example.com:
    web2.example.com:

databases:
  hosts:
    db1.example.com:
```

See the [Ansible inventory documentation](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html) for further information.

## Playbooks ▶️

Playbooks define the tasks (plays) that Ansible will run against the servers in the inventory. Playbooks are written in YAML and executed using the `ansible-playbook` command.

A simple playbook could look like this:

```yaml
---
# playbook.yaml
- name: Install Nginx on webservers
  hosts: webservers
  tasks:
    - name: Install Nginx
      ansible.builtin.package:
        name: nginx
        state: present
```

Which you can run like this:

```bash
ansible-playbook -i inventory.yaml playbook.yaml
```

See the [Ansible playbook documentation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html) for further information.

## Modules and Plugins 🔌

Modules are the building blocks for doing anything in Ansible. Each task in a playbook uses a module and there are many modules available for use right out of the box. Modules can perform a wide range of tasks such as installing packages, managing users, configuring services and more. The example playbook above uses the built-in [`package`](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/package_module.html) module to install Nginx on the `webservers`.

Plugins allow us to extend Ansible's core functionality. There are many types of plugins in Ansible. Some examples include:

- Inventory plugins to create dynamic inventory.
- Callback plugins which change the output format.
- Become plugins for changing the privilege escalation method.

The main difference between a module and a plugin is that modules are executed on the remote server whereas plugins are executed on the control node itself.

See the following documentation pages for more information on modules and plugins:

- https://docs.ansible.com/ansible/latest/module_plugin_guide/index.html
- https://docs.ansible.com/ansible/latest/module_plugin_guide/modules_intro.html
- https://docs.ansible.com/ansible/latest/plugins/module.html
- https://docs.ansible.com/ansible/latest/plugins/plugins.html

## Roles

Roles contain tasks, files, variables, templates and more for performing repeatable actions. The main benefit of roles is that they can be shared and reused easily across playbooks. You may have to regularly install and configure Nginx on servers; but the configuration differs slightly each time. This would be the perfect use-case for a role.

Roles can be installed using the `ansible-galaxy` command. For example, to install the [geerlingguy.nginx](https://galaxy.ansible.com/geerlingguy/nginx) role you would run the following command:

```bash
ansible-galaxy role install geerlingguy.nginx
```

Then, you can use the role in your playbook like this:

```yaml
# playbook.yaml
- name: Install and configure Nginx on webservers
  hosts: webservers
  roles:
    - geerlingguy.nginx
```

This would use the role defaults but nevertheless, it's far simpler and maintainable right?! No copying tasks from a playbook in one project to another! 🙅‍♂️ We need to stay sane after all 🤪

## Collections 📦

Collections are the newest distribution format for Ansible content. They can be used to package and distribute playbooks, roles, modules and plugins. Collections can also be installed using the `ansible-galaxy` command. For example, to install the [community.docker](https://galaxy.ansible.com/community/docker) collection you would run the following command:

```bash
ansible-galaxy collection install community.docker
```

# Installing Ansible ⚙️

My advice would be to install Ansible in a Python virtual environment. This is because more often than not, the version of Ansible provided by your package manager will be sorely outdated. Furthermore, this way the installation of Ansible won't interfere with your system's Python installation and avoiding the pains of dependency hell.

To install Ansible in a virtual environment you'll need Python 3 installed (the newer the better) and the [`venv`](https://docs.python.org/3/library/venv.html) module.

Begin by creating a directory to hold your virtual environment and initialise it:

```bash
mkdir -pv ~/.venv/ansible && python -m venv ~/.venv/ansible
```

Next, activate the virtual environment and install Ansible:

```bash
source ~/.venv/ansible/bin/activate
pip install ansible
```

Now you're ready to go: :tada:

```bash
ansible --version
ansible [core 2.14.3]
  config file = None
  configured module search path = ['/Users/user/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /Users/user/.venv/ansible/lib/python3.11/site-packages/ansible
  ansible collection location = /Users/user/.ansible/collections:/usr/share/ansible/collections
  executable location = /Users/user/.venv/ansible/bin/ansible
  python version = 3.11.1 (main, Feb  8 2023, 19:50:54) [Clang 13.1.6 (clang-1316.0.21.2.5)] (/Users/user/.venv/ansible/bin/python)
  jinja version = 3.1.2
  libyaml = True
```

# Ad-hoc Commands

Ad-hoc commands are a perfect way for running one off commands quickly and easily against one or more servers. For example, if we wanted to check the storage space of block devices on our servers we could run:

```bash
ansible webservers -i inventory.yml -m ansible.builtin.command -a "df -hT"
```

Yay! ✨ No more `ssh`ing into each server and running the command manually! 🤮 💪

# What am I using Ansible for?

Whilst configuring the devices in my Homelab, I quickly realised that I was deploying Caddy as a container and setting up Autorestic (my personal backup tool of choice) over and over again. I decided to create a [Caddy Docker](https://github.com/dbrennand/ansible-role-caddy-docker) and a [Autorestic](https://github.com/dbrennand/ansible-role-autorestic) role and use them in my playbooks. This way I can easily deploy them to any new device I add to my Homelab! :smile:

I've also been using Ansible in my Homelab to deploy my Raspberry Pi K3s cluster and configure my Intel NUC media server. These devices have several containerised services running on them such as Pi-hole, Jellyfin, the *arrs and more. It's still a work in progress and I plan on releasing the playbooks publicly once I've finalised them; so keep an eye out for that! :eyes:

# Conclusion

We've only just scratched the surface of Ansible in this blog post. There are many more features and capabilities that Ansible has to offer. I'm sure I'll be writing more blog posts about Ansible in the future. If you have any questions or suggestions on what I should write next, feel free to reach out to me on [Twitter](https://twitter.com/dbrenuk) or via [email](mailto:contact@danielbrennand.com).

Finally, if your thirst for knowledge isn't quenched, I highly recommend checking out the following resources:

- [Jeff Geerling's Ansible 101 YouTube series](https://www.youtube.com/watch?v=goclfp6a2IQ).
- [Jeff Geerling's Ansible for DevOps book](https://www.ansiblefordevops.com/).
- [Ansible documentation](https://docs.ansible.com/ansible/latest/index.html).

That's all folks! :wave:
