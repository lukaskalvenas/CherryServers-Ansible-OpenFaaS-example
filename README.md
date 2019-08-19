![Cherry Servers](https://www.serchen.com/images/thumbnails/large/54097.jpg)
# CherryServers-Ansible-OpenFaaS-example
This example will use Ubuntu as the base operating system to deploy one master node and a user-specified amount (e.g. two) worker nodes on Docker swarm. Those will then automatically join the master node via public IP address and Docker swarm token combination. 
# Requirements
<ul>
  <li><a href="https://www.ansible.com/" target="_blank">Ansible</a></li>
  <li><a href="https://download.docker.com/linux/ubuntu/dists/zesty/pool/stable/amd64/docker-ce_17.12.0~ce-0~ubuntu_amd64.deb" target="_blank">Docker CE.</a> Donwload the file into Ansible's working directory.</li>
  <li>The Cherry Servers module connects to Cherry Servers Public API via cherry-python package. You need to install it with pip (this might need to be done as sudo):</li>
  
  ```
    pip install cherry-python --user
  ```
</ul>

# Before you start
You will need a <a href="https://portal.cherryservers.com" target="_blank">cherrservers account</a> with credit in balance to order services with hourly billing. 

Create an API key at <a href="https://portal.cherryservers.com/#/settings/api-keys/" target="_blank">https://portal.cherryservers.com/#/settings/api-keys/</a> and export it to your working terminal session<br>
```
export CHERRY_AUTH_TOKEN="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXUyJ9"
```
Configure the global variables on the "group_vars/all.yml" file. Download and run the <a href="https://github.com/cherryservers/cherryctl" target="_blank">cherryctl</a> script to get a list of server plan IDs.
```
$ vim Downloads/CherryServers-Ansible-OpenFaaS-example-master/group_vars/all.yml
project_id: 88513
plan_id: 161
region: EU-East-1
number_of_masters: 1
number_of_workers: 2
image: 'Ubuntu 16.04 64bit'
key_file: '/home/lukas/.ssh/id_rsa'
key_file_pub: "{{ key_file }}.pub"
master_hostname: master.node%02d
worker_hostname: worker.node%02d
```

Last, but not least, make sure that all bash scripts in the Ansible working directory have an "execute" flag:

```
$ cd Downloads/CherryServers-Ansible-OpenFaaS-example-master/
sudo chmod +x *.sh
```
Please note that by this time you should already have exported your CherryServers API token, otherwise none of the playbooks will run.

# Adding your SSH key to CherryServers

Run  "ansible-playbook ssh_add_keys.yml" playbook to upload your SSH key to CherryServers. 
```
$ cd Downloads/CherryServers-Ansible-OpenFaaS-example-master/
$ ansible-playbook ssh_add_keys.yml 
 [WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not
match 'all'

PLAY [Cherry Servers API] ***************************************************************************************

TASK [Gathering Facts] ******************************************************************************************
ok: [localhost]

TASK [Add SSH key] **********************************************************************************************
changed: [localhost]

PLAY RECAP ******************************************************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0   
```
# Running the deployment playbook

Before running the playbook, make sure you have the necessary files in Ansible's working directory
```
Downloads/CherryServers-Ansible-OpenFaaS-example-master$ tree .
.
├── ansible.cfg
├── docker-ce_17.12.0_ce-0_ubuntu_amd64.deb
├── group_vars
│   └── all.yml
├── hosts
├── install-openfaas.sh
├── library
│   ├── cherryservers_ips.py
│   ├── cherryservers_server.py
│   └── cherryservers_sshkey.py
├── openfaas_deploy.yml
├── README.md
├── server_terminate.yml
└── ssh_add_keys.yml

2 directories, 12 files
```
Once you're ready, execute "ansible-playbook openfaas_deploy.yml" playbook. The full process may take up to 20 minutes to complete. For detailed playbook output, run "ansible-playbook -vvv openfaas_deploy.yml"
```
$ cd Downloads/CherryServers-Ansible-OpenFaaS-example-master/
$ ansible-playbook openfaas_deploy.yml 
 [WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not
match 'all'


PLAY [OpenFaaS deployment on Cherry Servers] ********************************************************************

TASK [Gathering Facts] ******************************************************************************************
ok: [localhost]

TASK [Deploying master node] ************************************************************************************
changed: [localhost]

TASK [Register master node] *************************************************************************************
changed: [localhost]

TASK [set_fact] *************************************************************************************************
ok: [localhost]

PLAY [Preparing the master node and registering necessary variables] ***************
```

It will first deploy the master node and register all the necessary variables. Once that's done, the specified amount of worker servers will follow to deploy. 

The worker servers will then be automatically added to the Docker swarm. When the playbook finishes, log into the master node, change the working directory to "~/faas" and run the "deploy_stack.sh" script. 

This will install the default OpenFaaS function stack for you. Use the provided login credentials to access the master GUI control panel at http://$master_ip:8080 and begin working.

Good luck!


# When no longer needed
```
$ cd Downloads/CherryServers-Ansible-OpenFaaS-example-master/
$ ansible-playbook server_terminate.yml

 [WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not
match 'all'


PLAY [Cherry Servers API module] ********************************************************************************

TASK [Gathering Facts] ******************************************************************************************
ok: [localhost]

TASK [Terminating master node(s)] *******************************************************************************
changed: [localhost]

TASK [Terminating worker node(s)] *******************************************************************************
changed: [localhost]

PLAY RECAP ******************************************************************************************************
localhost                  : ok=3    changed=2    unreachable=0    failed=0 

```
You may need to edit the playbook according to your preference to terminate all servers succesfully. Keep in mind that all Docker swarm members will be terminated and the data will get permanently wiped.
