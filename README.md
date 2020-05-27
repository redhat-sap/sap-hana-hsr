# sap-hana-hsr [![Build Status](https://travis-ci.com/redhat-sap/sap-hana-hsr.svg?branch=master)](https://travis-ci.com/redhat-sap/sap-hana-hsr)

This role enables SAP HANA Sysetem Replication between 2 given RHEL 7.x or 8.x hosts.

## Requirements

SAP HANA 1.0 SPS 12 and above (HANA2) must be installed and running on the given hosts.

## Role Variables

Variables to be used with this role must be added with different scopes. Some of the variables can be applied to both hosts, and some of them must apply individually to each host. This is due the nature of SAP HANA System Replication, where hosts will have different roles (primary and secondary) in the replication architecture.

### Common variables for each hosts

| variable | info | required? |
|:--------:|:----:|:---------:|
|sap_hana_hsr_hana_sid|SAP HANA System ID|yes|
|sap_hana_hsr_hana_instance_number|Instance Number|yes, **it must be declared as a string** e.g. "00"|
|sap_hana_hsr_hana_db_system_password|Database User (SYSTEM) Password|yes|
|sap_hana_hsr_hana_primary_hostname|System Hostname for the primary node|yes|

### Specific variables per host

| variable | info | required? |
|:--------:|:----:|:---------:|
|sap_hana_hsr_role|The host role in the replication architecture|yes, options are **primary** or **secondary**|
|sap_hana_hsr_alias|Alias used to identify the HANA Site Name in the replication layout|yes|

## Example Playbook

```yaml
    - hosts: hana
      roles:
        - { role: redhat_sap.sap_hana_hsr }
```

## Example Inventory

```yaml
# cat <inventory_dir>/group_vars/hana.yml
## Variables required for 'sap_hana_hsr' role
sap_hana_hsr_hana_sid: RH1
sap_hana_hsr_hana_instance_number: "00"
sap_hana_hsr_hana_db_system_password: "MyS3cr3tP4ssw0rd"
sap_hana_hsr_hana_primary_hostname: hana-0c0c0

# cat <inventory_dir>/host_vars/hana-0c0c0.yml
sap_hana_hsr_role: "primary"
sap_hana_hsr_alias: "DC1"

# cat <inventory_dir>/host_vars/hana-0c0c1.yml
sap_hana_hsr_role: "secondary"
sap_hana_hsr_alias: "DC2"
```

## HANA System Replication check

Once HANA System Replication has been configured using this role, you can check the actual status doing the following in the **primary** host:

```bash
# su - <sid>adm
# python /usr/sap/<SID>/HDB<INSTANCE_NUMBER>/exe/python_support/systemReplicationStatus.py

| Database | Host       | Port  | Service Name | Volume ID | Site ID | Site Name | Secondary  | Secondary | Secondary | Secondary | Secondary     | Replication | Replication | Replication    |
|          |            |       |              |           |         |           | Host       | Port      | Site ID   | Site Name | Active Status | Mode        | Status      | Status Details |
| -------- | ---------- | ----- | ------------ | --------- | ------- | --------- | ---------- | --------- | --------- | --------- | ------------- | ----------- | ----------- | -------------- |
| SYSTEMDB | hana-25e40 | 30001 | nameserver   |         1 |       1 | DC1       | hana-25e41 |     30001 |         2 | DC2       | YES           | SYNC        | ACTIVE      |                |
| RH1      | hana-25e40 | 30007 | xsengine     |         2 |       1 | DC1       | hana-25e41 |     30007 |         2 | DC2       | YES           | SYNC        | ACTIVE      |                |
| RH1      | hana-25e40 | 30003 | indexserver  |         3 |       1 | DC1       | hana-25e41 |     30003 |         2 | DC2       | YES           | SYNC        | ACTIVE      |                |

status system replication site "2": ACTIVE
overall system replication status: ACTIVE

Local System Replication State
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

mode: PRIMARY
site id: 1
site name: DC1
```

## License

GPLv3

## Author Information

Red Hat SAP Community of Practice
