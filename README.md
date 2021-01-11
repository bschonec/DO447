# DO447
Notes I've taken in order to study for Red Hat Ansible Best Practices


I have not taken the EX447 exam.  I am bound by a non discloure agreement and cannot and will not answer any questions about any Red Hat Exam.  These are notes that I've taken while studying for the exam.


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

{{ ipaddr  | ipaddr }}  # Reverse record
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
  
  Chapter 7: Creating and Managing Ansible Tower Users
  Organization:  Logical collection of teams, projects and inventories.  All users must be in an organization.
  
  You need 'tower-cli' to manage users and teams effectively.  Run 'tower-cli config' to configure the tool.
  
  tower-cli role grant --user=daniel --target-team=Developers --type=admin
  tower-cli user create --username=Brian --email=brian@lab.example.com --password=password123 .....
  
  Chapter 9:  Managing Projects and Launching Ansible Jobs
  Chapter 10:  Constructing Advanced Job Workflows
  Chapter 11:  Communicating with APIs Using Ansible
  
  ansible-doc uri  # search for 'api' and you'll have a near-perfect example.
  
  use https://tower.lab.example.com/api/v2/job_templates
  
  curl -X GET https://tower.lab.example.com/api/v2 -k -s | 
  curl -X get --user admin:redhat <blah blah>
  
  # Chapter 12:  Managing advanced inventories
  awx-manage inventory_import --source=inventory/ --inventory-name="My Tower Inventory" --overwrite
  
  Inventories can be synced from a GIT repository.
  
  Dynamic inventory scripts must accept '--list' and '--host' as command line arguments.
  
  Smart inventories use fact caching to apply the smart host filter.  Fact caching must be run via a simple job template or playbook that runs
  gather_facts: true (and really nothing else) periodically or from a schedule.
  
  Smart host filter must have FACT:VALUE  (ansible_facts.ansible_distribution:RedHat)
  
  # Chapter 14
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
