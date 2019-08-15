IBM Spectrum Scale (GPFS) GUI Ansible Role  (Not finished)
======================================

 
Highly-customizable Ansible role for installing and configuring IBM Spectrum Scale (GPFS) GUI nodes. 

Particularly looking for [feedback](https://github.com/acch/ansible-scale-gui/issues) and future [requirements](https://github.com/acch/ansible-scale-gui/issues/new)!


Features
--------

- **Install GUI and zimon packages**
- **Configure Performance Monitoring and collectors**
- **Install and configure CallHome**
- **Dual/HA GUI Nodes**
- **Create Local Users for GUI**
- **Change Default Password Policy for GUI Users**
- **Configure Active Directory and Map roles to AD Groups**
- **Configure E-Mail Notifications**
- **Configure E-Mail Receptions**
- **Configure SNMP Notification**


The following installation methods are available:
- Install from (existing) YUM repository


Future plans:
- Upgrade and Disable services.
- Hasicorp Integration for Consul and Vault. 




Example Playbook
----------------

The simplest possible playbook to install Spectrum Scale on a node with GUI:

```

---
- hosts: scale01.example.com
  vars:
    - scale_version: 5.0.3.2
    - scale_install_localpkg_path: /path/to/Spectrum_Scale_Standard-4.2.3.4-x86_64-Linux-install
  roles:
    - acch.spectrum_scale
    - acch.ansible-scale-gui
```

This will install all required packages and create a single-node Spectrum Scale cluster.

In reality you'll most probably want to install Spectrum Scale on a number of nodes, and you'll also want to consider the node roles in order to achieve high-availability. The cluster will be configured with all hosts in the current play:

```
# host_vars/scale01:
---
scale_storage:
  - filesystem: gpfs01
    blockSize: 4M
    defaultMetadataReplicas: 2
    defaultDataReplicas: 2
    numNodes: 16
    automaticMountOption: true
    defaultMountPoint: /mnt/gpfs01
    disks:
      - device: /dev/sdb
        nsd: nsd_1
        servers: scale01
        failureGroup: 10
        usage: metadataOnly
        pool: system
      - device: /dev/sdc
        nsd: nsd_2
        servers: scale01
        failureGroup: 10
        usage: dataOnly
        pool: data
    
    scale_nodeclass:
      - class1
    scale_config:
      - nodeclass: class1
        params:
          - pagepool: 2G
          - autoload: yes
          - ignorePrefetchLunCount: yes
scale_gui_collector: true
```
```
# host_vars/scale02:
---
scale_storage:
  - filesystem: gpfs01
    disks:
      - device: /dev/sdb
        nsd: nsd_3
        servers: scale02
        failureGroup: 20
        usage: metadataOnly
        pool: system
      - device: /dev/sdc
        nsd: nsd_4
        servers: scale02
        failureGroup: 20
        usage: dataOnly
        pool: data
scale_gui_collector: true  ##Dual/HA GUI
```


All the variable below can be added to *host_vars/host:*  - the node you want to be GUI node. 
For the Second GUI node, is only needed to have `scale_gui_collector: true`



#####Active Directory information for Managing GUI users in an external AD or LDAP server
```
scale_gui_ldap_integration: true
scale_gui_ldap_name: 'myad'  ##Alias for your LDAP server
scale_gui_ldap_host: 'myad.mydomain.local' 
scale_gui_ldap_bindDn: 'CN=Administrator,CN=Users,DC=mydomain,DC=local'
scale_gui_ldap_bindPassword: 'password'
scale_gui_ldap_baseDn: 'CN=Users,DC=mydomain,DC=local'
scale_gui_ldap_port: '389'  #only if you need to change default
scale_gui_ldap_type: 'Microsoft Active Directory' #default is MS AD
scale_gui_ldap_keystore: /tmp/ad.jks #local on GUI Node
```

#####LDAP Mappings to Roles: 
The AD Groups needs to be create in the LDAP. don't need to be before deployment.
```
scale_gui_groups_securityadmin: 'scale-administrator'
scale_gui_groups_storageadmin: 'scale-storage-administrator'
scale_gui_groups_snapadmin: 'scale-snapshot-administrator'
scale_gui_groups_data_access: 'scale-data-access'
scale_gui_groups_monitor: 'scale-monitor'
scale_gui_groups_protocoladmin: 'scale-protocoladmin'
```

##### Spectrum Scale GUI Admin.
This is to be able to access the Spectrum Scale GUI. 
Change the role for the user do the Permission that is sufficient. 
```
scale_gui_admin_user: "admin"
scale_gui_admin_password: "Admin@GUI"
scale_gui_admin_role: "SecurityAdmin,SystemAdmin"
```

##### Spectrum Scale GUI user. 
Ekstra User can be created. example Monitor or RestAPI.
Change the role for the user do the Permission that is sufficient. 
```
scale_gui_user_username: 'SEC'
scale_gui_user_password: 'Storage@Scale1'  #password policy
scale_gui_user_role: 'SystemAdmin'
```

#### Change Default GUI User Password Policy
Add and Change what you need and rest wil use default.
If you only want to change max age of password. 
Add  `scale_gui_password_policy_change: true `scale_gui_password_policy_maxAge: '90'`
```
scale_gui_password_policy_change: true
scale_gui_password_policy_minLength: '6'
scale_gui_password_policy_maxAge: '90'
scale_gui_password_policy_minAge: '0'
scale_gui_password_policy_remember: '3'  #Remember number Of Old Passwords
scale_gui_password_policy_minUpperChars: '0'  #minimumUpperChars
scale_gui_password_policy_minLowerChars: '0'
scale_gui_password_policy_minSpecialChars: '0'
scale_gui_password_policy_minDigits: '0'
scale_gui_password_policy_maxRepeat: '0'
scale_gui_password_policy_minDiff: '1'
scale_gui_password_policy_rejectOrAllowUserName: '--rejectUserName'  ## --allowUserName
```


##### Default CallHome Information.
```
scale_gui_callhome: true
scale_gui_callhome_customer_name: 'ACME'
scale_gui_callhome_customer_id: 'ACMECUSTOMERID'
scale_gui_callhome_country_code: 'NO'
scale_gui_callhome_email: 'acme@acme.com'
```

## Callhome with Proxy
```
- scale_gui_callhome_proxy: true                        ##Mandatory
- scale_gui_callhome_proxy_host: 'ACMEPROXYHOST'        ##Mandatory
- scale_gui_callhome_proxy_port: '8080'                 ##Mandatory
- scale_gui_callhome_proxy_username: #'ACMEPROXYUSER'
- scale_gui_callhome_proxy_password: #'ACMEPROXYPASS'

scale_gui_callhome_proxy_with_auth: '--with-proxy-auth'
```

#### Parameters for configure E-Mail notification
```
scale_gui_email_name: 'SMTP_1' #Mandatory
scale_gui_email_ip_adress: '10.33.3.13'  ## Mandatory
scale_gui_email_ip_port: '25'            ## Mandatory
scale_gui_email_replay_email_address: "scale-server-test@no.ibm.com" ## Mandatory
scale_gui_email_contact_name: 'scale-contact-person'  ## Mandatory
scale_gui_email_subject: "&cluster&message"   ## Variables:  &message &messageId &severity &dateAndTime &cluster&component
scale_gui_email_sender_login_id:
scale_gui_email_password:
scale_gui_email_headertext: 'testheadertext'
scale_gui_email_footertext:
```

##### Parameters for configure E-Mail recipients
```
- scale_gui_email_notification: true
- scale_gui_email_recipients_name: 'acme_email_recipient_name' 
- scale_gui_email_recipients_address: 'email_recipient_address@acme.com' 
- scale_gui_email_recipients_components: 'SCALEMGMT='
- scale_gui_email_recipients_components_security_level: 'WARNING' ##
- scale_gui_email_recipients_reports: '--quotaNotification'
- scale_gui_email_recipients_quotathreshold: '100.0'
```

#### Enable SNMP notifications in Spectrum Scale GUI

```
scale_gui_snmp_notification: true
scale_gui_snmp_server_ip_adress: '10.33.3.13'
scale_gui_snmp_server_ip_port: '162'
scale_gui_snmp_server_community: 'Public'
```


Limitations
-----------

This role can (currently) only be used to create new clusters. 


Troubleshooting
---------------

To get output from Ansible task in stdout and stderr for some tasks. add  `scale_gui_ansible_task_output: true` in your host var.

This role stores configuration files in `/var/tmp` on the first host in the play. These configuration files are kept to determine if definitions have changed since the previous run, and to decide if it's necessary to run certain Spectrum Scale commands (again). When experiencing problems one can simply delete these configuration files from `/var/tmp` in order to clear the cache &mdash; this will force re-application of all definitions upon the next run. As a downside, the next run may take longer than expected as it might re-run unnecessary Spectrum Scale commands. Doing so will automatically re-generate the cache.

Please use the [issue tracker](https://github.com/acch/ansible-scale/issues) to ask questions, report bugs and request features.

