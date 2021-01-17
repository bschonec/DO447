# DO447
Notes I've taken in order to study for Red Hat Ansible Best Practices

I have not taken the EX447 exam.  I am bound by a non discloure agreement and cannot and will not answer any questions about any Red Hat Exam.  These are notes that I've taken while studying for the exam.

# Chapter 1

use "ansible_host" parameter if you want to 'rename' a host in an inventory file.

# Setting up git:

default push = simple

Variables must start with an alpha.  Underscores are the only "punctuation" character allowed.  Numbers are allowed but not at beginning.


# ansible-doc -l -t <plugin type>
# ansible-doc -l -t inventory
# ansible-doc -t inventory vmware_vm_inventory

# ansible-config dump


 :help modeline

search for 'et' (expand tab)

ansible-inventory -i <inventory_file> --list  # returns inventory in JSON format
ansible-inventory -i <inventory_file> --list --yaml # returns inventory in YAML format

# lab activity-name start
# lab activity-name finish

ls -l bad
^bad^good  (rerun last command replacing bad with good

lab development-practices start # Run every time you rebuild the lab


ansible-config dump

ansible -t inventory -l  # lists inventory plugins
ansible-inventory --yaml -i inventory --list --output inventory.yml  # convert INI inventory to YML



Roles defined in a play are run first, before tasks regardless of their order in the playbook.

Ansible runs handlers in the order they are given in the yaml file, not the order they were called.

Avoid using or disable callback plugins on Ansible Tower.

# Optimization:
disable fact gathering when possible
don't use loop for YUM module...use array of packages


- hosts: localhost
  remote_user: root
  roles:
    - this_role
	- that_role


ansible-galaxy list


"import" will load in the entire file and then treat that file like it's part of the parent file.  When running/syntax checking, import will make one big file.
"include" will load the file only at run time.  Errors in "include" files won't be found until they're instantiated at run time.

"force_handlers: true" will force the handler to execute even if the play has errors
- meta: flush_handlers  # run handlers now.


- hosts: all
  order: sorted/inventory/shuffle
  gather_facts: false
  tasks:
  
ansible-playbook --list-tasks --list-tags

with pipelining, require TTY for sudo must be disabled.


ansible -l -t callback

lookup plugins:
  {{ lookup('file', '/etc/hosts' ) }}   
  {{ query('file', '/etc/hosts' ) }}    # returns slightly different output as a list
  
tasks:
  
hostvars['hostname']['myfacts']



Job template wraps project, inventory, credentials

Projects are contained in tower:/var/lib/awx/projects directory

Credentials to teams (for users) must be done at the tower-cli level.  You cannot use the GUI to grant permissions to teams.


curl --user admin:redhat -X -k POST  <url of job template>

WHen using a playbook and URI module, ensure "return_content: true" is set so that you can use/check the returned data.




YAML-based inventory structure:

web_servers:
  hosts:
    webserver_1:
	  ansible_host: server1.lab.example.com
	webserver_2:
	  ansible_host: 1.2.3.4

lb_servers:
  hosts:
    lb_1:
	  ansible_host: lb1.lab.example.com
	lb_2:
	  ansible_host: lb2.lab.example.com

# Chapter 4:  Transforming Data with Filters and Plugins	  
ansible-doc -t garbage -l  # shows all the types you can list
ansible-doc -t lookup -l    # shows all the lookup plugins
ansible-doc -t lookup list  # dos for list lookup

## Creating users with password lookup

user:
  password: "{{ secret_password | password_hash('sha512') }}"


find module.  Use 'files' return variable
"{{ found_files |  map(attribute='path') | list }}" # from the previous 'find' module call
"{{ map('relpath', '/etc/') | list }}"

## Templating External Data using Lookups

Typically, dictionaries do NOT have a - as the first character.  Transform a dict to a list with "dict2items" filter.

Lists/netsted lists do start with a dash.  Use "{{ list_var | subelements("subelement") }}" to get the item.0.name and item.1 (the subelement itself) to use.



Ensure that you have the required python libraries for the filter you intend to use!
ansible-doc -t lookup dig   # this will have a REQUIRED section where it shows what libraries need to be installed
# Read contents of a file into a variable
var: "{{ lookup('file', '/path/to/file') }}"

vars:
  characters_dict:
    Douglas: human
	C3PO: robot
	Marvin: humah
	
| dict2items tranforms a dictionary key/value pair into:
characters_dict:
  - key: Douglas
    value:  human
  - key: C3PO
    value: robot
  - key: Marvin
    value: human

Conversely, items2dict transforms the -key/value: items into a dictionary.

| flatten   takes out nested lists and flattens all values into a single list

file lookup plugin reads the contents of a file on disk into a variable

"{{ secret_password | password_hash('sha512') }}"
"{{ value | b64encode }}"
"{{ my_string | quote }}"    # sanitizes string to prevent code injection
"{{ 'marvin' | upper }}"


"{{ lookup('file', '/path/to/yaml') | from_yaml }}"



{{ my_value | default('default_value', true) }}
{{ my_value | default(omit) }}

# These two are identical
loop: "{{ users_dict | dict2items }}"
with_dict: "{{ users }}"

# Replaces with_subelements syntax:
loop: "{{ lookup('subelements', users, 'groups') }}"
loop: "{{ users | subelements('groups') }}":web_servers

loop: "{{ public_keys_list | map(attribute='public_keys') | flatten }}"

ansible -m setup localhost setup -a 'gather_subset=!all,network' | less


# python3-netaddr package must be installed in order to use the network filters
# python3-dns package is needed for the 'dig' lookup


Network address filters
netmask = "192.168.0.0/255.255.255.0"

{{ '10.2.1.3' | ipaddr }}   # returns the IP address and true equivalent

{{ ipaddr  | ipaddr('prefix') }} # "24"
{{ ipaddr  | ipaddr('net') }} # "192.168.0.1/24"
  
# Chapter 5: Coordinating Rolling Updates
## Delegating tasks

## Managing rolling updates
---
- name: rolling updates
  serial: 2
  max_fail_percentage: 50%
  
  Ansible halts the playbook when ALL hosts in the "serial" directive fail.  If you have "serial: 2"
  and both hosts fail then all other processing stops.
  
  max_fail_percentage works against the "serial" directive.
  
  
# Chapter 6:  Installing Ansible Tower 3.5
Job template requires at minimum, inventory, project (with a at least one playbook)
  optional:  credentials (ssh key, sudo password, etc)
  
  Job templates execute jobs.
  
  
  
# Chapter 7 Managing Access with Users and Teams
  Organization:  Logical collection of teams, projects and inventories.  All users must be in an organization.
  
  You need 'tower-cli' to manage users and teams effectively.  Run 'tower-cli config' to configure the tool.

  tower-cli config verify_ssl false
  tower-cli config password redhat
  tower-cli config host tower.example.com
  
  tower-cli role grant --user=daniel --target-team=Developers --type=admin
  tower-cli user create --username=Brian --email=brian@lab.example.com --password=password123 .....


job templates
users
credentials
inventories

roles-based access control

use separate organizations to give users access to tasks they need to perform eg: Provisioning org, Networking org, Security org, etc etc.


TEAMS:  Users can have roles in multiple organizations.  


You have to use the tower-cli utility to add any other roles other than "MEMBER" to users.

# Chapter 8 Managin inventories and credentials
When an inventory is first created, it is only accessible by users who have the Admin, Inventory
Admin, or Auditor roles for the organization to which the inventory belongs. All other access
must be specifically configured.

SCM Update Options:
  CLEAN: Local modifications purged
  DELETE ON UPDATE: local repo directory is deleted and recloned
  UPDATE REVISION ON LAUNCH: "git pull"

## Creating machine credentials for access to inventory hosts



# Chapter 9:  Managing Projects and Launching Ansible Jobs

/var/lib/awx/projects is where PROJECTs clone git code into.

# Chapter 10:  Constructing Advanced Job Workflows

# Chapter 11:  Communicating with APIs Using Ansible
API:  curl -X GET https://tower.lab.example.om/api/v2/activity_stream -key
  
  ansible-doc uri  # search for 'api' and you'll have a near-perfect example.
  
  use https://tower.lab.example.com/api/v2/job_templates
  
  curl -X GET https://tower.lab.example.com/api/v2 -k -s | 
  curl -X get --user admin:redhat <blah blah>
  


# Chapter 12:  Managing Advanced Inventories

awx = Ansible Worx
awx-manage inventory_import <file.[json,ini,yml]> --source=../path/to/inventory --inventory-name="Tower Inventory Name"     # -- manages dynamic inventories

ansible-lint


awx-manage


#Import existing *file* inventory into *tower inventory*.  The tower inventory obect must already exist.
awx-manage inventory_import --source=inventory/ --inventory-name="My Tower Inventory"


PUT is used to modify existing or create NEW properties
POST is used to run jobs,templates, or to modify EXISTING stuff.


  awx-manage inventory_import --source=/path/to/inventory/ --inventory-name="My Tower Inventory" --overwrite
  
  Inventories can be synced from a GIT repository.
  
  Dynamic inventory scripts must accept '--list' and '--host' as command line arguments.
  
  Smart inventories use fact caching to apply the smart host filter.  Fact caching must be run via a simple job template or playbook that runs
  gather_facts: true (and really nothing else) periodically or from a schedule.
  
  Smart host filter must have FACT:VALUE  (ansible_facts.ansible_distribution:RedHat)

GIT based inventories must have their PROJECT and then the Inventory/Sources objects *refreshed* before the inventory in the HOSTS tab is refreshed.

 
Dynamic inventory scripts must accept --list and --host options and return the appropriate inventory.

 
# Chapter 14


Ensure to select the *Ansible Tower* section of the documentation and the Installation section for this chapter.

  supervisorctl status
  awx-manage changepassword admin 
  awx-manage createsuperuser  
  firewall-cmd --list-ports
  
  ansible-tower-service status
  
  backup:  ./setup.sh -b
  
  # SSL configuration:  /etc/nginx/nginx.conf 
  ssl_certificate /etc/tower/tower.cert
  ssl_certificate_key /etc/tower/tower.key
  semange fcontect -a -t cert_t "/etc/tower(/.*)?"
  restorecon -Rv /etc/tower/
  
  ipa-getcert request | grep PEM # both switches you need
  ipa-getcert request -f /etc/tower/tower.cert -k /etc/tower/tower.key
  ansible-service-restart
  
 
 awx-manage 


# Comprehensive review

the dynamic inventory python scripts don't work out of the box.  You must install python3-ldap and run the "lab advinventory-dynamic start" script to get IDM installed.

