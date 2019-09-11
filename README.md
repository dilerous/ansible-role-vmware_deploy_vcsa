vmware_deploy_vcsa
=========

Deploys the VMware vCenter Server Appliance from OVA.

The following configuration can be achieved:
- Update root account expire date or set to never expire.
- Configure Syslog.
- Join appliance to AD domain and configure identity sources.
- Import CA Signed Certificates.
- Configure vCenter general and statistics settings.
- Start and Stop services.
- Assign Global Permissions.
- Assign vCenter Administrators and Read Only users.
- Assign vCenter License.
- Create Datacenters.
- Create Clusters and configure HA, DRS and vSAN.
- Assign vSAN License (experimental).
- Add ESXi hosts.
- Assign ESXi License(s).

Supported VMware vCenter Server
------------

- VMware vCenter Server Appliance (VCSA) 6.5
- VMware vCenter Server Appliance (VCSA) 6.7

Requirements
------------

- python >= 2.6
- PyVmomi

Role Variables
--------------

### The following mandatory parameters need to be defined in host_vars:

#### Network Configuration

Set Network Configuration for the appliance.
```
network_ip_address: "x.x.x.x"
network_label: "VM Network"
network_prefix: "xx"
network_gateway: "x.x.x.x"
```

#### Credentials and Access

Set the SSH admin username and password for the appliance.
```
vcsa_admin_username: "root"
vcsa_admin_password: "VMwar3!!"
```

Set SSO Administrator username and password for the appliance.
```
vcsa_sso_username: "administrator@vsphere.local"
vcsa_sso_password: "VMwar3!!"
```

Set the Ansible connection variables (use exactly as shown)
```
ansible_user: "{{ vcsa_admin_username }}"
ansible_password: "{{ vcsa_admin_password }}"
ansible_host: "{{ network_ip_address }}"
```

### The following mandatory parameters need to be defined, as extra vars, or in group_vars or host_vars:

#### OVA Deployment Variables

Set the OVA deployment variables.
```
ova_deployment_hostname: "vcenter/esxi hostname"
ova_deployment_username: "vcenter/esxi username"
ova_deployment_password: "vcenter/esxi password"
```

Set the target datastore. Datastore clusters are not supported by the module.
```
ova_deployment_datastore: "datastore"
```

The following are only required when deploying to vCenter Server. If folder is not defined then the appliance will deploy to the default folder.
```
ova_deployment_datacenter: "vcenter datacenter"
ova_deployment_cluster: "vcenter cluster"
ova_deployment_folder: "vcenter folder"
```

#### OVA Configuration

Set the OVA file name.
```
ova_file: "ova_file.ova"
```

Set the local path to the OVA file (do not use a leading /).
```
ova_path: "/path/to/ova_file"
```

#### DNS Configuration

Set the DNS domain that should be used.
```
dns_domain: "example.com"
```

Provide a list of available DNS Servers.
```
dns_servers:
  - "x.x.x.x"
  - "x.x.x.x"
```

### The following optional parameters can be defined, as extra vars, or in group_vars or host_vars:

#### OVA Download Configuration

Set the URL to the OVA file if '**ova_source**' is set to '**http**' (do not use a leading /). The '**ova_source**' variable defaults to '**local**' in the **vmware_deploy_ova** role and can be overridden.
```
ova_url: "http[s]://example.com/ova"
```

#### Root Account Policies

Set the root account policies. Set the following variables to override the defaults shown.
```
vcsa_root_expiration_disable: no
vcsa_root_expiration_days: 90
```

#### Import CA Signed Certificates

Set '**vcsa_use_signed_certificate**' to '**yes**' if you would like to import CA signed certificates. The default setting is '**no**'.
```
vcsa_use_signed_certificate: no
```

If this setting is enabled, then the the following certificates are requird and should be placed in the '**files/certs**' folder for the role.
- Host certificate with the file name '**hostname.pem**' (the hostname must match what has been set in the inventory). The PEM file must include the host certificate and CA chain.
- Host certificate key with the file name '**hostname.key**' (the hostname must match what has been set in the inventory). This needs to be the unencrypted file if it has been encrypted.
- CA root certificate with the file name '**ca.crt**'.

#### Active Directory Domain Membership

Set whether the appliance should be joined to an Active Directory domain. Default is '**no**' but can be overridden by setting the following variable to '**yes**'
```
vcsa_join_to_domain: no
```

Set Active Directory membership settings when '**vcsa_join_to_domain**' is set to '**yes**'.
```
vcsa_ad_dom_join_domain: "ad.domain.local"
vcsa_ad_dom_join_username: "svc_dom_join@ad.domain.local"
vcsa_ad_dom_join_password: "VMwar3!!"
```

The OU in which to create the AD computer object can also be specified. Do not set this if you wish to use the default container.
```
vcsa_ad_dom_join_ou: "CN=Computers,DC=AD,DC=DOMAIN,DC=LOCAL"
```

#### Apply vCenter License Key

Set the vCenter License key that should be applied to the vCenter Server.
```
vcsa_license_key: XXXXX-XXXXX-XXXXX-XXXXX-XXXXX
```

#### vCenter Services

##### List of Services that should be started

Set the following variable to list the services which should be started and set to AUTOMATIC start-up type. The list provided is just an example.
```
vcsa_services_to_start:
  - updatemgr
  - vsphere-ui
  - content-library
  - vapi-endpoint
```

##### List of Services that should be stopped

Set the following variable to list the services which should be stopped and set to MANUAL start-up type. The list provided is just an example.
```
vcsa_services_to_stop:
  - rbd
  - imagebuilder
```

#### vCenter Permissions

##### Define Global Administrators

Set the following variable to a list of groups that should be assigned '**Global Administrators**' rights. The appliance needs to be joined to an Active Directory domain for these permissions to be applied. The groups will be discovered from the Active Directory domain that the the appliance was joined to.
```
vcsa_global_admin_groups:
  - "vSphere-Gloabl-Admins"
```

##### Define vCenter Administrators

Set the following variable to a list of groups that should be assigned the '**Administrator**' role for the vCenter Server. The appliance needs to be joined to an Active Directory domain for these permissions to be applied. The groups will be discovered from the Active Directory domain that the the appliance was joined to.
```
vcsa_vcenter_admin_groups:
  - "vSphere-Admins"
```

##### Define vCenter Read-Only Users

Set the following variable to a list of groups that should be assigned the '**Read-Only**' role for the vCenter Server. The appliance needs to be joined to an Active Directory domain for these permissions to be applied. The groups will be discovered from the Active Directory domain that the the appliance was joined to.
```
vcsa_vcenter_readonly_groups:
  - "vSphere-ReadOnly"
```

#### vCenter Settings

The following variables can be used to configure vCenter settings. All values displayed are the default settings.

##### Database Settings

Use the following variables to configure vCenter database settings.
```
vcsa_database_max_connections: 50
vcsa_task_cleanup: true
vcsa_task_retention: 30
vcsa_event_cleanup: true
vcsa_event_retention: 30
```

##### Runtime Settings

Use the following variables to configure vCenter runtime settings.
```
vcsa_unique_id: 1
```

##### User Directory Settings

Use the following variables to configure vCenter user directory settings.
```
vcsa_user_directory_timeout: 60
vcsa_user_directory_query_limit: true
vcsa_user_directory_query_limit_size: 5000
vcsa_user_directory_validation: true
vcsa_user_directory_validation_period: 1400
```

##### Mail Server Settings

Use the following variables to configure vCenter mail settings.
```
vcsa_mail_server: ""
vcsa_mail_sender: ""
```

##### SNMP Receiver Settings

Use the following variables to configure vCenter SNMP receiver settings.
```
vcsa_snmp_receiver_url: "localhost"
vcsa_snmp_receiver_enabled: true
vcsa_snmp_receiver_port: 162
vcsa_snmp_receiver_community: "public"
```

##### Timeout Settings

Use the following variables to configure vCenter timeout settings.
```
vcsa_normal_operations_timeout: 30
vcsa_long_operations_timeout: 120
```

##### Logging Settings

Use the following variables to configure vCenter logging settings.
```
vcsa_logging_level: "info"
```

##### Statistics Settings

Use the following variables to configure vCenter statistics settings.
```
vcsa_interval_past_day_level: 1
vcsa_interval_past_week_level: 1
vcsa_interval_past_month_level: 1
vcsa_interval_past_year_level: 1
```

#### Inventory Configuration

Use the following variables to create objects in the vCenter Server inventory, such as datacenters, clusters and ESXi hosts.

##### Create Datacenters

Set the following variable to list the datacenters that should be created. The list provided is just an example.
```
vcsa_datacenters:
  - DC1
  - DC2
```

##### Create Clusters

Set the following variable to list the clusters that should be created. The list provided is an example. The cluster settings can be ommited to use the default values shown (as per the '**Cluster-02**' example). If '**vsan_license_key**' is not provided then the default evaluation license will be used.
```
vcsa_clusters:
  - name: "Cluster-01"
    datacenter: "DC1"
    enable_ha: yes
    ha_vm_monitoring: "vmMonitoringOnly"
    enable_drs: yes
    drs_vm_behavior: "fullyAutomated"
    enable_vsan: no
    vsan_license_key: XXXXX-XXXXX-XXXXX-XXXXX-XXXXX
    vsan_autoclaim_storage: no
  - name: "Cluster-02"
    datacenter: "DC1"
```

##### Add ESXi Hosts

Set the following variable to list the ESXi hosts that should be added. The list provided is just an example. If '**license_key**' is not provided then the default evaluation license will be used.
```
vcsa_esxi_hosts:
  - hostname: "esxi01.domain.local"
    datacenter: "DC1"
    cluster: "Cluster-01"
    license_key: XXXXX-XXXXX-XXXXX-XXXXX-XXXXX
  - hostname: "esxi02.domain.local"
    datacenter: "DC1"
    cluster: "Cluster-01"
```

Each ESXi host can also be added to the Ansible inventory with the following host_vars variables set. This allows for variables to be set on a per-host basis.
```
esxi_admin_username: "root"
esxi_admin_password: "VMwar3!!"
esxi_license_key: XXXXX-XXXXX-XXXXX-XXXXX-XXXXX
```

If you wish to use global credentials for all ESXi hosts then set the following credential variables in a group_var.
```
default_esxi_admin_username: "root"
default_esxi_admin_password: "VMwar3!!"
```

Note that '**esxi_admin_username**' and '**esxi_admin_password**' will always override the defaults.

### Additional default variables

The following additional default variables have also been set and can be overridden by setting them in a group_var.

#### Default list of NTP Servers that will be used.
```
ntp_servers:
  - "0.pool.ntp.org"
  - "1.pool.ntp.org"
  - "2.pool.ntp.org"
  - "3.pool.ntp.org"
```

#### HTTP REST API Variables
```
http_content_type: "application/json"
http_accept: "application/json"
http_validate_certs: no
http_body_format: "json"
```

#### Default for enabling certificate verification when connecting to the VCSA appliance.
```
vcsa_validate_certs: no
```

Dependencies
------------
  ```
  - { role: vmware_deploy_ova, tags: [ 'deploy' ] }
  ```

Example Playbook
----------------
    ```
    - hosts: vcsa_appliances
      become: no
      roles:
        - nmshadey.vmware_deploy_vcsa
    ```

License
-------

MIT

Author Information
------------------

Gavin Stephens (https://www.simplygeek.co.uk)