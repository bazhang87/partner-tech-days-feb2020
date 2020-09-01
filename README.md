# Red Hat Partner Tech Days


Greetings. This is the main page for Red Hat Partner Tech Days - Feb 2020. These 1-day partner events are run in various cities across North America with the main landing/registration page located here:

https://redhat-events.com/PartnerTraining/workshops

The intent is to enable Red Hat Business Partners with updates on our latest products/solutions and deliver capabilities through self-paced open labs.  

## Agenda

* **8:30 a.m. – 9:00 a.m.**	*Check-In and Continental Breakfast*
* **9:00 a.m. – 10:20 a.m.**	Red Hat Infrastructure Migration Solution
* **10:20 a.m. – 10:30 a.m.**	*Break*
* **10:30 a.m. – 12:00 p.m.**	Microsoft SQL Server/RHEL

* **12:00 p.m. – 12:30 p.m.**	*Networking Lunch*

* **12:30 a.m. – 1:30 p.m.**	Building OpenShift Applications
* **1:30 p.m. – 3:30 p.m.**	Open Labs

* **3:30 p.m. – 5:00 p.m.**	*Happy Hour/Reception*


## Content

  * Presentation Decks & Open Labs Guides
    - [IMS](https://github.com/redhat-partner-tech/partner-tech-days-feb2020/tree/master/IMS) - Deck & Self-paced Red Hat Infrastructure Migration Solution lab 
    - [SQL-Server-RHEL](https://github.com/redhat-partner-tech/partner-tech-days-feb2020/tree/master/SQL-Server-RHEL) - Deck & Self-paced SQL Server 2019 on RHEL 8 lab 
    - [OpenShift-AppDev](https://github.com/redhat-partner-tech/partner-tech-days-feb2020/tree/master/OpenShift-AppDev) - Deck & Self-paced OpenShift 4 w/Quarkus & BPA lab 


#### Section 1 : Exploring the lab environment (10 minutes) - done

#### Section 2 : Quick intro to F5 Load Balancer (15 minutes) - done

#### Section 3 : Projects, Template, Tower, Teams & Users (10 minutes) - work in progress

#### Section 4 : Orchestration of Linux and F5, with Job/Workflow, survey and Approvals (10 minutes) - work in progress

Plan: this is the lab where the student will build a playbook where:

-   A developer would launch a job to add node3 to the webserver pool in F5

-   The job would then install webserver software on node3, create a proper HTML index file, start the webserver, add the webserver to the F5 pool, but this all won't go without an approval from an approval role which was set in section 3.

#### Section 5 : ??

#### Section 1 : Exploring the lab environment (10 minutes)

Objective: A quick introduction to Ansible command line for students with no experience with Ansible. Those with prior knowledge of Ansible might breeze through this section.

![](https://lh3.googleusercontent.com/gdPi31Dc0UGVOCHPkbgkfzt4jSr84JqyhLCVG8kgMdxkqU--7OeCNk2Bl8-voFvozhnPWP0xkDyybiqm97eWgEEAJuJbEi2uw5IDM4OEvc7Qz2EExmFIROlIfLgCXfVK9plgTbKl)

#### Step 1

Navigate to the home directory.

[student1@ansible ~]$

#### Step 2

Run the ansible command with the --version command to look at what is configured:

[student1@ansible ~]$ ansible --version

ansible 2.6.2

  config file = /home/student1/.ansible.cfg

  configured module search path = [u'/home/student1/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']

  ansible python module location = /usr/lib/python2.7/site-packages/ansible

  executable location = /usr/bin/ansible

  python version = 2.7.5 (default, May  3 2017, 07:55:04) [GCC 4.8.5 20150623 (Red Hat 4.8.5-14)]

Note: The Ansible version you see might differ from the above output

This command gives you information about the version of Ansible, location of the executable, version of Python, search path for the modules and location of the ansible configuration file.

#### Step 3

Use the cat command to view the contents of the ansible.cfg file.

```[student1@ansible ~]$ cat ~/.ansible.cfg

[defaults]

stdout_callback = yaml

connection = smart

timeout = 60

deprecation_warnings = False

host_key_checking = False

retry_files_enabled = False

inventory = /home/student1/lab_inventory/hosts

[persistent_connection]

connect_timeout = 200

command_timeout = 200

[student1@ansible ~]$
```

Note the following parameters within the ansible.cfg file:

-   inventory: shows the location of the ansible inventory being used

#### Step 4

The scope of a play within a playbook is limited to the groups of hosts declared within an Ansible inventory. Ansible supports multiple [inventory](http://docs.ansible.com/ansible/latest/intro_inventory.html) types. An inventory could be a simple flat file with a collection of hosts defined within it or it could be a dynamic script (potentially querying a CMDB backend) that generates a list of devices to run the playbook against.

In this lab you will work with a file based inventory written in the ini format. Use the cat command to view the contents of your inventory:

```[student1@ansible ~]$ cat ~/lab_inventory/hosts```

The output will look as follows with student2 being the respective student workbench:

```node3 ansible_host=54.152.203.104 ansible_user=ec2-user private_ip=172.16.182.197

node4 ansible_host=100.25.219.137 ansible_user=ec2-user private_ip=172.16.247.51

[all:vars]

ansible_user=student2

ansible_ssh_pass=ansible

ansible_port=22

[lb]

f5 ansible_host=34.199.128.69 ansible_user=admin private_ip=172.16.26.136 ansible_ssh_pass=admin

[control]

ansible ansible_host=107.23.192.217 ansible_user=ec2-user private_ip=172.16.207.49

[web]

host1 ansible_host=107.22.141.4 ansible_user=ec2-user private_ip=172.16.170.190

host2 ansible_host=54.146.162.192 ansible_user=ec2-user private_ip=172.16.160.13
```

Note that the IP addresses will be different in your environment. Notice that node3 and node4 are not in the web group. We will use node3 and node4 in later exercises.

#### Step 5

In the above output every [ ] defines a group. For example [web] is a group that contains the hosts host1 and host2.

Note: A group called all always exists and contains all groups and hosts defined within an inventory.

We can associate variables to groups and hosts. Host variables are declared/defined on the same line as the host themselves. For example for the host f5:
```
f5 ansible_host=34.199.128.69 ansible_user=admin private_ip=172.16.26.136 ansible_ssh_pass=admin
```
-   f5 - The name that Ansible will use. This can but does not have to rely on DNS

-   ansible_host - The IP address that ansible will use, if not configured it will default to DNS

-   ansible_user - The user ansible will use to login to this host, if not configured it will default to the user the playbook is run from

-   private_ip - This value is not reserved by ansible so it will default to a [host variable](http://docs.ansible.com/ansible/latest/intro_inventory.html#host-variables). This variable can be used by playbooks or ignored completely.

-   ansible_ssh_pass - The password ansible will use to login to this host, if not configured it will assume the user the playbook ran from has access to this host through SSH keys.

Does the password have to be in plain text? No, Red Hat Ansible Tower can take care of credential management in an easy to use web GUI or a user may use [ansible-vault](https://docs.ansible.com/ansible/latest/network/getting_started/first_inventory.html#protecting-sensitive-variables-with-ansible-vault)

Go back to the home directory, all the following exercises will be performed in the home directory.

[student1@ansible ~]$ cd ~

#### Section 2 : Quick intro to F5 Load Balancer (15 minutes)

Objective: To cover the basics of a load balancer, in this case an F5 BigIP is used as an example of a load balancer and a network device that Ansible can control and automate.

#### Step 1 - Gather Facts

Using your text editor of choice create a new file called bigip-facts.yml.
```
[student1@ansible ~]$ nano bigip-facts.yml
```
vim and nano are available on the control node, as well as Visual Studio

Ansible playbooks are YAML files. YAML is a structured encoding format that is also extremely human readable (unlike it's subset - the JSON format).

Enter the following play definition into bigip-facts.yml:

```
---

- name: GRAB F5 FACTS

  hosts: f5

  connection: local

  gather_facts: no

-   The --- at the top of the file indicates that this is a YAML file.

-   The hosts: f5, indicates the play is run only on the F5 BIG-IP device

-   connection: local tells the Playbook to run locally (rather than SSHing to itself)

-   gather_facts: no disables facts gathering. We are not using any fact variables for this playbook.

Next, add the first task. This task will use the bigip_device_info module to grab useful information from the BIG-IP device.

 tasks:

- name: COLLECT BIG-IP FACTS

      bigip_device_info:

        gather_subset:

- system-info

        provider:

          server: "{{private_ip}}"

          user: "{{ansible_user}}"

          password: "{{ansible_ssh_pass}}"

          server_port: 8443

          validate_certs: no

      register: device_facts
```
A play is a list of tasks. Tasks and modules have a 1:1 correlation. Ansible modules are reusable, standalone scripts that can be used by the Ansible API, or by the ansible or ansible-playbook programs. They return information to ansible by printing a JSON string to stdout before exiting.

-   name: COLLECT BIG-IP FACTS is a user defined description that will display in the terminal output.

-   bigip_device_info: tells the task which module to use. Everything except register is a module parameter defined on the module documentation page.

-   The gather_subset: system_info parameter tells the module only to grab system level information.

-   The provider: parameter is a group of connection details for the BIG-IP.

-   The server: "{{private_ip}}" parameter tells the module to connect to the F5 BIG-IP IP address, which is stored as a variable private_ip in inventory

-   The user: "{{ansible_user}}" parameter tells the module the username to login to the F5 BIG-IP device with

-   Thepassword: "{{ansible_ssh_pass}}" parameter tells the module the password to login to the F5 BIG-IP device with

-   The server_port: 8443 parameter tells the module the port to connect to the F5 BIG-IP device with. 8443 is what's being used in this lab, but could be different depending on the deployment.

-   register: device_facts tells the task to save the output to a variable bigip_device_info

Next, append the second task to above . This task will use the debug module to print the output from device_facts variable we registered the facts to.

- name: DISPLAY COMPLETE BIG-IP SYSTEM INFORMATION

      debug:

        var: device_facts

-   The name: COMPLETE BIG-IP SYSTEM INFORMATION is a user defined description that will display in the terminal output.

-   debug: tells the task to use the debug module.

-   The var: device_facts parameter tells the module to display the variable bigip_device_info.

Finally let's append two more tasks to get more specific info from facts gathered, to the above playbook.

- name: DISPLAY ONLY THE MAC ADDRESS

      debug:

        var: device_facts['system_info']['base_mac_address']

- name: DISPLAY ONLY THE VERSION

      debug:

        var: device_facts['system_info']['product_version']

-   var: device_facts['system_info']['base_mac_address'] displays the MAC address for the Management IP on the BIG-IP device

-   device_facts['system_info']['product_version'] displays the product version BIG-IP device'''

Because the bigip_device_info module returns useful information in structured data, it is really easy to grab specific information without using regex or filters. Fact modules are very powerful tools to grab specific device information that can be used in subsequent tasks, or even used to create dynamic documentation (reports, csv files, markdown).

Run the playbook - exit back into the command line of the control host and execute the following:

```[student1@ansible ~]$ ansible-playbook bigip-facts.yml
```

The output will look as follows.

```
{% raw %}

[student1@ansible ~]$ ansible-playbook bigip-facts.yml

PLAY [GRAB F5 FACTS] ****************************************************************************************************************************************

TASK [COLLECT BIG-IP FACTS] *********************************************************************************************************************************

changed: [f5]

TASK [DISPLAY COMPLETE BIG-IP SYSTEM INFORMATION] ***********************************************************************************************************

ok: [f5] => {

    "device_facts": {

        "changed": true,

        "failed": false,

        "system_info": {

            "base_mac_address": "0a:54:53:51:86:fc",

            "chassis_serial": "685023ec-071e-3fa0-3849dcf70dff",

            "hardware_information": [

                {

                    "model": "Intel(R) Xeon(R) CPU E5-2676 v3 @ 2.40GHz",

                    "name": "cpus",

                    "type": "base-board",

                    "versions": [

                        {

                            "name": "cpu stepping",

                            "version": "2"

                        },

                        {

                            "name": "cpu sockets",

                            "version": "1"

                        },

                        {

                            "name": "cpu MHz",

                            "version": "2399.981"

                        },

                        {

                            "name": "cores",

                            "version": "2  (physical:2)"

                        },

                        {

                            "name": "cache size",

                            "version": "30720 KB"

                        }

                    ]

                }

            ],

            "marketing_name": "BIG-IP Virtual Edition",

            "package_edition": "Point Release 7",

            "package_version": "Build 0.0.1 - Tue May 15 15:26:30 PDT 2018",

            "platform": "Z100",

            "product_build": "0.0.1",

            "product_build_date": "Tue May 15 15:26:30 PDT 2018",

            "product_built": 180515152630,

            "product_changelist": 2557198,

            "product_code": "BIG-IP",

            "product_jobid": 1012030,

            "product_version": "13.1.0.7",

            "time": {

                "day": 15,

                "hour": 23,

                "minute": 46,

                "month": 4,

                "second": 25,

                "year": 2019

            },

            "uptime": 1738.0

        }

    }

}

TASK [DISPLAY ONLY THE MAC ADDRESS] *************************************************************************************************************************

ok: [f5] => {

    "device_facts['system_info']['base_mac_address']": "0a:54:53:51:86:fc"

}

TASK [DISPLAY ONLY THE VERSION] *****************************************************************************************************************************

ok: [f5] => {

    "device_facts['system_info']['product_version']": "13.1.0.7"

}

PLAY RECAP **************************************************************************************************************************************************

f5 : ok=4    changed=1    unreachable=0    failed=0v
```
And finally, please confirm that you have web access to your F5 load balancer. You can find its details in your Ansible inventory file. For example, for the entry below, you would go to [https://34.199.128.69:8443](http://34.199.128.69:8443) (the web interface listens to port 8443), username is admin, password is admin
```
[lb]

f5 ansible_host=34.199.128.69 ansible_user=admin private_ip=172.16.26.136 ansible_ssh_pass=admin
```

#### Step 2 - Add nodes

![](https://lh3.googleusercontent.com/zgXTQFiWu3e9IzWRQFj3uprL16ztn9yxJ2cVEkPyo9cOFxt-rru6y9Q2SkBv0Pb7YMCNtv4x8sInPEjS9eo4_D8y6iHYCzCZagfOM-q1nwKXd_4P-XE9FmlkljbyXT_IcBslktE9)

Using your text editor of choice create a new file called bigip-node.yml.
```
[student1@ansible ~]$ nano bigip-node.yml
```
vim and nano are available on the control node, as well as Visual Studio and Atom via RDP

Enter the following play definition into bigip-node.yml:
```
---

- name: BIG-IP SETUP

  hosts: lb

  connection: local

  gather_facts: false

  tasks:

- name: CREATE NODES

    bigip_node:

      provider:

        server: "{{private_ip}}"

        user: "{{ansible_user}}"

        password: "{{ansible_ssh_pass}}"

        server_port: 8443

        validate_certs: no

      host: "{{hostvars[item].ansible_host}}"

      name: "{{hostvars[item].inventory_hostname}}"

    loop: "{{ groups['web'] }}"
```
-   The --- at the top of the file indicates that this is a YAML file.

-   The hosts: lb, indicates the play is run only on the lb group. Technically there only one F5 device but if there were multiple they would be configured simultaneously.

-   connection: local tells the Playbook to run locally (rather than SSHing to itself)

-   gather_facts: false disables facts gathering. We are not using any fact variables for this playbook.

-   name: CREATE NODES is a user defined description that will display in the terminal output.

-   bigip_node: tells the task which module to use. Everything except loop is a module parameter defined on the module documentation page.

-   The server: "{{private_ip}}" parameter tells the module to connect to the F5 BIG-IP IP address, which is stored as a variable private_ip in inventory

-   The provider: parameter is a group of connection details for the BIG-IP.

-   The user: "{{ansible_user}}" parameter tells the module the username to login to the F5 BIG-IP device with

-   The password: "{{ansible_ssh_pass}}" parameter tells the module the password to login to the F5 BIG-IP device with

-   The server_port: 8443 parameter tells the module the port to connect to the F5 BIG-IP device with

-   The host: "{{hostvars[item].ansible_host}}" parameter tells the module to add a web server IP address already defined in our inventory.

-   The name: "{{hostvars[item].inventory_hostname}}" parameter tells the module to use the inventory_hostname as the name (which will be node1 and node2).

-   The validate_certs: "no" parameter tells the module to not validate SSL certificates. This is just used for demonstration purposes since this is a lab.

-   loop: tells the task to loop over the provided list. The list in this case is the group web which includes two RHEL hosts.

Run the playbook - exit back into the command line of the control host and execute the following:
```
[student1@ansible ~]$ ansible-playbook bigip-node.yml
```
The output will look as follows.
```
[student1@ansible]$ ansible-playbook bigip-node.yml

PLAY [BIG-IP SETUP] ************************************************************

TASK [CREATE NODES] ************************************************************

changed: [f5] => (item=node1)

changed: [f5] => (item=node2)

PLAY RECAP *********************************************************************

f5 : ok=1    changed=1    unreachable=0    failed=0
```
Confirm that you can see the nodes added in F5 by logging in to F5 via the web interface:

![](https://lh4.googleusercontent.com/X5RvMqmwzDN-_Juj52jO870oHilI-Gq8aixf9zGF_9LSOeBP0yuQIMe0AM21hikZRhmKIiuzCcD_pZBSBoPLH_daGLiJoLoyyG8fj7haAF2myFSXuiJa8M3-5xq6EdJJcEIg19AD)

#### Step 3 - Create a pool and add members

Enter the following play definition into bigip-pool.yml:
```
---

- name: BIG-IP SETUP

  hosts: lb

  connection: local

  gather_facts: false

  tasks:

- name: CREATE POOL

    bigip_pool:

      provider:

        server: "{{private_ip}}"

        user: "{{ansible_user}}"

        password: "{{ansible_ssh_pass}}"

        server_port: 8443

        validate_certs: no

      name: "http_pool"

      lb_method: "round-robin"

      monitors: "/Common/http"

      monitor_type: "and_list"
```
-   The --- at the top of the file indicates that this is a YAML file.

-   The hosts: lb, indicates the play is run only on the lb group. Technically there only one F5 device but if there were multiple they would be configured simultaneously.

-   connection: local tells the Playbook to run locally (rather than SSHing to itself)

-   gather_facts: false disables facts gathering. We are not using any fact variables for this playbook.

-   name: CREATE POOL is a user defined description that will display in the terminal output.

-   bigip_pool: tells the task which module to use.

-   The server: "{{private_ip}}" parameter tells the module to connect to the F5 BIG-IP IP address, which is stored as a variable private_ip in inventory

-   The provider: parameter is a group of connection details for the BIG-IP.

-   The user: "{{ansible_user}}" parameter tells the module the username to login to the F5 BIG-IP device with

-   The password: "{{ansible_ssh_pass}}" parameter tells the module the password to login to the F5 BIG-IP device with

-   The server_port: 8443 parameter tells the module the port to connect to the F5 BIG-IP device with

-   The name: "http_pool" parameter tells the module to create a pool named http_pool

-   The lb_method: "round-robin" parameter tells the module the load balancing method will be round-robin. A full list of methods can be found on the documentation page for bigip_pool.

-   The monitors: "/Common/http" parameter tells the module that the http_pool will only look at http traffic.

-   The monitor_type: "and_list" ensures that all monitors are checked.

-   The validate_certs: "no" parameter tells the module to not validate SSL certificates. This is just used for demonstration purposes since this is a lab.

Run the playbook - exit back into the command line of the control host and execute the following:
```
[student1@ansible ~]$ ansible-playbook bigip-pool.yml
```
The output will look as follows.
```
[student1@ansible ~]$ ansible-playbook bigip-pool.yml

PLAY [BIG-IP SETUP] ************************************************************

TASK [CREATE POOL] *************************************************************

changed: [f5]

PLAY RECAP *********************************************************************

f5 : ok=1    changed=1    unreachable=0    failed=0

Now that the pool has been created, let's add node1 and node2 into the pool.

Enter the following play definition into bigip-pool-members.yml:

---

- name: BIG-IP SETUP

  hosts: lb

  connection: local

  gather_facts: false

  tasks:

- name: ADD POOL MEMBERS

    bigip_pool_member:

      provider:

        server: "{{private_ip}}"

        user: "{{ansible_user}}"

        password: "{{ansible_ssh_pass}}"

        server_port: 8443

        validate_certs: no

      state: "present"

      name: "{{hostvars[item].inventory_hostname}}"

      host: "{{hostvars[item].ansible_host}}"

      port: "80"

      pool: "http_pool"

    loop: "{{ groups['web'] }}"
```
-   The --- at the top of the file indicates that this is a YAML file.

-   The hosts: lb, indicates the play is run only on the lb group. Technically there only one F5 device but if there were multiple they would be configured simultaneously.

-   connection: local tells the Playbook to run locally (rather than SSHing to itself)

-   gather_facts: false disables facts gathering. We are not using any fact variables for this playbook.

-   name: ADD POOL MEMBERS is a user defined description that will display in the terminal output.

-   bigip_pool_member: tells the task which module to use.

-   The server: "{{private_ip}}" parameter tells the module to connect to the F5 BIG-IP IP address, which is stored as a variable private_ip in inventory

-   The provider: parameter is a group of connection details for the BIG-IP.

-   The user: "{{ansible_user}}" parameter tells the module the username to login to the F5 BIG-IP device with

-   The password: "{{ansible_ssh_pass}}" parameter tells the module the password to login to the F5 BIG-IP device with

-   The server_port: 8443 parameter tells the module the port to connect to the F5 BIG-IP device with

-   The state: "present" parameter tells the module we want this to be added rather than deleted.

-   The name: "{{hostvars[item].inventory_hostname}}" parameter tells the module to use the inventory_hostname as the name (which will be host1 and host2).

-   The host: "{{hostvars[item].ansible_host}}" parameter tells the module to add a web server IP address already defined in our inventory.

-   The port: parameter tells the pool member port.

-   The pool: "http_pool" parameter tells the module to put this node into a pool named http_pool

-   The validate_certs: "no" parameter tells the module to not validate SSL certificates. This is just used for demonstration purposes since this is a lab. Finally there is a loop parameter which is at the task level (it is not a module parameter but a task level parameter:

-   loop: tells the task to loop over the provided list. The list in this case is the group web which includes two RHEL hosts.

Run the playbook - exit back into the command line of the control host and execute the following:
```
[student1@ansible ~]$ ansible-playbook bigip-pool-members.yml
```
The output will look as follows.
```
[student1@ansible ~]$ ansible-playbook bigip-pool-members.yml

PLAY [BIG-IP SETUP] ************************************************************

TASK [ADD POOL MEMBERS] ********************************************************

changed: [f5] => (item=host1)

changed: [f5] => (item=host2)

PLAY RECAP *********************************************************************

f5 : ok=1    changed=1    unreachable=0    failed=0
```
Confirm that you see the new pool created and its members by logging in to your F5 load balancer web interface.

![](https://lh3.googleusercontent.com/uTrq9zCI_EM6g16NbauJNxKdIAP2mBRSqp7TWJTl6pAOP5bSAl27NvtUW99g6E8tGkpggfiUoi892K2mCslioEkS2qM7zGecxskPE2J6G5eFg8peGwL1xMlaKpkXWU9ppulTAb3G)

![](https://lh5.googleusercontent.com/o12awsgoHQxrIN18_GPhUFVaXOjw3F82XPp-4f84MiTs7B9KgkfxAqhHXofI4pA6jCNKR5vDRQUb8kQ4IDuKbZpJCG1QwPED0WshZVAlsFf1RzlZRValOSDeYYMmUWhgvAk8qw_A)

#### Step 4 - Create a virtual server

A virtual server in F5 load balancer is a combination of IP and port number. When an incoming web request comes to this virtual server, the request will then be passed to the pool of nodes that is assigned to that virtual server. Who gets to answer the requests? That depends on the load balancing policy chosen, for example if "round robin" is used, each member node of the pool will take turn to answer requests. (Remember, we did set our pool to use round robin)

Now let's create a virtual server.

Enter the following play definition into bigip-virtual-server.yml:
```
---

- name: BIG-IP SETUP

  hosts: lb

  connection: local

  gather_facts: false

  tasks:

- name: ADD VIRTUAL SERVER

    bigip_virtual_server:

      provider:

        server: "{{private_ip}}"

        user: "{{ansible_user}}"

        password: "{{ansible_ssh_pass}}"

        server_port: 8443

        validate_certs: no

      name: "vip"

      destination: "{{private_ip}}"

      port: "443"

      enabled_vlans: "all"

      all_profiles: ['http','clientssl','oneconnect']

      pool: "http_pool"

      snat: "Automap"
```
-   The --- at the top of the file indicates that this is a YAML file.

-   The hosts: f5, indicates the play is run only on the F5 BIG-IP device

-   connection: local tells the Playbook to run locally (rather than SSHing to itself)

-   gather_facts: no disables facts gathering. We are not using any fact variables for this playbook.

-   name: ADD VIRTUAL SERVER is a user defined description that will display in the terminal output.

-   bigip_virtual_server: tells the task which module to use.

-   The server: "{{private_ip}}" parameter tells the module to connect to the F5 BIG-IP IP address, which is stored as a variable private_ip in inventory

-   The provider: parameter is a group of connection details for the BIG-IP.

-   The user: "{{ansible_user}}" parameter tells the module the username to login to the F5 BIG-IP device with

-   Thepassword: "{{ansible_ssh_pass}}" parameter tells the module the password to login to the F5 BIG-IP device with

-   The server_port: 8443 parameter tells the module the port to connect to the F5 BIG-IP device with

-   The name: "vip" parameter tells the module to create a virtual server named vip

-   The destination" parameter tells the module which IP address to assign for the virtual server

-   The port paramter tells the module which Port the virtual server will be listening on

-   The enabled_vlans parameter tells the module which all vlans the virtual server is enbaled for

-   The all_profiles paramter tells the module which all profiles are assigned to the virtuals server

-   The pool parameter tells the module which pool is assigned to the virtual server

-   The snat paramter tells the module what the Source network address address should be. In this module we are assigning it to be Automap which means the source address on the request that goes to the backend server will be the self-ip address of the BIG-IP

-   The validate_certs: "no" parameter tells the module to not validate SSL certificates. This is just used for demonstration purposes since this is a lab.

Run the playbook - exit back into the command line of the control host and execute the following:
```
[student1@ansible ~]$ ansible-playbook bigip-virtual-server.yml

[student1@ansible]$ ansible-playbook bigip-virtual-server.yml

PLAY [BIG-IP SETUP]*************************************************************

TASK [ADD VIRTUAL SERVER] ******************************************************

changed: [f5]

PLAY RECAP *********************************************************************

f5 : ok=1    changed=1    unreachable=0    failed=0
```
Confirm that you see the new virtual server in F5:

![](https://lh6.googleusercontent.com/Gc60uP1hTk2T-M4oMyuvQ_idl3Y5zCmXt_jlc1BCoDvXFHCmHlI62kytLxBeyMfGkEzrj8Z2aozLkhVDCooskvIJbrGAcNrU4VehcImTltP4a0Vo_kKTiQ3cYi4t4to8i6QSxRGA)

Verify that the load balancing does really work:

Each RHEL web server actually already has apache running. The above exercises have successfully set up the load balancer for the pool of web servers. Open up the public IP of the F5 load balancer in your web browser:

This time use port 443 instead of 8443, e.g. [https://X.X.X.X:443/](https://x.x.x.x/)

Each time you refresh the host will change between node1 and node2. Here is animation of the host field changing: ![animation](https://lh4.googleusercontent.com/M6cMaFmTKMh6JjGQ1dR7LrSy3WNR1H1XHdqCYvSx8S9aa9KE9EBi_peizZxQbk76dYPi1AkXa8LaCs6d4Gf0qbxSaiYZ227WOIEgraaAXr9-KruVpmlDZgDFEFn70dnqPRWhUCwv)

Instead of using a browser window it is also possible to use the command line on the Ansible control node. Use the curl command on the ansible_host to access public IP or private IP address of F5 load balancer in combination with the --insecure and --silent command line arguments. Since the entire website is loaded on the command line it is recommended to | grep for the student number assigned to the respective workbench. (e.g. student5 would | grep student5)
```
[studentX@ansible ~]$ curl https://172.16.26.136:443 --insecure --silent | grep studentX

    <p>F5TEST-studentX-node1</p>

[studentX@ansible ~]$ curl https://172.16.26.136:443 --insecure --silent | grep studentX

    <p>F5TEST-studentX-node2</p>

[studentX@ansible ~]$ curl https://172.16.26.136:443 --insecure --silent | grep studentX

    <p>F5TEST-studentX-node1</p>
```
