# ansible training resources

This repository contains resources that were developed in the course of following 
the excellent [Ansible Essentials with Hands-on Labs](https://www.udemy.com/course/ansible-essentials/)

Below are some notes. 

## Notes

In the ansible hosts file it is possible to use a friendly name for a machine, a name which does not resolve through DNS but is tied to an IP address within the ansible hosts file
```
friendlyname ansible_host=192.168.60.5
```

In the ansible hosts file a host can be repeated in order to belong to multiple groups.

In the host files it is possible to instruct ansible which port to use for SSH to a particular host:
```
dbserver1 ansible_port=2222 ansible_user=dbadmin
```

The above is an example of behavioural inventory parameters.

Definition does not need to  be host specific:
```
[all:vars]
ansible_port=2222
```

```
ansible-config dump [--only-changed]

```

edit `/etc/ansible/ansible-config` to set inventory (hosts) file location

Or can have ansible.config in current working directory on the ansible controller

Running ad-hoc commands using ansible
```
ansible [-i inventoryfile] target_hosts -m module -a argos
```
eg
```
ansible all -m ping
```
```
ansible web2 -m command -a "uptime"
```
to push /etc/hosts out to other machines:
```
ansible web -m copy -a "src=/etc/hosts dest=/tmp/"
```
```
ansible web -m yum -a "name=ntp state=present" -b -K
```
("-b" means "use sudo")

ansible understands state. If there's nothing that needs doing, it won't do anything.

```
ansible all -m shell -a "cat /etc/redhat-release"

```

gather facts:
```
ansible web1 -m setup
```
```
ansible all -m setup -a "filter=ansible_memtotal_mb"
```

### a bit about YAML

string values don't usually need quotes but will if they contain stuff that a YAML parser could get fooled by, eg
```
path: "C:/a/b/c/"
```

#### Lists
```
fruit:
  - Apple
  - Orange
```

#### Dictionary
```
marta:
  name: Marta
  job: Developer
  skill: Elite
```

#### All together:
```
devs:
 - Marta:
   name: Marta
   skills:
     - python
     - lisp
  - Fred:
```

Doc start can be indicated with `---`
Doc end can be indicated with `...`

### Back to Ansible...
A Playbook is a list of Plays

A Play is a group of Tasks to be performed

A Play is a dictionary:

"hosts" key defines target machines

"tasks" key defines tasks

other keys might define SSH user and/or variables
eg
```
---
#comment
 - hosts: frontend
   remote_user: root
   tasks:
     - name:
       yum:
         name: httpd
         state: present
```

A task defines what state the host should be in.
So a task does not necessarily define work which must be done.

A key principle is idempotency.

Low-level modules (e.g 'command' or 'shell') are not idempotent.

Options when running a Playbook

```
ansible-playbook myplaybook.yml --syntax-check
```

or, for a dry run:
```
ansible-playbook myplaybook.yml -C
```

`-K` option means "prompt for sudo password". Without this, in some scenarios, there is a risk of a playbook getting stuck at the privilege escalation prompt.

Don't have to stick with running stuff as the logged-in user (albeit perhaps with a sudo thrown in). Can specify a different user account, eg. `become_user: root`

Use `remote_user` to specify the SSH user.

To see more of what's going on, run the playbook with `-v` or even `-vvv`

Use `{{ ansible_facts }}` in your playbook tasks to get properties relating to the host. (These facts are obtained by ansible thanks to an implicit "gathering facts" task which executes before other tasks.)

From one host it is possible to use `{{ hostvars }}` to access the `{{ ansible_facts }}` for a different host e.g. `{{ hostvars.web2.ansible_facts }}`

package module vs yum: package module is an abstraction so that you don't have to worry whether, for a given host, you should be using 'yum' or 'apt'.

Can pull variables in from a file:
```
- hosts: all
  include_vars: blah.yml
```

Can prompt for variable
```
- hosts: web
  vars-prompt:
    - name: "version"
      prompt: "which version?"
```

Can also pass in variable definitions from the command-line, e.g. to override.

set_fact variables (host-specific) endure/are visible across multiple Plays (but not multiple executions of the Playbook).

Standardised directory structure for files which define variables at the host level, or variables which apply to a group of hosts.

Ansible tracks the incidence of tasks having made changes (idempotency again)

Define a Block like this:
```
block
rescue
always
```

`import` is static and parsing is up-front

`include` is dynamic

see, for example, `include_tasks` and `import_tasks`

evaluation of "when" is different in each case too.

Roles have a predefined directory structure.

Roles are defined separately from the Playbook and the Inventory.

```
mkdir roles
cd roles
ansible-galaxy init myrole
```

refactoring a Playbook into roles is easy.

Role files go in the files subdirectory for the role. But the role tasks can refer to the file simply using the file name without any path.

There are 2 places where a role can define variables:

```
myrole
  - vars
  - defaults
```

variables defined under 'defaults' have the lowest precedence and are the most easily overridden. They can be overridden by inventory variables, playbook variables or the cmd line.

variables defined under 'vars' are less easily overridden. These are more like constants. They can only be overridden using the command line.

It is recommended to prefix a variable with the role-name, like a namespace.

Go to galaxy.ansible.com to browse roles that others have created.

Use the dependencies section within the `meta/main.yml` file to declare dependencies on other roles. These roles will be called first, before your role gets called.



When pulling in another role, it is possible to inject/override variables:
```
- hosts: web
  become: true
  roles:
    - role: web
      vars:
        haproxy_backend_weight: 100
```


Example of conditionally including a role:
```
- host: someservers
  tasks:
    - include_role:
        name: myrole
      when: ansible_facts["os_family"] == "RedHat"
```

