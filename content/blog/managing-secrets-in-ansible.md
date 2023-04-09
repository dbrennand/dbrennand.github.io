---
title: "Managing Secrets in Ansible"
date: 2023-04-03T17:31:31+01:00
draft: false
tags: ["Ansible", "Managing Secrets", "Credentials"]
showToc: true
---

Hi there! 👋

In my previous [blog post](https://danielbrennand.com/blog/getting-started-ansible/) I covered how to get started with Ansible. In this blog post I'll be continuing to write about Ansible and covering how to manage secrets in Ansible playbooks.

Ready... set... go! 🚀

# Managing Secrets in Ansible 🔑

As with most automation, we need to use credentials to authenticate to our servers and other applications. Examples of secrets include usernames and passwords, API keys, SSH keys, etc. When using these types of secrets in playbooks, we need to store them securely but also still allow Ansible to access them when needed.

# Introducing Ansible Vault 🔐

[Ansible Vault](https://docs.ansible.com/ansible/2.8/user_guide/vault.html#) is a command line tool that is part of the `ansible-core` package and allows us to encrypt secrets in our playbooks, inventory and vars files. It can encrypt an entire file or a specific variable within a file. Furthermore, Ansible Vault uses the AES256 encryption algorithm to encrypt files and variables using a password provided by the user.

# Using Ansible Vault

Let's take a look at how we can use Ansible Vault to encrypt a file.

## Encrypting a File 🔒

To encrypt a file use the `ansible-vault encrypt` command, providing the path for the file to be encrypted. When prompted enter a password to encrypt the file:

```bash
ansible-vault encrypt vars/main.yml
New Vault password:
Confirm New Vault password:
Encryption successful
```

Now we can see the file is encrypted:

```bash
cat vars/main.yml
$ANSIBLE_VAULT;1.1;AES256
64383033313638636464333937393463663432616532353763646137356664376533326261373265
3466643962633863613061646235333137623266666238650a356163653039616634653632383933
30316438663233316365383637643135633736303435666236356130356565363263353262306564
3931613035373031360a396431626666343564626264346636633266343766316535346266346537
62393334393030386365376237393565343637663766336261313862363936343161
```

## Editing or Viewing an Encrypted File

To edit or view the contents of an encrypted file, use the `ansible-vault edit` or `ansible-vault view` command respectively:

```bash
ansible-vault edit vars/main.yml
Vault password:
ansible-vault view vars/main.yml
Vault password:
```

## Decrypting a File 🔓

To decrypt the file, use the `ansible-vault decrypt` command and as before, provide the path to the file and the password:

```bash
ansible-vault decrypt vars/main.yml
Vault password:
```

Next, let's see how we can use Ansible Vault to encrypt a variable.

## Encrypting a Variable

To encrypt a variable, use the `ansible-vault encrypt_string` command providing the name of the variable and the value to encrypt. When prompted, enter a password to encrypt the variable:

```bash
ansible-vault encrypt_string -n my_secret_var "<Secret to encrypt here>"
New Vault password:
Confirm New Vault password:
Encryption successful
my_secret_var: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          63396563373936313066626338313132343139613636353538633938303135666562326139373534
          3962633831613839623663353938393130343036333936340a383830366436323261646137383036
          36373738346433366333666632653966323630323237666466326538336535323330663030303762
          3436336162363433310a396134663062323431363263393930356237366563306238363463383066
          6633
```

You can now copy and paste the output into your playbook `vars` section, vars file or inventory.

Great! So we now know how to encrypt a file and a variable, but how about a real world example! 🌎

# Real World Example

I sometimes use [Hetzner Cloud](https://www.hetzner.com/) to spin up virtual servers for testing. In this short playbook which I use to create the server, there is a variable called `hetzner_api_key` which contains my API key. I don't want to store this API key in plain text in my playbook! :scream: 🙅‍♂️

```yaml
# playbook.yml
---
- name: Provision Hetzner VPS
  hosts: all
  gather_facts: false
  vars:
    hetzner_api_key: "my-secret-api-key"
  tasks:
    - name: Create a basic server
      hetzner.hcloud.hcloud_server:
        api_token: "{{ hetzner_api_key }}"
        location: fsn1
        name: my-server
        server_type: cx11
        image: ubuntu-18.04
        state: present
```

Using Ansible Vault we can encrypt this variable in the playbook:

```bash
ansible-vault encrypt_string -n hetzner_api_key "my-secret-api-key"
New Vault password:
Confirm New Vault password:
Encryption successful
hetzner_api_key: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          30646134626337346138323439636637366233633261373339666233653735616364623533616539
          3133376230313265326631343439383238653539373034650a633632613037653337626237653730
          38636136613162386530363530393537323132303538653265633635326236336561633234306562
          3061613964343037320a323261363731613130343932393439626133616163346663623933313562
          35313765626338653133336436393332636635656361363730363033626335643261
```

With the variable safely encrypted, place the variable into the playbook:

```yaml
# playbook.yml
---
- name: Provision Hetzner VPS
  hosts: all
  gather_facts: false
  vars:
    hetzner_api_key: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          30646134626337346138323439636637366233633261373339666233653735616364623533616539
          3133376230313265326631343439383238653539373034650a633632613037653337626237653730
          38636136613162386530363530393537323132303538653265633635326236336561633234306562
          3061613964343037320a323261363731613130343932393439626133616163346663623933313562
          35313765626338653133336436393332636635656361363730363033626335643261
  tasks:
    - name: Create a basic server
      hetzner.hcloud.hcloud_server:
        api_token: "{{ hetzner_api_key }}"
        location: fsn1
        name: my-server
        server_type: cx11
        image: ubuntu-18.04
        state: present
```

Finally, when running the playbook use the `--ask-vault-pass` option and enter the password used to encrypt the variable:

```bash
ansible-playbook -c local -i localhost, --ask-vault-pass playbook.yml
```

All done! :tada:

# Prompting for the Vault Password Non-Interactively

The above example was interactively prompting for the vault password. So how do we do this non-interactively? Say we want to run the playbook in a CI/CD pipeline. We can't prompt for the password in this type of scenario :thinking:

To solve this, we can use a [vault password file](https://docs.ansible.com/ansible/2.8/user_guide/vault.html#providing-vault-passwords).

First, add the password to a file called `.vault_password` and lock down the permissions::

```bash
# Note the space at the beginning of the command!
# This is needed to prevent the password from being stored in the shell history.
 echo "my-encryption-password" > .vault_password
chmod 600 .vault_password
```

Now, we run the playbook with the `--vault-password-file` option instead:

```bash
ansible-playbook -c local -i localhost, --vault-password-file .vault_password playbook.yml
```

# Conclusion

In this post we have covered how to use Ansible Vault for safely storing secrets for use in our playbooks. We've covered encrypting files and variables and how to use a vault password file to prompt for the vault password non-interactively.

I hope you found this post useful and if you have any questions or feedback, feel free to reach out to me on [Twitter](https://twitter.com/dbrenuk) or via [email](mailto:contact@danielbrennand.com).

Until next time, happy automating! :rocket: :wave:
