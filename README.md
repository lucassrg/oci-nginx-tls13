# Setup TLSv1.3 on nginx with Ansible

In this document we are going to install nginx, setup TLSv1.3 with a valid certificate based on Let's Encrypt and deploy a sample nodejs application with support to TLSv1.3.


## Prerequisites

- OCI Tenancy
- Compute Instance running with `Oracle Linux 8` or `Ubuntu 20`
- SSH Access to the Compute instance via Public IP
- Access to OCI Cloud Shell or a Linux box to run python/ansible
- Python 3

## Cloud Shell

Oracle Cloud Infrastructure (OCI) [Cloud Shell](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellintro.htm) is a web browser-based terminal, accessible from the OCI Console. The Cloud Shell provides access to a Linux shell, with a pre-authenticated Oracle Cloud Infrastructure CLI and other useful tools. You can open up Cloud Shell directly from the OCI Console header.

## Clone git repository

1. Clone the git repository:
    ```bash
    git clone https://github.com/lucassrg/oci-nginx-tls13.git oci-nginx-tls13
    ```
1. Go to the project workspace
    ```bash
    cd oci-nginx-tls13/ansible
    ```

## Install Ansible on Python Virtual Environment

A Python virtual environment is a tool used to create an isolated environment for Python projects. It's useful to avoid dependencies conflicts with other projects that relies on different package versions.



1. Create a directory for your virtual environments, e.g. `mkdir virtual_envs`

1. Install virtualenv with python3: `python3 -m venv virtual_envs`

1. Activate it:
    ```
    source virtual_envs/bin/activate
    ```
    You will notice that a "prefix" `(virtual_envs)` will show up in front of your shell.

    At anytime, to disable virtualenv just run `deactivate`

1. (Optional) Upgrade pip3
    ```
    pip3 install --upgrade pip
    ```

1. Install Ansible, OCI Python SDK and additional dependencies.
    ```
    pip3 install -r requirements/requirements.txt
    ```
    Output:
    ```
    Collecting ansible
    Using cached ansible-4.4.0.tar.gz (35.4 MB)
    Collecting oci
    Downloading oci-2.43.2-py2.py3-none-any.whl (10.1 MB)
        |████████████████████████████████| 10.1 MB 11.5 MB/s 
    Collecting ansible-core<2.12,>=2.11.3
    Using cached ansible-core-2.11.3.tar.gz (6.8 MB)
    Collecting cryptography<=3.4.7,>=3.2.1
    Downloading cryptography-3.4.7-cp36-abi3-manylinux2014_x86_64.whl (3.2 MB)
        |████████████████████████████████| 3.2 MB 1.7 MB/s 
    Collecting pyOpenSSL<=19.1.0,>=17.5.0
    Using cached pyOpenSSL-19.1.0-py2.py3-none-any.whl (53 kB)
    Collecting python-dateutil<3.0.0,>=2.5.3
    Downloading python_dateutil-2.8.2-py2.py3-none-any.whl (247 kB)
        |████████████████████████████████| 247 kB 34.2 MB/s 
    Collecting pytz>=2016.10
    Using cached pytz-2021.1-py2.py3-none-any.whl (510 kB)
    Collecting certifi
    Downloading certifi-2021.5.30-py2.py3-none-any.whl (145 kB)
        |████████████████████████████████| 145 kB 29.3 MB/s 
    Collecting configparser==4.0.2
    Using cached configparser-4.0.2-py2.py3-none-any.whl (22 kB)
    Collecting jinja2
    Using cached Jinja2-3.0.1-py3-none-any.whl (133 kB)
    Collecting PyYAML
    Using cached PyYAML-5.4.1-cp36-cp36m-manylinux1_x86_64.whl (640 kB)
    Collecting packaging
    Downloading packaging-21.0-py3-none-any.whl (40 kB)
        |████████████████████████████████| 40 kB 7.7 MB/s 
    Collecting resolvelib<0.6.0,>=0.5.3
    Downloading resolvelib-0.5.4-py2.py3-none-any.whl (12 kB)
    Collecting cffi>=1.12
    Downloading cffi-1.14.6-cp36-cp36m-manylinux1_x86_64.whl (401 kB)
        |████████████████████████████████| 401 kB 38.4 MB/s 
    Collecting pycparser
    Using cached pycparser-2.20-py2.py3-none-any.whl (112 kB)
    Collecting six>=1.5.2
    Using cached six-1.16.0-py2.py3-none-any.whl (11 kB)
    Collecting MarkupSafe>=2.0
    Downloading MarkupSafe-2.0.1-cp36-cp36m-manylinux2010_x86_64.whl (30 kB)
    Collecting pyparsing>=2.0.2
    Downloading pyparsing-2.4.7-py2.py3-none-any.whl (67 kB)
        |████████████████████████████████| 67 kB 8.4 MB/s 
    Using legacy 'setup.py install' for ansible, since package 'wheel' is not installed.
    Using legacy 'setup.py install' for ansible-core, since package 'wheel' is not installed.
    Installing collected packages: pycparser, pyparsing, MarkupSafe, cffi, six, resolvelib, PyYAML, packaging, jinja2, cryptography, pytz, python-dateutil, pyOpenSSL, configparser, certifi, ansible-core, oci, ansible
        Running setup.py install for ansible-core ... done
        Running setup.py install for ansible ... done
    Successfully installed MarkupSafe-2.0.1 PyYAML-5.4.1 ansible-4.4.0 ansible-core-2.11.3 certifi-2021.5.30 cffi-1.14.6 configparser-4.0.2 cryptography-3.4.7 jinja2-3.0.1 oci-2.43.2 packaging-21.0 pyOpenSSL-19.1.0 pycparser-2.20 pyparsing-2.4.7 python-dateutil-2.8.2 pytz-2021.1 resolvelib-0.5.4 six-1.16.0
    ```    

1. Install Ansible Collections
    ```
    ./virtual_envs/bin/ansible-galaxy collection install -f -r collections/requirements.yml -vvv
    ```
    Note: `-vvv` was used to increase the verbosity of the log messages.

    Output:
    ```
    [DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 3.6.8 (default, Mar  9 2021, 15:08:44) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44.0.3)]. This feature will be removed from ansible-core in 
    version 2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
    ansible-galaxy [core 2.11.3] 
    config file = None
    configured module search path = ['/home/lucas_gome/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
    ansible python module location = /home/lucas_gome/oci-nginx-tls13/ansible/virtual_envs/lib64/python3.6/site-packages/ansible
    ansible collection location = /home/lucas_gome/.ansible/collections:/usr/share/ansible/collections
    executable location = ./virtual_envs/bin/ansible-galaxy
    python version = 3.6.8 (default, Mar  9 2021, 15:08:44) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44.0.3)]
    jinja version = 3.0.1
    libyaml = True
    No config file found; using defaults
    Reading requirement file at '/home/lucas_gome/oci-nginx-tls13/ansible/collections/requirements.yml'
    Starting galaxy collection install process
    Process install dependency map
    Created /home/lucas_gome/.ansible/galaxy_token
    Starting collection install process
    Downloading https://galaxy.ansible.com/download/oracle-oci-2.27.0.tar.gz to /home/lucas_gome/.ansible/tmp/ansible-local-1850q_ih1yv8/tmpcnl7f0p6/oracle-oci-2.27.0-_w7nu5b5
    Collection 'oracle.oci:2.27.0' obtained from server default https://galaxy.ansible.com/api/
    Installing 'oracle.oci:2.27.0' to '/home/lucas_gome/.ansible/collections/ansible_collections/oracle/oci'
    oracle.oci:2.27.0 was installed successfully
    ```    

## Playbook Setup

### OCI Dynamic Inventory file 

The Oracle Cloud Infrastructure [inventory plugin](https://docs.oracle.com/en-us/iaas/Content/API/SDKDocs/ansibledynamicinventoryplugin.htm) allow you to define the data sources used to compile an inventory of hosts that Ansible uses to target tasks.

The inventory configuration file [inventory.oci.yml](.\inventory\inventory.oci.yml) is located on `inventory\inventory.oci.yml`. The default inventory variable file `vars.yml` is setup for connecting to Oracle Linux hosts. There's an additional variable file for Ubuntu `\inventory\ubuntu\vars.yml` which should be used to connect to Ubuntu hosts. 

We need to make some changes to the inventory file to reference our environment. Typically we need to define how the plugin authenticates with the OCI account and which region and compartment it will look up for instances.

However, when running Ansible from Cloud Shell, we don't need to specify how the plugin authenticates, since Cloud Shell authenticates automatically with the current OCI logged user session and it injects the authentication mode from the environment variables.

When running ansible from outside Cloud Shell, you need to setup the oci config file if you haven't done yet. One easy way to generate the config file is using the OCI CLI, running the command below, which will prompt you to complete the wizard to generate the config file:
```
oci setup config
```

Next, you need to declare the location of the configuration file and which profile you are using. Uncomment the following block of code in the inventory file and update it accordingly:
```
config_file: ~/.oci/config
config_profile: DEFAULT
```

Next, specify the region where your Compute instance is running. 
```
regions:
  - us-ashburn-1
```

Set the compartment where the instance is running. You can specify either a `compartment_ocid` or `compartment_name`. 

Finally, review the `filters` section. By default, the plugin will fetch all instances that are `RUNNING` and contain a [freeform_tag](https://docs.oracle.com/en-us/iaas/Content/Tagging/Concepts/taggingoverview.htm):

```
"role": "webserver"
```
If you have different tags, adjust the filter accordingly.

You can add tags to the Compute instance at [Launch ( *Show Advanced Options*)](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm#linux__linux-create) or [later](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm#tags).

### OCI Dynamic Inventory vars

We need to import the SSH private key file that Ansible will use to authenticate against the host in order execute all tasks.

1. First, we need to upload the SSH key via Cloud Shell. In the Cloud Shell (hamburger) menu, on the top left hand side corner of the Cloud Shell console, click on `Upload`.

1. Drop a file or upload it. It will be saved in the Cloud Shell home directory.

1. Update the variable `ansible_ssh_private_key_file` in the `inventory\vars.yml` with the path to the file, e.g.:
```
all:
  vars:
    ansible_user: "opc"
    ansible_ssh_private_key_file: "/home/<oci_user>/<ssh_private_key_file>"
```


###  Check Dynamic inventory

You can check the hosts using `ansible-inventory` command. 

To list *Oracle Linux* based hosts run the following command:
```
./virtual_envs/bin/ansible-inventory -i inventory --list
```
or to show a graph:
```
./virtual_envs/bin/ansible-inventory -i inventory --graph
```

To list *Ubuntu* based hosts, you need to include one additional inventory file:
```
./virtual_envs/bin/ansible-inventory -i inventory -i inventory/ubuntu --list
```
or to show a graph:
```
./virtual_envs/bin/ansible-inventory -i inventory -i inventory/ubuntu --graph
```

Note: The dynamic plugin do not filter hosts by Linux Distribution, however, the inventory define the user that SSH to the instance when running the playbook.


## Running the Playbook

The playbook is responsible for setting up Linux firewall policies, OS security, SSL certificate (managed by Let's Encrypt), nginx as a reverse proxy, TLSv1.3 and also deploys a sample "Hello World" nodejs application which outputs a `Hello World` string.

There are 3 variables file on the playbook:

- `default.yml`: contains domain and SSL certificate variables
- `os_OracleLinux8.yml`: contains variables with the package names/versions used on Oracle Linux 8.
- `os_Ubuntu20.yml`: contains variables with the package names/versions used on Ubuntu 20.

By default, the playbook use the public IP address of the instance to generate a valid certificate, using free DNS wildcard services like nip.io, sslip.io or pseudo.host. 

If the public IP of the compute instance is already associated with a DNS "A" record, assign the DNS domain name to the `domain` variable in the default variables file. 

To run the playbook, use the instructions below for each operating system. 

###  Oracle Linux 8

```bash
./virtual_envs/bin/ansible-playbook -i inventory main.yml -vvv
```
###  Ubuntu 20
```bash
./virtual_envs/bin/ansible-playbook -i inventory -i inventory/ubuntu main.yml -vvv
```
