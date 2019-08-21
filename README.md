IBM Spectrum Scale (GPFS) GUI Ansible Role  (Not finished)
======================================

 
Highly-customizable Ansible role for installing and configuring IBM Spectrum Scale (GPFS) GUI nodes. 

Particularly looking for [feedback](https://github.com/acch/ansible-scale-gui/issues) and future [requirements](https://github.com/acch/ansible-scale-gui/issues/new)!


Features
--------

- **Install GUI and zimon packages**
- **Configure Performance Monitoring and collectors**
- **Install and configure Call Home**
- **Dual/HA GUI Nodes**
- **Create Local Users for GUI**
- **Change Password Policy for GUI Users**
- **Configure Active Directory and Map roles to LDAP Groups**
- **Configure E-Mail Notifications**
- **Configure E-Mail Receptions**
- **Configure SNMP Notification**
- **Hasicorp integration for Consul and Vault.**


The following installation methods are available:
- **Install from (existing) YUM repository**


Future plans:
- **Upgrade and Disable services.**




Example Playbook
----------------

The simplest possible playbook to install Spectrum Scale on a node with GUI:

```

---
- hosts: scale01.example.com
  vars:
    - scale_version: 5.0.3.2
    - scale_install_localpkg_path: /path/to/Spectrum_Data-Managment-5.0.3.2-x86_64-Linux-install
  scale_gui_collector: true
  roles:
    - acch.spectrum_scale
    - acch.ansible-scale-gui
```

This will install all required packages and create a single-node Spectrum Scale cluster with GUI.

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


All the variable below can be added to **host_vars/host:**  on the node you want to be GUI node. 
For the Second GUI node, is only needs to have `scale_gui_collector: true`



##### Managing GUI users in an external AD or LDAP  Parameters

```
scale_gui_ldap_integration: true
scale_gui_ldap_name: 'myad'  ##Alias for your LDAP/AD server
scale_gui_ldap_host: 'myad.mydomain.local' 
scale_gui_ldap_bindDn: 'CN=Administrator,CN=Users,DC=mydomain,DC=local'
scale_gui_ldap_bindPassword: 'password'
scale_gui_ldap_baseDn: 'CN=Users,DC=mydomain,DC=local'
scale_gui_ldap_port: '389'  #only if you need to change default
scale_gui_ldap_type: 'Microsoft Active Directory' #default is MS AD
scale_gui_ldap_keystore: /tmp/ad.jks #local on GUI Node
```

##### Managing GUI users in an external AD or LDAP - LDAP/AD Mappings to Roles: 
- The LDAP/AD Groups needs to be create in the LDAP. (don't need to be before deployment.)

```
scale_gui_groups_securityadmin: 'scale-administrator'
scale_gui_groups_storageadmin: 'scale-storage-administrator'
scale_gui_groups_snapadmin: 'scale-snapshot-administrator'
scale_gui_groups_data_access: 'scale-data-access'
scale_gui_groups_monitor: 'scale-monitor'
scale_gui_groups_protocoladmin: 'scale-protocoladmin'
```

##### Spectrum Scale GUI Admin.
- This is to be able to access the Spectrum Scale GUI. Change the role for the user do the Permission that is sufficient. 

- Default password policy is length of 6 Character. 

**All of the Parameters is mandatory.**
```
scale_gui_admin_user: "admin"
scale_gui_admin_password: "Admin@GUI"
scale_gui_admin_role: "SecurityAdmin,SystemAdmin"
```

##### Spectrum Scale GUI user. 
- Extra Users can be created.
- Change the Role for the user do the Permission that is sufficient. 

**All of the Parameters is mandatory.**
```
scale_gui_user_username: 'SEC'
scale_gui_user_password: 'Storage@Scale1'
scale_gui_user_role: 'SystemAdmin'
```

##### GUI User Password Policy Parameters
Add and Change what you need and rest wil use default.
If you only want to change max age of password. 
Add  `scale_gui_password_policy_change: true` `scale_gui_password_policy_maxAge: '90'`
```
scale_gui_password_policy_change: true  # Change of Password Policy
scale_gui_password_policy_minLength: '6' # Minimum Lengt of Password
scale_gui_password_policy_maxAge: '90' # Max Age of Password
scale_gui_password_policy_minAge: '0' # Minium Age of Password
scale_gui_password_policy_remember: '3'  # Remember number of old passwords
scale_gui_password_policy_minUpperChars: '0'  # Minimum Upper Characters
scale_gui_password_policy_minLowerChars: '0'  # Minimum Lower Characters
scale_gui_password_policy_minSpecialChars: '0' # Minimum Spesical Characters
scale_gui_password_policy_minDigits: '0'  # Minimum number of Digits
scale_gui_password_policy_maxRepeat: '0' # Maximum number repeted characters #todo check
scale_gui_password_policy_minDiff: '1' # 
scale_gui_password_policy_rejectOrAllowUserName: '--rejectUserName'  ## --allowUserName
```


#### Call Home Information Parameters
IBM Spectrum Scale introduces two call home features. The first feature is to collect predefined data from each GPFS node on a regular basis and upload this data to IBM support center. 
This data can be used by the support or development personnel for problem analysis. 
The second feature is to upload a specific file to IBM support center. 

All of the Parameters is mandatory. 

```
scale_gui_callhome: true
scale_gui_callhome_customer_name: 'ACME'
scale_gui_callhome_customer_id: 'ACMECUSTOMERID1234'
scale_gui_callhome_country_code: 'NO'  ## country-code 2 alphabet ISO  
scale_gui_callhome_email: 'acme@acme.com'
```

- Customer-Name Business/company name   This name can consist of any  alphanumeric characters and these non-alphanumeric characters: `'-', '_', '.', ' ', ','           
- Customer-id: This can consist of any alphanumeric characters and  these non-alphanumeric characters: `'-', '_', '.' `
- Country-code:  2 alphabet ISO   (EN, JP, NO,)
- E-mail Customer: E-mail ID of the customer. All alphanumeric and non-alphanumeric characters  are supported.  


*By enabling call home, you agree to allow IBM and its subsidiaries to store and use your contact information and your support 
information anywhere they do business worldwide. 
For more information, please refer to the Program license agreement and  documentation.*

                              
#### Call home with Proxy Parameters
```
scale_gui_callhome_proxy: true                        ## Mandatory
scale_gui_callhome_proxy_host: 'ACMEPROXYHOST'        ## Mandatory
scale_gui_callhome_proxy_port: '8080'                 ## Mandatory

scale_gui_callhome_proxy_username: #'ACMEPROXYUSER'
scale_gui_callhome_proxy_password: #'ACMEPROXYPASS'
scale_gui_callhome_proxy_with_auth: '--with-proxy-auth'
```

#### E-Mail notifications Parameters.

```
scale_gui_email_notification: true
scale_gui_email_name: 'SMTP_1'           ## Mandatory
scale_gui_email_ip_adress: '10.33.3.13'  ## Mandatory
scale_gui_email_ip_port: '25'            ## Mandatory
scale_gui_email_replay_email_address: "scale-server-test@no.ibm.com" ## Mandatory
scale_gui_email_contact_name: 'scale-contact-person'  ## Mandatory
scale_gui_email_subject: "&cluster&message"   ## Variables:  &message &messageId &severity &dateAndTime &cluster&component
scale_gui_email_sender_login_id:
scale_gui_email_password:
scale_gui_email_headertext:
scale_gui_email_footertext: 
```

#### E-Mail Recipients Parameters
To add E-mail Recipients the scale_gui_email_notification need to be configured.
Add all of the parameters change them to your environment.

The value `scale_gui_email_recipients_components_security_level: ` Need to contain the Component and the Warning Level 


- Components_security_level
```
AFM=WARNING,AUTH=WARNING,BLOCK=WARNING,CESNETWORK=WARNING,CLOUDGATEWAY=WARNING,CLUSTERSTATE=WARNING,DISK=WARNING,FILEAUDITLOG=WARNING,FILESYSTEM=WARNING,GPFS=WARNING,GUI=WARNING,HADOOPCONNECTOR=WARNING,KEYSTONE=WARNING,MSGQUEUE=WARNING,NETWORK=WARNING,NFS=WARNING,OBJECT=WARNING,PERFMON=WARNING,SCALEMGMT=WARNING,SMB=WARNING,CUSTOM=WARNING,AUTH_OBJ=WARNING,CES=WARNING,CESIP=WARNING,NODE=WARNING,THRESHOLD=WARNING,WATCHFOLDER=WARNING,NVME=WARNING,POWERHW=WARNING'
```

- Reports
```
AFM,AUTH,BLOCK,CESNETWORK,CLOUDGATEWAY,CLUSTERSTATE,DISK,FILEAUDITLOG,FILESYSTEM,GPFS,GUI,HADOOPCONNECTOR,KEYSTONE,MSGQUEUE,NETWORK,NFS,OBJECT,PERFMON,SCALEMGMT,SMB,CUSTOM,AUTH_OBJ,CES,CESIP,NODE,THRESHOLD,WATCHFOLDER,NVME,POWERHW 
```

```
scale_gui_email_recipients_name: 'acme_email_recipient_name' 
scale_gui_email_recipients_address: 'email_recipient_address@acme.com' 
removed - scale_gui_email_recipients_components: 'SCALEMGMT='
scale_gui_email_recipients_components_security_level: 'SCALEMGMT=WARNING' 
scale_gui_email_recipients_reports: 'DISK,GPFS'
scale_gui_email_recipients_quotaNotification ' --quotaNotification' ##if defined it enabled quota Notification
scale_gui_email_recipients_quotathreshold: '70.0'
```
- Example: Issue  the  following  command to add an email recipient named "acme_email_recipient_name" who is registered to receive reports on quota violations over 70% of the hard limit as well as an email for every 
WARNING event of the components SCALEMGMT and a report for all events of the components GPFS and DISK


Options:
      
      --address userAddress
              Specifies the address of the e-mail user. Optional.

      --reports listOfComponents
              Specifies the components to be reported. The tasks generating reports are scheduled by default to send a report once per day. Optional.

       --quotaNotification
              Enables quota notifications which are sent out if the specified threshold is violated. (See --quotathreshold)

       --quotathreshold valueInPercent
              Sets  the  threshold(percent  of the hard limit)for including quota violations in the quota digest report.  The default value is 100. The values -3, -2, -1, and zero have special meaning. Specify the value -2 to
              include all results, even entries where the hard quota not set. Specify the value -1 to include all entries where hard quota is set and current usage is greater than or equal to the soft quota.Specify the  value
              -3 to include all entries where hard quota is not set and current usage is greater than or equal to the soft quota only. Specify the value 0 to include all entries where the hard quota is set.

       Using unlisted options can lead to an error.       
   
       
#### SNMP Notifications Parameters

- To Configure SNMP Notification. Change the Server IP, Port and community.
```
scale_gui_snmp_notification: true
scale_gui_snmp_server_ip_adress: 'SNMPSERVER'
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

