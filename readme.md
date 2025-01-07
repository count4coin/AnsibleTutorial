## how to validate whether a task is successful or not
-> The task would say 'changed' or 'ok' in the task metadata and should not have error keys.

## Lets say there is a complex scenario to deploy or automate something throughout the flow an error happens or a task fails how will you roll back earlier changes?**
-> Use block and rescue with always directive if required.

```
tasks:
   - name: Always do X
     block:
       - name: Print a message
         ansible.builtin.debug:
           msg: 'I execute normally'

       - name: Force a failure
         ansible.builtin.command: /bin/false

       - name: Never print this
         ansible.builtin.debug:
           msg: 'I never execute :-('
     always:
       - name: Always do this
         ansible.builtin.debug:
           msg: "This always executes, :-)"
```



## how to run a task in parallel.

*or, how to run a task in parallel on more than 1 hosts or how to run the same task parallely.*

By Default when you specify the hosts in an inventory ansible will run the play or the playbook on all the hosts in parallel.

-> Async and poll to run a task asynchronously
```
- name: Simulate long running op, allow to run for 45 sec, fire and forget
    ansible.builtin.command: /bin/sleep 15
    async: 45
    poll: 0
```
-> use serial directive to specify the batch size, and ansible will process the tasks on the hosts in batches of the set amount.

```
---
- name: test play
  hosts: webservers
  serial: 3
  gather_facts: False

  tasks:
    - name: first task
      command: hostname
    - name: second task
      command: hostname
```

## how does a task gets executed when using loops (in parallel or consecutively).
-> it gets executed consecutively

## Can you make a loop run parallelly?
yes you can by setting async and poll async is the directive for setting a timeout, poll is the directive wich instructs ansible to poll or check the status of a asynchronous task periodically.

```
- name: My long runing task
   some_module_name:
     ip: "{{item.fabric}}"
     username: "{{user}}"
     password: "{{password}}"
     secret: "{{secret}}"
   loop: "{{zoning_list}}"
   register: _alias_vc_0
   async: 60
   poll: 5
```


## how to make api calls in ansible, how to pass in a request body, how to handle different types of authentications.

-> API calls can be made using the `uri` directive.
-> Request body can be passed as json in key/value pair under the body element of uri.
-> API/Bearer token based auth, Oauth, credential based.

```
- name: fetch filtered incident data
  uri:
    url: "{{ servicenow_instance }}/api/now/table/incident?sysparm_query={{query}}&sysparm_fields={{servicenow.fields}}"
    method: GET
    headers:
      Authorization: "Basic {{ (servicenow.username + ':' + servicenow.password) | b64encode}}"
      Accept: application/json
    status_code: 200
    retries: 5
    return_content: yes
  register: servicenow_resp
```


## how to add retries to a task.


-> by using the retries directive with delay and an exit condition.

```
- name: Retry a task until a certain condition is met
  ansible.builtin.shell: /usr/bin/foo
  register: result
  until: result.stdout.find("all systems go") != -1 # this is the exit condition
  retries: 5 # max number of retries
  delay: 10 # the delay before each retry
```


## when to use jinja templates.
-> when you want to have more control over what you do with variables and conditional statements.

## filters, trim and other functions that are available in ansible.
-> filter: utilites that are present in Ansible like trim, reg_exp, lookup etc.
-> trim: removes whitespaces from either side of a string
-> reg_exp: lets you use regular expressions on strings.
-> lookup: lets you find or apply ansible environment variables.

## what are inventory variables.
-> These variables define the details and parameters for hosts, groups, and other settings within your inventory.

- Host Variables:
Variables specific to a single host.

```
ansible_host: 192.168.1.10
ansible_user: admin
ansible_password: secret
```

- Group Variables:
Variables shared among all hosts in a group.

```
web_servers:
  hosts:
    server1:
    server2:
  vars:
    ansible_user: deploy
    app_version: 2.1
```

- All Variables (all group):
Variables applicable to all hosts in the inventory.

```
all:
  vars:
    ansible_python_interpreter: /usr/bin/python3
```

### For your understanding

```
web_servers:
  hosts:
    server1:
    server2:
    server3:
    server4:
  vars:
    ansible_user: deploy
    app_version: 2.1

web_servers:
 
  hosts:
   
    server1:
      ansible_user: deploy
      app_version: 2.1
   
    server2:
      ansible_user: deploy
      app_version: 2.1
   
    server3:
      ansible_user: deploy
      app_version: 2.1

    server4:
      ansible_user: deploy
      app_version: 2.1
```

Example:

```
all:
  vars:
    ansible_user: admin
    ansible_ssh_private_key_file: /keys/id_rsa
  children:
    web_servers:
      hosts:
        web1:
          http_port: 80
        web2:
          http_port: 8080
    db_servers:
      hosts:
        db1:
          ansible_host: 192.168.10.1
          db_port: 5432
```

## Dynamic inventories

-> Use Custom Scripts or Dynamic Inventory Plugins to pull inventory and variables dynamically.
Example: AWS EC2 dynamic inventory plugin fetches ansible_host for each instance automatically.

## Variables Precedence Order Cheat sheet

### Precedence of Variables
When variables overlap, Ansible resolves them based on the following order (highest to lowest precedence):

0. Command line variables - injected at the time of playbook execution as inputs
1. Playbook variables - variables that you set inside your playbook
2. Host variables - variables inside your inventory file that are specific to hosts.
3. Group variables - variables in your inventory that are applied to group of hosts.
4. Inventory variables defined for all
5. Default values in roles or playbooks - In a role you can use default values to setup context for your variables, to initialize them to default values. you can later overrwrite them.


### Special Variables
- `ansible_host`: IP address or hostname for connecting to the host.
- `ansible_user`: Username for SSH.
- `ansible_ssh_private_key_file`: Path to the SSH private key.
- `ansible_python_interpreter`: Path to the Python interpreter.

