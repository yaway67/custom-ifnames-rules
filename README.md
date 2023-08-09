# custom-ifnames-rules
As it is currently written, it will only add ens[100] thru ens[999].  This was
created in an ESXi environment, thus all of the NICs assigned fall in the above
naming convention.  The 'when' condition should be able to be adjust to meet 
your requirements.

Syntax:

ansible-playbook -i <INVENTORY_FILE> generate_udev_custom_ifnames.yml -b


-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-
