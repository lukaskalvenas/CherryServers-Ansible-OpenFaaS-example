![Cherry Servers](https://www.serchen.com/images/thumbnails/large/54097.jpg)
# CherryServers-Ansible-OpenFaaS-example
This example will use Ubuntu as the base operating system to deploy one master node and a user-specified amount (e.g. two) worker nodes via Docker swarm. Those will then automatically join the master node via public IP address and Docker swarm token combination. 
# Prerequisites
<ul>
  <li><a href="https://www.ansible.com/" target="_blank">Ansible</a></li>
  <li><a href="https://stedolan.github.io/jq/download/" target="_blank">JQ package for the host PC/laptop</a></li>
  <li>The Cherry Servers module connects to Cherry Servers Public API via cherry-python package. You need to install it with pip (this might need to be done as sudo):</li>
  
  ```
    pip install cherry-python --user
  ```

You will need to download <a href="https://github.com/cherryservers/cherry-ansible-module/tree/master/cherryservers">this</a> directory into the <b>ansible/library</b> subdirectory of this project.
This is the Cherry Servers Ansible Module that we will use to interact with Cherry Server's API. The ansible.cfg file in the ansible directory has a library entry that points to the library subdirectory and tells ansible where to find our custom modules.
</ul>

# Before you start
You will need a <a href="https://portal.cherryservers.com" target="_blank">cherrservers account</a> with credit in balance to order services with hourly billing. 

Create an API key at <a href="https://portal.cherryservers.com/#/settings/api-keys/" target="_blank">https://portal.cherryservers.com/#/settings/api-keys/</a> and export it to your working terminal session<br>
```
export CHERRY_AUTH_TOKEN="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXUyJ9"
```
Configure the global variables on the "group_vars/all.yml" file. Download and run the <a href="https://github.com/cherryservers/cherryctl" target="_blank">cherryctl</a> script to get a list of server plan IDs.
```
project_id: 88513
plan_id: 94
region: EU-East-1
number_of_masters: 1
number_of_workers: 2
image: 'Ubuntu 16.04 64bit'
key_file: '/home/lukas/.ssh/id_rsa'
key_file_pub: "{{ key_file }}.pub"
```

Last, but not least, make sure that all bash scripts in the Ansible working directory have an "execute" flag:

```
sudo chmod +x *.sh
```

# Adding your SSH key to CherryServers

Run  "ansible-playbook ssh_add_keys.yml" playbook to upload your SSH key to CherryServers. Please note that by this time you should already have exported your CherryServers API token, otherwise none of the playbooks will run.

# How to use

Once you're ready, execute "ansible-playbook deploy_openfaas.yml" playbook. The full process may take up to 20 minutes to complete. For detailed playbook output, run "ansible-playbook -vvv deploy_openfaas.yml"

It will first deploy the master node and register all the necessary variables. Once that's done, the specified amount of worker servers will follow to deploy. 

The worker servers will then be automatically added to the Docker swarm. When the playbook finishes, log into the master node, change the working directory to "~/faas" and run the "deploy_stack.sh" script. 

This will install the default OpenFaaS function stack for you. Use the provided login credentials to access the master GUI control panel at http://$master_ip:8080 and begin working.

Good luck!


# When no longer needed
```
ansible-playbook server_terminate.yml
```
You may need to edit the playbook according to your preference to terminate all servers succesfully. Keep in mind that all Docker swarm members will be terminated and the data will get permanently wiped.
