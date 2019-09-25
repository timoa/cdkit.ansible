# [CDKit](https://github.com/timoa/cdkit): Ansible

*[CDKit](https://github.com/timoa/cdkit) is a DevOps framework that helps to deploy mobile apps (iOS and Android) to the app stores (iTunes and Google Play).*

## Getting started

[Ansible](https://www.ansible.com/) is a configuration manager that let you run scripts (playbooks) on a small to large number of computers.

It's agentless and needs only a SSH connexion to the computers you want to update.

We will use Ansible to install and maintain the Android SDK, Java 8, Fastlane, SonarQube scanner, ImageMagick, etc. on the Go.CD agents

## Prerequisites

Before you start, you need to have followed the steps to install the Go.CD server and agent(s)

## Server

### Install Ansible

#### Linux

Please refer to this page to [install Ansible for your Linux distribution](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-the-control-machine)

#### macOS

If you installed previously Homebrew, let's use it to install Ansible with this command:

```bash
brew install ansible
```

### Configure Ansible

#### Checkout this repo

After you installed Ansible, you need to checkout this repository to the folder `/opt/ansible` (both Linux or macOS):

```bash
[sudo] mkdir /opt/ansible
git clone git@github.com:timoa/cdkit.ansible.git /opt/ansible
```

#### Configure your hosts

In case of the `hosts` file has not been packaged with your Ansible installation, you can use the template in this GIT repository or the folder `/opt/ansible/hosts`.

```bash
[sudo] cp /opt/ansible/hosts /etc/ansible/hosts
```

#### Create your Vault

To save securely your hosts credentials or any other sensitive information (API Keys, etc.), we will use an Ansible Vault.

To create one for the agents, just type this command:

```bash
ansible-vault create /opt/ansible/vault_agents
```

```bash
ansible_become_pass: {agents user password}
appc_username: {Appcelerator/Axway username (email address)}
appc_password: {Appcelerator/Axway password}
appc_org: {Appcelerator/Axway organisation ID}
```

You can also create one for the GoCD server to apply automatic updates (set the same Vault password):

```bash
ansible-vault create /opt/ansible/vault_gocd
```

```bash
ansible_become_pass: {gocd server user password}
```

Finally, we need to create a text file that will allow Ansible to programmatically open the Vault (ignored by Git)

```bash
vi /opt/ansible/.vaultpasswordfile
```

```bash
mysupersecurepassword
```

#### Edit the Ansible configuration file

Now, your need to open the `ansible.cfg` file to let Ansible knwo that we use a different location for our playbook, roles, etc.

```bash
[sudo] vi /etc/ansible/ansible.cfg
```

##### Roles

Change the roles path to `/opt/ansible/roles`:

```bash
roles_path    = /opt/ansible/roles
```

##### Vault password file

Uncomment `vault_password_file` and add the `/opt/ansible/.vaultpasswordfile` path.

```bash
vault_password_file = /opt/ansible/.vaultpasswordfile
```

##### Hosts in GIT (optional)

If you want to keep the management of your hosts under a GIT repository, I will suggest that you fork this repository and change this line under your `/etc/ansible/ansible.cfg` file:

```bash
inventory      = /opt/ansible/hosts
```

> If you don't fork this GIT repository and make any changes on the `hosts` file, these changes will be replace by a newer version the next time you update the project with the `git pull` command in the `/opt/ansible` folder!

#### Test the connectivity to your hosts

To test if everything is ok, you can `ping` your hosts.

```bash
ansible all -m ping
```

Output:

```bash
gocd | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
agent01 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
agent02 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
agent03 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

#### Install the Ansible Playbook Roles

```bash
cd /opt/ansible
```

##### Homebrew Ansible Role

```bash
ansible-galaxy install geerlingguy.homebrew
```

##### macOS Software Updates Role

This role includes macOS security updates and also
software updates of apps installed with the Mac App Store

```bash
ansible-galaxy install aadl.softwareupdate
```

##### Xcode Ansible Role

This role install Xcode BUT it doesn't download it or update it.

You need to store the XPI file on a network drive (NFS, SMB, etc.)
or temporary folder before running it.

```bash
ansible-galaxy install macstadium.xcode
```

## Go.CD Agent(s)

### Install Homebrew

[Homebrew](https://brew.sh/) is the "The missing package manager for macOS" and it will helps to install some software automatically.

You need to run this command in a terminal on each of your Go.CD agents:

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

## Run the playbooks (from the server)

### Run them manually

#### Homebrew

```bash
ansible-playbook /opt/ansible/playbooks/homebrew.yml
```

#### Android SDK

##### Android SDK with API from 21 to 28

```bash
ansible-playbook /opt/ansible/playbooks/android/sdkInstall.yml
```

##### Android SDK emulators (optional)

```bash
ansible-playbook /opt/ansible/playbooks/android/emulatorsInstall.yml
```

#### Node.js (via nvm)

```bash
ansible-playbook /opt/ansible/playbooks/nvm.yml
```

#### Titanium SDK

```bash
ansible-playbook /opt/ansible/playbooks/titanium/sdkInstall.yml
```

#### Appium (for UI Automation)

```bash
ansible-playbook /opt/ansible/playbooks/appium.yml
```

#### macOS Software Updates

```bash
ansible-playbook /opt/ansible/playbooks/macos.yml
```

#### Automate the run with CRON

Ideally, you Ansible playbooks need to be run automatically and nothing is simplier than a CRON job for that!

Update the `crontab.txt` file to fill your own time preferences and run this command:

```bash
crontab /opt/ansible/crontab.txt
```

Content of the `crontab.txt`:

```bash
# Ansible - Update Homebrew packages on all agents every day at 1:00 AM
0 1 * * * echo -e " \n #################$(date)################# \n" >> /opt/ansible/logs/homebrew.log ; ansible-playbook /opt/ansible/playbooks/homebrew.yml >> /opt/ansible/logs/homebrew.log

# Ansible - Update Android SDK packages on all agents every day at 2:00 AM
0 2 * * * echo -e " \n #################$(date)################# \n" >> /opt/ansible/logs/androidSdkUpdate.log ; ansible-playbook /opt/ansible/playbooks/android/sdkUpdate.yml >> /opt/ansible/logs/androidSdkUpdate.log

# Ansible - Update Titanium SDK packages on all agents every day at 3:00 AM
0 3 * * * echo -e " \n #################$(date)################# \n" >> /opt/ansible/logs/titaniumSdkUpdate.log ; ansible-playbook /opt/ansible/playbooks/titanium/sdkUpdate.yml >> /opt/ansible/logs/titaniumSdkUpdate.log

# Ansible - Update Appium package on all agents every Monday at 3:15 AM
15 3 * * MON echo -e " \n #################$(date)################# \n" >> /opt/ansible/logs/appium.log ; ansible-playbook /opt/ansible/playbooks/appium.yml >> /opt/ansible/logs/appium.log

# Ansible - Apply macOS Software updates on all agents every day at 3:30 AM
30 3 * * * echo -e " \n #################$(date)################# \n" >> /opt/ansible/logs/macos.log ; ansible-playbook /opt/ansible/playbooks/macos.yml >> /opt/ansible/logs/macos.log

# Ansible - Reboot all agents every day at 5:00 AM
00 5 * * * echo -e " \n #################$(date)################# \n" >> /opt/ansible/logs/reboot.log ; ansible-playbook /opt/ansible/playbooks/reboot.yml >> /opt/ansible/logs/reboot.log
```

## TODO

* Add instructions for the XCode install
* Add the Terminal app into the macOS Accessibility permission (to launch XCode Organiser)
* Prevent the macOS apps to reopen after a reboot
* Create a script for the XCode update that remove the current version
* Create an Ansible playbook to install/update Genymotion + default VMs
* Create an Ansible playbook to install/update the Go.CD agent software
* Create an Ansible playbook to install/update the Go.CD server software
* Create an Ansible playbook to install/update a your Mac with the same settings as the agents
* Create an Ansible role instead of multiple playbooks + a playbook that will configure what we want ot install/configure
