# custom-ifnames-rules
As it is currently written, it will only add ens[100] thru ens[999].  This was
created in an ESXi environment, thus all of the NICs assigned fall in the above
naming convention.  The 'when' condition should be able to be adjust to meet 
your requirements.

Syntax:

ansible-playbook -i <INVENTORY_FILE> generate_udev_custom_ifnames.yml -b


-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-
---
- hosts: all

  vars:
    mynics: "{{ ansible_facts['interfaces'] }}"

  tasks:
  - name: Display NICs on server
    debug:
      msg: "{{ item }}"
    with_items: "{{ mynics }}"

  - name: Create 70-custom-ifnames.rules file
    lineinfile:
      create: true
      path: /etc/udev/rules.d/70-custom-ifnames.rules
      line: SUBSYSTEM=="net",ACTION=="add",ATTR{address}=="{{ ansible_facts[item]['macaddress']|default(None) }}",ATTR{type}=="1",NAME="{{ item }}"
#      msg: "{{ ansible_facts[item]['macaddress']|default(None) }}"
    with_items:
    - "{{ mynics }}"
    when: item >= 'ens100' and item <= 'ens999'
