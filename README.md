saphana-deploy
==============

This adds HANA System Replication (HSR) to previously deployed SAP HANA 1.0 SPS 12 and above (HANA2) systems.

Requirements
------------

This role requires the system to be prepared with saphana-preconfigure and deployed with saphana-deploy role.

Role Variables
--------------

### Configuration for Instance deployment

This role uses the same variables as the saphana-preconfigure and saphana-deploy role.
It introduces the following additional variables:
- `hsr_deploy_type`: This has to be set `enable` on the primary and `register` on the secondary node.
- `hsr`: this is a dictionary that defines the parameters for enabling system replication. See example below

These  variables need to be set in the host_vars file. See [Best Practises](http://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html?highlight=host_var#group-and-host-variables) for more details.

### Example host_vars file for primary server 'node1'

     ---
     deployment_instance: true
     hsr_deploy_type: enable

     instances:
        l01:
           id_user_sidadm: "30210"
           pw_user_sidadm: "Adm12356"
           hana_pw_system_user_clear: "System123"
           hana_components: "client,server"
           hana_system_type: "Master"
           id_group_shm: "30220"
           hana_instance_hostname: node1
           hana_addhosts:
           hana_sid: L01
           hana_instance_number: 10
           hana_system_usage: custom
     hsr:
       l01:
         hana_instance_hostname: node1
         hana_sid: L01
         hana_instance_number: 10
         hana_pw_system_user_clear: "System123"
         hsr_name: DC1
         hsr_type: PRIMARY
         hsr_configure: yes
         hsr_type_remote_host: node2
         hsr_operation_mode: logreplay
         hsr_replicationmode: sync
         hsr_backup_directory: /hana/shared/L01/HDB10/backup/data

### Example host_vars file for secondary server 'node2'

     ---
     deployment_instance: true
     hsr_deploy_type: register

     instances:
        l01:
           id_user_sidadm: "30210"
           pw_user_sidadm: "Adm12356"
           hana_pw_system_user_clear: "System123"
           hana_components: "client,server"
           hana_system_type: "Master"
           id_group_shm: "30220"
           hana_instance_hostname: node2
           hana_addhosts:
           hana_sid: L01
           hana_instance_number: 10
           hana_system_usage: custom

     hsr:
       l01:
         hana_instance_hostname: node2
         hana_sid: L01
         hana_instance_number: 10
         hana_pw_system_user_clear: "System123"
         hsr_name: DC2
         hsr_type: SECONDARY
         hsr_configure: yes
         hsr_type_remote_host: node1
         hsr_operation_mode: logreplay
         hsr_replicationmode: sync
         hsr_backup_directory: /hana/shared/L01/HDB10/backup/data



Example Playbook
----------------

Here is an example playbook that installs two complete servers and setup system replication. Please note that this playbook requires correctly configured host_vars files.

    ---
    - hosts: hana
      remote_user: root

      vars:
              # subscribe-rhn role variables
              reg_activation_key: myregistration
              reg_organization_id: 123456

              repositories:
                      - rhel-7-server-rpms
                      - rhel-sap-hana-for-rhel-7-server-rpms

              # disk-init role variables
              disks:
                      /dev/vdc: vg00
                      /dev/vdb: vg00
              logvols:
                      hana_shared:
                              size: 24G
                              vol: vg00
                              mountpoint: /hana/shared
                      hana_data:
                              size: 24G
                              vol: vg00
                              mountpoint: /hana/data
                      hana_logs:
                              size: 12G
                              vol: vg00
                              mountpoint: /hana/logs
                      usr_sap:
                              size: 49G
                              vol: vg00
                              mountpoint: /usr/sap


              # rhel-system-roles.timesync variables
              ntp_servers:
                      - hostname: 0.rhel.pool.ntp.org
                        iburst: yes
                      - hostname: 1.rhel.pool.ntp.org
                        iburst: yes
                      - hostname: 2.rhel.pool.ntp.org
                        iburst: yes
                      - hostname: 3.rhel.pool.ntp.org
                        iburst: yes


              # SAP Precoonfigure role

              # SAP-Media Check
              install_nfs: "mynfsserver:/installi-export"
              installroot: /install
              hana_installdir: "{{ installroot + '/HANA2SPS02' }}"

              hana_pw_hostagent_ssl: "MyS3cret!"
              id_user_sapadm: "30200"
              id_group_shm: "30220"
              id_group_sapsys: "30200"
              pw_user_sapadm_clear: "MyS3cret!"

      roles:
              - { role: mk-ansible-roles.subscribe-rhn }
              - { role: mk-ansible-roles.disk-init }
              - { role: linux-system-roles.timesync }
              - { role: mk-ansible-roles.saphana-preconfigure }
              - { role: mk-ansible-roles.saphana-deploy }
              - { role: mk-ansible-roles.saphana-hsr }

License
-------

Apache License
Version 2.0, January 2004

Author Information
------------------

Markus Koch

Please leave comments in the github repo issue list
