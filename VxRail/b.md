#author:  Lukasz Stasiak
#date:    05.2019
#version:   0.3
#description:  This playbook will:
#-create HashicorpVault VM from Ubuntu template
#-deploy Hashicorp Vault on cretaed VM
#-configure Hashicorp Vault (without LDAP configuration and internal CA certificate as this will be done in separate playbook)
#0.2 18.12.2019 Łukasz Stasiak -Added creation of a Vault paths used to store secrets for:activedirectory,servers,templates,vracloud,antivirus and backup.
#0.3 22.06.2020 Łukasz Stasiak -Added creation of a Vault path for alcatraz.
---
- name: Create HashiVault Host from template (Ubuntu)
  hosts: localhost
  gather_facts: false

  tasks:
    - name: "Running Role: dpc-createLinuxVm '{{ mgmtDns.hsv001.name }}'"
      include_role:
        name: dpc-createLinuxVm
      vars:
        portGroup: "{{ networkAvnLocalRegion.name }}"
        vmName: "{{ mgmtDns.hsv001.name }}"
        vmIP: "{{ mgmtDns.hsv001.cidr }}.{{  mgmtDns.hsv001.octet }}"
        vmNetmask: "{{ networkAvnLocalRegion.netmask }}"
        vmGW: "{{ networkAvnLocalRegion.cidr }}.{{ networkAvnLocalRegion.gw }}"
      run_once: yes

- name: configure ubuntu compliance
  hosts: hsv001
  gather_facts: false
  tasks:
    - name: Configure Ubuntu Compliance
      import_role:
        name: dhc-configureUbuntuCompliance

- name: Deploy Hashicorp Vault on created VM
  hosts: hsv001
  gather_facts: false
  tasks:
    - name: "Run role: dpc-createVault  on  '{{ mgmtDns.hsv001.name }}'"
      include_role:
        name: dpc-createVault
      run_once: yes

- name: Configure Hashicorp Vault
  hosts: hsv001
  gather_facts: false
  tasks:
    - name: "Run role: dpc-configureVault  on  '{{ mgmtDns.hsv001.name }}'"
      include_role:
        name: dpc-configureVault
      run_once: yes

- name: Pre-create DHC Vault paths
  hosts: localhost
  gather_facts: false

  tasks:
    - name: "Create activedirectory path"
      include_role:
        name: dpc-createSecretVaultEntry
      vars:
        vaultMachineName: "activedirectory"
        vaultAccountName: "path_entry"
        vaultAccountPass: "path_entry"

    - name: "Create servers path"
      include_role:
        name: dpc-createSecretVaultEntry
      vars:
        vaultMachineName: "servers"
        vaultAccountName: "path_entry"
        vaultAccountPass: "path_entry"

    - name: "Create templates path"
      include_role:
        name: dpc-createSecretVaultEntry
      vars:
        vaultMachineName: "templates"
        vaultAccountName: "path_entry"
        vaultAccountPass: "path_entry"

    - name: "Create vracloud path"
      include_role:
        name: dpc-createSecretVaultEntry
      vars:
        vaultMachineName: "vracloud"
        vaultAccountName: "path_entry"
        vaultAccountPass: "path_entry"

    - name: "Create antivirus path"
      include_role:
        name: dpc-createSecretVaultEntry
      vars:
        vaultMachineName: "antivirus"
        vaultAccountName: "path_entry"
        vaultAccountPass: "path_entry"

    - name: "Create backup path"
      include_role:
        name: dpc-createSecretVaultEntry
      vars:
        vaultMachineName: "backup"
        vaultAccountName: "path_entry"
        vaultAccountPass: "path_entry"

    - name: "Create alcatraz path"
      include_role:
        name: dpc-createSecretVaultEntry
      vars:
        vaultMachineName: "alcatraz"
        vaultAccountName: "path_entry"
        vaultAccountPass: "path_entry"
