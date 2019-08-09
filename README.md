IBM Spectrum Scale (GPFS) GUI Ansible Role  (Not finsished)
======================================

 
Highly-customizable Ansible role for installing and configuring IBM Spectrum Scale (GPFS) GUI nodes. 

Particularly looking for [feedback](https://github.com/acch/ansible-scale/issues) and future [requirements](https://github.com/acch/ansible-scale/issues/new)!


Features
--------

- Install GUI and zimon packages
- Configure Performance Monitoring and collectors.
- Install and configure CallHome.
- Dual GUI Nodes.
- Create Local Users for GUI
- Password Policy for GUI
- Configure Active Directory and Map roles to AD Groups.
- Configure E-MAIL
- Configure SNMP



The following installation methods are available:
- Install from (existing) YUM repository


Future plans:
- Upgrade and Disable services.
- Hasicorp Integration for Consul and Vault. 




Limitations
-----------

This role can (currently) be used to create new clusters. 


Troubleshooting
---------------

To get output from Ansible task in stdout and stderr for some tasks. add  `scale_gui_ansible_task_output: true` in your host var.


This role stores configuration files in `/var/tmp` on the first host in the play. These configuration files are kept to determine if definitions have changed since the previous run, and to decide if it's necessary to run certain Spectrum Scale commands (again). When experiencing problems one can simply delete these configuration files from `/var/tmp` in order to clear the cache &mdash; this will force re-application of all definitions upon the next run. As a downside, the next run may take longer than expected as it might re-run unnecessary Spectrum Scale commands. Doing so will automatically re-generate the cache.

Please use the [issue tracker](https://github.com/acch/ansible-scale/issues) to ask questions, report bugs and request features.

