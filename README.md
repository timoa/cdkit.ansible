# [CDKit](https://github.com/timoa/cdkit): Ansible

*[CDKit](https://github.com/timoa/cdkit) is a framework that helps to deploy mobile apps (iOS and Android) to the app stores (iTunes and Goole Play).*

## Getting started

[Ansible](https://www.ansible.com/) is a configuration manager that let you run scripts (playbooks) on a small to large number of computers.

It's agentless and needs only a SSH connexion to the computers you want to update.

We will use Ansible to install and maintain the Android SDK, Java 8, Fastlane, SonarQube scanner, ImageMagick, etc. on the GO.CD agents

## Prerequisites

Before you start, you need to have followed the steps to install the GO.CD server and agent(s)

## Server

### Install Ansible

#### Linux

Please refer to this page to [install Ansible for your Linux distribution](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-the-control-machine)

#### macOS

If you installed previously Homebrew, let's use it to install Ansible with this command:

``` bash
brew install ansible
```

### Configure Ansible

#### Checkout this repo

After you installed Ansible, you need to checkout this repository to the folder `/opt/ansible` (both Linux or macOS):

``` bash
[sudo] mkdir /opt/ansible
git clone git@github.com:timoa/cdkit.ansible.git
```

#### Configure your hosts

In case of the `hosts` file has not been packaged with your Ansible installation, you can use the template in this GIT repository or the folder `/opt/ansible/hosts`.

``` bash
[sudo] cp /opt/ansible/hosts /etc/ansible/hosts
```

#### Create the Vault password file

To save securely your hosts credentials, we will use an Ansible vault password file.

To create it, just type this command:

``` bash
ansible-vault create /opt/ansible/vaultpasswordfile.yml
```


#### Edit the Ansible configuration file

Now, your need to open the `ansible.cfg` file to let Ansible knwo that we use a different location for our playbook, roles, etc.

``` bash
[sudo] vi /etc/ansible/ansible.cfg
```

##### Roles

Change the roles path to `/opt/ansible/roles`:

``` bash
roles_path    = /opt/ansible/roles
```

##### Vault password file

Uncomment `vault_password_file` and add the `/opt/ansible/vaultpasswdfile.yml` path.

``` bash
vault_password_file = /opt/ansible/vaultpasswdfile.yml
```

##### Hosts in GIT (optional)

If you want to keep the management of your hosts under a GIT repository, I will suggest that you fork this repository and change this line under your `/etc/ansible/ansible.cfg` file:

``` bash
inventory      = /opt/ansible/hosts
```

> If you don't fork this GIT repository and make any changes on the `hosts` file, these changes will be replace by a new version next time you update the project with the `git pull` command in the `/opt/ansible` folder!

#### Test the connectivity to your hosts

To test if everything is ok, you can `ping` your hosts.

``` bash
ansible all -m ping
```

Output:

``` bash
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

``` bash
cd /opt/ansible
```

##### Homebrew playbook

``` bash
ansible-galaxy install geerlingguy.homebrew
```

##### Java 8 playbook (Oracle)

``` bash
ansible-galaxy install srsp.oracle-java
```

## GO.CD Agent(s)

### Install Homebrew

[Homebrew](https://brew.sh/) is the "The missing package manager for macOS" and it will helps to install some software automatically.

You need to run this command in a terminal on each of your GO.CD agents:

``` bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

## Run the playbooks (from the server)

### Run them manually

#### Homebrew

``` bash
ansible-playbook /opt/ansible/playbooks/homebrewUpdate.yml
```

#### Java JDK

``` bash
ansible-playbook /opt/ansible/playbooks/javaUpdate.yml
```

#### Android SDK

``` bash
ansible-playbook /opt/ansible/playbooks/androidSdkPkgUpdate.yml
```

#### Automate the run with CRON

Ideally, you Ansible playbooks need to be run automatically and nothing is simplier than a CRON job for that!

Update the `crontab.txt` file to fill your own time preferences and run this command:

``` bash
crontab /opt/ansible/crontab.txt
```

Content of the `crontab.txt`:

``` bash
# Ansible - Update Homebrew packages in all GO.CD agents every day at 1:00 am
0 0 1 * * echo -e " \n #################$(date)################# \n" >> /opt/ansible/logs/homebrewUpdate.log ; ansible-playbook /opt/ansible/playbooks/homebrewUpdate.yml >> /opt/ansible/logs/homebrewUpdate.log

# Ansible - Update Java JDK packages in all GO.CD agents every Monday at 2:00 am
0 0 2 * MON echo -e " \n #################$(date)################# \n" >> /opt/ansible/logs/javaUpdate.log ; ansible-playbook /opt/ansible/playbooks/javaUpdate.yml >> /opt/ansible/logs/javaUpdate.log

# Ansible - Update Android SDK packages in all GO.CD agents every day at 3:00 am
0 0 3 * * echo -e " \n #################$(date)################# \n" >> /opt/ansible/logs/androidSdkPkgUpdate.log ; ansible-playbook /opt/ansible/playbooks/androidSdkPkgUpdate.yml >> /opt/ansible/logs/androidSdkPkgUpdate.log
```

## TODO

* Create a playbook role to install GO.CD agent software
* Add the Xcode playbook role to install/update Xcode on the GO.CD agents
