# custom-ifnames-rules
As it is currently written, it will only add ens[100] thru ens[999].  This was
created in an ESXi environment, thus all of the NICs assigned fall in the above
naming convention.  The 'when' condition should be able to be adjusted to meet 
your requirements.

Syntax:

ansible-playbook -i <INVENTORY_FILE> generate_udev_custom_ifnames.yml -b


-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-
<<<<<<< HEAD
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


-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-
This is an updated version.  One of the issues was that the ansbile doesn't capture the real MAC address of a NIC when the NIC is under teamd control.  This ifnormation can
be retrived with the 'ip -P <nic>'
Syntax:
 ansible-playbook -i <INVENTORY_FILE> gen_udev_custom_ifnames_v2.yml


#(@) gen_udev_v2.yml
# Author: Scott McKay (mckays7)
# Purpose: This script was written because Xilinx Solarflare V8 package(s), both Open Source and Enterprise, has decided to overwrite RHEL 8
# default naming convention.  For exmaple, the default name of 'ens1f0np0' wold change to 'ens1f0' after the Xilinx/AMD package is installed.
# Doing so, it would break the network, requiring to boot into single user mode inorder to rename the devices.  This generates a 70-custom-ifnames.rules
# that forces the naming convention to not change, provided it's listed in the file.  Ansible does pull the MAC address for the NICs.  However,
# if the active device is a teaming device, the MAC address is shared for 1 of the ports.  Ansible picks up the shared MAC address.  This script
# uses "ethtool -P" which is able to resolve the 'permaddr'
# If the NICs have already been renamed, as in the Solarflare package was already installed and one has already renamed the devices in the
# /etc/sysconfig/network-scripts, this script will not work since it will not be able to find any 'ensXfXnpX'.  To fix, you would need to uninstall
# the Solarflare package, undo the changes to the /etc/sysconfig/network-scripts, reboot the server successfully and have the networking running,
# Run this ansible script, install the Solarflare package(s) and reboot.
# If you check the /etc/udev/rules.d file, the MAC address for same card, for example ens1*, ens2*, etc..., should have a different MAC address.
#
# Example Syntax:
# SUBSYSTEM=="net",ACTION=="add",ATTR{address}=="00:11:22:33:44:55",ATTR{type}=="1",NAME="ens1f0np0"
---
- hosts: localhost
  become: false

  # I'm a bit anal and want any file generated with a timestamp to all have the same timestamp.
  pre_tasks:
    - set_fact: 
        mydate: "{{ ansible_date_time.year }}{{ ansible_date_time.month }}{{ ansible_date_time.day }}_{{ ansible_date_time.hour }}{{ ansible_date_time.minute }}{{ ansible_date_time.second }}"

- hosts: all
  become: true
  vars:
    mynics: "{{ ansible_facts['interfaces'] }}"
    mydate: "{{ hostvars['localhost']['mydate'] }}"
    mytmpfile: "/tmp/{{ ansible_hostname }}_{{ mydate }}"
    my70udevfile: /etc/udev/rules.d/70-custom-ifnames.rules

  tasks:
    - name: Run command
      ansible.builtin.shell:
        cmd: echo "SUBSYSTEM==\"net\",ACTION==\"add\",ATTR{address}==\"$(export PATH=/usr/sbin:/usr/bin;ethtool -P {{ item }} | awk '{ print $3 }')\",ATTR{type}==\"1\",NAME=\"{{ item }}\"" >> {{ mytmpfile }}
      with_items:
      - "{{ mynics }}"
#      when: item is match('ens[0-9][0-9][0-9]') or item is match('ens[1-9]f[0-9]np[0-9]')  # For testing 
      when: item is match('ens[1-9]f[0-9]np[0-9]')  # For Solarflare cards.  The 'npX' is on rhel8 and possible RHEL 9.
      notify:
      - place 70-custom-ifnames.rules
      - retrieve a copy of the file
    
  handlers:
    - name: place 70-custom-ifnames.rules
      ansible.builtin.shell:
        cmd: cp {{ mytmpfile }} {{ my70udevfile }}

    - name: retrieve a copy of the file
      ansible.builtin.fetch:
        src: "{{ mytmpfile }}"
        dest: downloaded/
        flat: true       
=======
>>>>>>> f304bd0277ad3930d185e272968406e93e22dc2a
