# custom-ifnames-rules
Script: generate_udev_custom_ifnames.yml

As it is currently written, it will only add ens[100] thru ens[999].  This was
created in an ESXi environment, thus all of the NICs assigned fall in the above
naming convention.  The 'when' condition should be able to be adjusted to meet 
your requirements.

Syntax:

ansible-playbook -i <INVENTORY_FILE> generate_udev_custom_ifnames.yml -b


-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-
Script: gen_udev_custom_ifnames_v2.yml

This is an updated version.  One of the issues was that the ansbile doesn't capture the real MAC address of a NIC when the NIC is under teamd control.  This ifnormation can
be retrived with the 'ip -P <nic>'

Syntax:

ansible-playbook -i <INVENTORY_FILE> gen_udev_custom_ifnames_v2.yml -b


