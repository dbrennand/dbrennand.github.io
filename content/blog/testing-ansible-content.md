---
title: "Testing Ansible Content with Molecule"
date: 2023-07-30T16:14:50+01:00
draft: false
tags: ["Ansible", "Testing", "Molecule"]
showToc: true
---

Time for another blog post! :rocket:

In this blog post, I will be discussing how to test Ansible content with [Molecule](https://github.com/ansible-community/molecule) 🧪

## What is Molecule?

> Molecule aids in the development and testing of Ansible content: collections, playbooks and roles.
>
> <cite>github.com/ansible-community/molecule[^1]</cite>

## Why do we need to test Ansible content?

Testing is an integral part of the software development lifecycle. It helps identify and prevent bugs from reaching production and in some cases, helps identify performance issues. When creating Ansible content we need to ensure that it works as expected and that we are not introducing any undesired behaviour. This is where Molecule comes in.

## Molecule Terminology 📚

Molecule has several terms that are used throughout the documentation. Let's go over them now.

## Instances & Drivers 🚗

Molecule instances are what your Ansible content is executed against. Instances are created using a driver. Molecule has several drivers for handling the creation and destruction of instances. The drivers are currently located in [ansible-community/molecule-plugins](https://github.com/ansible-community/molecule-plugins) repository.

For example, the default [Docker driver](https://github.com/ansible-community/molecule-plugins/blob/main/src/molecule_plugins/docker/driver.py) can be used to create a container instance and the [Vagrant driver](https://github.com/ansible-community/molecule-plugins/blob/main/src/molecule_plugins/vagrant/driver.py) can create a virtual machine instance.

## Scenarios 📖

Molecule scenarios can be thought of as a test suite. Each scenario contains its own instances and configuration. For example, a scenario could be used to test an Ansible role against different distributions or test a specific configuration of a role.

There should always be a `default` scenario which is used to test Ansible content with its default configuration.

## Molecule Directory Structure 📁

A basic Molecule directory structure is:

```
molecule
└── default
    ├── prepare.yml
    ├── converge.yml
    ├── verify.yml
    └── molecule.yml
```

Within the `molecule` directory is the `default` scenario directory which contains the `prepare.yml`, `converge.yml` and `verify.yml` playbooks, as well as the `molecule.yml` configuration file.

### Prepare Playbook ▶️

The `prepare.yml` playbook defines the preparation tasks to run before the `converge.yml` playbook. For example, this could be used configure the instance.

### Converge Playbook ▶️

The `converge.yml` playbook defines the Ansible content to be tested. This can be a playbook, role or collection.

### Verify Playbook ▶️

The `verify.yml` playbook is executed after the `converge.yml` playbook. This playbook contains verification tasks to check the Ansible content has been applied correctly.

### Configuration File `molecule.yml` 📝

The `molecule.yml` file contains configuration for each Molecule component[^2]:

<!-- Create a Markdown table with 2 columns: Molecule Component and Description -->

| Component                                                                                          | Description                                                                                                                        |
| -------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| [Dependency Manager](https://ansible.readthedocs.io/projects/molecule/configuration/#dependency)   | Molecule uses [Ansible Galaxy](https://galaxy.ansible.com/) as the default manager for resolving role and collection dependencies. |
| [Driver](https://ansible.readthedocs.io/projects/molecule/configuration/#driver)                   | [Instances & Drivers](#instances--drivers-).                                                                                       |
| [Platforms (Instances)](https://ansible.readthedocs.io/projects/molecule/configuration/#platforms) | Defines instances to be created by the driver for the scenario.                                                                    |
| [Provisioner](https://ansible.readthedocs.io/projects/molecule/configuration/#provisioner)         | Molecule uses Ansible as the provisioner. The provisioner manages the life cycle of instances by communicating with the driver.    |
| [Scenario](https://ansible.readthedocs.io/projects/molecule/configuration/#scenario)               | Molecule's default scenario configuration can be overridden for full control over each sequence.                                   |
| [Verifier](https://ansible.readthedocs.io/projects/molecule/configuration/#verifier)               | Molecule uses Ansible as the verifier. The verifier uses the `verify.yml` playbook to check the state of the instance.             |

The example below shows a basic configuration using the Docker driver to create a Debian container instance.

```yaml
---
dependency:
  name: galaxy
driver:
  name: docker
platforms:
  - name: instance
    image: geerlingguy/docker-debian10-ansible:latest
    pre_build_image: true
provisioner:
  name: ansible
verifier:
  name: ansible
```

### Molecule Commands 📜

Molecule has several commands for performing different actions. The most common commands are:

<!-- Create a Markdown table with two columns: Molecule Command and Description -->

| Molecule Command | Description                                                                                                                                           |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| `create`         | Creates the instance defined in the `molecule.yml` file.                                                                                              |
| `destroy`        | Destroys the instance defined in the `molecule.yml` file.                                                                                             |
| `login`          | Spawns a shell inside the instance. Useful for troubleshooting.                                                                                       |
| `converge`       | Performs the converge sequence which includes creating the instance and executing the `prepare.yml` and `converge.yml` playbooks.                     |
| `test`           | Performs a full test scenario. This includes the converge and idempotency sequences, executing the `verify.yml` playbook and destroying the instance. |

Now that we have covered the basics, let's test an Ansible playbook with Molecule! 🧪

## Demo 📺

I've created a [demo repository](https://github.com/dbrennand/molecule-demo) which contains a simple playbook to install `nginx` on a Debian container instance. You'll need to have Docker and Python installed to use the repository.

Begin by cloning the repository:

```bash
git clone https://github.com/dbrennand/molecule-demo.git && cd molecule-demo
```

Inspect the files and notice the `default` molecule scenario with the playbooks and `molecule.yml` configuration file. The `converge.yml` playbook calls the main `playbook.yml` two directories above.

Next, install Molecule and the Docker driver. I recommend using a virtual environment:

```bash
mkdir -pv ~/.virtualenvs
python3 -m venv ~/.virtualenvs/molecule-demo
source ~/.virtualenvs/molecule-demo/bin/activate
pip install -r requirements.txt
```

Now we can run the `molecule test` command to perform a full test scenario:

```bash
molecule test
```

The output should look similar to the following:

```text
INFO     Running default > create

PLAY [Create] ******************************************************************

TASK [Create molecule instance(s)] *********************************************
changed: [localhost] => (item=instance)

...

INFO     Running default > prepare

PLAY [Prepare] *****************************************************************

TASK [Gathering Facts] *********************************************************
ok: [instance]

TASK [Update apt cache] ********************************************************
changed: [instance]

...

INFO     Running default > converge

PLAY [Converge] ****************************************************************

TASK [Gathering Facts] *********************************************************
ok: [instance]

PLAY [Playbook | Install nginx] ************************************************

TASK [Gathering Facts] *********************************************************
ok: [instance]

TASK [Install nginx] ***********************************************************
changed: [instance]

PLAY RECAP *********************************************************************
instance                   : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

INFO     Running default > idempotence

PLAY [Converge] ****************************************************************

TASK [Gathering Facts] *********************************************************
ok: [instance]

PLAY [Playbook | Install nginx] ************************************************

TASK [Gathering Facts] *********************************************************
ok: [instance]

TASK [Install nginx] ***********************************************************
ok: [instance]

PLAY RECAP *********************************************************************
instance                   : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

INFO     Idempotence completed successfully.
INFO     Running default > side_effect
WARNING  Skipping, side effect playbook not configured.
INFO     Running default > verify
INFO     Running Ansible Verifier

PLAY [Verify] ******************************************************************

TASK [Gathering Facts] *********************************************************
ok: [instance]

TASK [Gather package list] *****************************************************
ok: [instance]

TASK [Verify nginx is installed] ***********************************************
ok: [instance] => {
    "changed": false,
    "msg": "All assertions passed"
}

PLAY RECAP *********************************************************************
instance                   : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

INFO     Verifier completed successfully.
INFO     Running default > cleanup
WARNING  Skipping, cleanup playbook not configured.
INFO     Running default > destroy

PLAY [Destroy] *****************************************************************

TASK [Wait for instance(s) deletion to complete] *******************************
FAILED - RETRYING: [localhost]: Wait for instance(s) deletion to complete (300 retries left).
changed: [localhost] => (item=instance)

PLAY RECAP *********************************************************************
localhost                  : ok=3    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
```

Tada! You've just tested an Ansible playbook with Molecule! ✨🎉

We now have peace of mind that our playbook works as expected! :slight_smile:

## Conclusion 📝

In this blog post, we covered the basics of using Molecule and how to test Ansible content. I recommend checking out the [Molecule documentation](https://ansible.readthedocs.io/projects/molecule/) for more information.

I hope you found this post useful and if you have any questions or feedback, feel free to reach out to me on [Twitter](https://twitter.com/dbrenuk) or via [email](mailto:contact@danielbrennand.com).

Until next time, happy testing! 🧪 :wave:

## References

[^1]: https://github.com/ansible-community/molecule

[^2]: https://ansible.readthedocs.io/projects/molecule/getting-started/#the-scenario-layout
