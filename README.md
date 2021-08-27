Ansible Role: iSCSI Target
=========

This role configure a Linux-LIO based iSCSI target on a Linux host using `targetcli`

Additionaly this role includes own python mudules to interact with targetcli command which can be used separately for more advanced stuff. Modules implement checking, creating and deleting.

> NOTE: Documentation about LIO and Target can be found [here] (https://linux-iscsi.org/wiki/Main_Page).

Requirements
------------

This role is not creating any disks/partitions/LVs. It is expected that they are already present on machine or created by some other role. For example: [ricsanfre.storage](https://galaxy.ansible.com/ricsanfre/storage)

Role Variables
--------------


To configure the iSCSI target, the following nested variable is used which defines how the configuration should look like.

```yml
iscsi_targets:
  - name: "iqn.2021-07.com.ricsanfre:target_server"
    disks:
      - name: lun_node1
        path: /dev/vg_iscsi/vg_iscsi_lv_node1
        type: block
        lunid: 0
      - name: lun_node2
        path: /dev/vg_iscsi/vg_iscsi_lv_node2
         type: block
        lunid: 1
      - name: lun_node3
        path: /dev/vg_iscsi/vg_iscsi_lv_node3
        type: block
        lunid: 2
      - name: lun_node4
        path: /dev/vg_iscsi/vg_iscsi_lv_node4
        type: block
        lunid: 3
    initiators:
      - name: iqn.2021-07.com.ricsanfre:node1
        authentication:
          userid: node1
          password: passwd1
          userid_mutual: sharedkey
          password_mutual: sharedsecret
        mapped_luns:
          - mapped_lunid: 0
            lunid: 0
          - mapped_lunid: 1
            lunid: 2
      - name: iqn.2021-07.com.ricsanfre:node2
        authentication:
          userid: node2
          password: passwd2
        mapped_luns:
          - mapped_lunid: 0
            lunid: 1
          - mapped_lunid: 1
            lunid: 3
            write_protect: 1
    portals:
      - ip: 192.168.1.45
        port: 5555
      - ip: 192.168.2.10
```


Dependencies
------------

None.

Example Playbook
----------------

This example use `ricsanfre.storage` role for creating the logical volumes used to configure the iSCSI target

```yml
- hosts: all
  become: true
  gather_facts: true
  vars:
    storage_partitions:
      - name: /dev/vdb
        number: 1
        flags:
          - lvm
        part_end: 1GB
    storage_volumegroups:
      - name: vg_iscsi
        devices:
          - /dev/vdb1
    storage_volumes:
      - name: vg_iscsi_lv_node1
        vg: vg_iscsi
        size: 100
      - name: vg_iscsi_lv_node2
        vg: vg_iscsi
        size: 100
      - name: vg_iscsi_lv_node3
        vg: vg_iscsi
        size: 100
      - name: vg_iscsi_lv_node4
        vg: vg_iscsi
        size: 100
    iscsi_targets:
      - name: "iqn.2021-07.com.ricsanfre:{{ ansible_facts['nodename'] }}"
        disks:
          - name: lun_node1
            path: /dev/vg_iscsi/vg_iscsi_lv_node1
            type: block
            lunid: 0
          - name: lun_node2
            path: /dev/vg_iscsi/vg_iscsi_lv_node2
            type: block
            lunid: 1
          - name: lun_node3
            path: /dev/vg_iscsi/vg_iscsi_lv_node3
            type: block
            lunid: 2
          - name: lun_node4
            path: /dev/vg_iscsi/vg_iscsi_lv_node4
            type: block
            lunid: 3
        initiators:
          - name: iqn.2021-07.com.ricsanfre:node1
            authentication:
              userid: node1
              password: passwd1
            mapped_luns:
              - mapped_lunid: 0
                lunid: 0
              - mapped_lunid: 1
                lunid: 2
          - name: iqn.2021-07.com.ricsanfre:node2
            authentication:
              userid: node2
              password: passwd2
            mapped_luns:
              - mapped_lunid: 0
                lunid: 1
              - mapped_lunid: 1
                lunid: 3
                write_protect: 1
        portals:
          - ip: "{{ ansible_default_ipv4.address | default(ansible_all_ipv4_addresses[0]) }}"

  roles:
    - ricsanfre.storage
    - ricsanfre.iscsi_target
```

License
-------

MIT/BSD

Author Information
------------------

Created by Ricardo Sanchez (ricsanfre) taking as base the development [targetcli](https://github.com/OndrejHome/ansible.targetcli) from Ondrej Famera <ondrej-xa2iel8u@famera.cz> 
