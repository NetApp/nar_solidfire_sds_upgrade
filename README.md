Role name
=========
nar_solidfire_sds_upgrade


Description
===========
This role is designed to perform an update of NetApp Element software on a storage cluster using either NetApp SolidFire Enterprise SDS (eSDS) nodes or SolidFire all-flash storage (AFA) nodes.


Requirements for controller
===========================
This role requires that Python version 3.6 or later and Ansible version 2.10 or later are installed on the controller.

Requirements for eSDS cluster upgrade
=====================================
* This role requires that the target storage cluster has NetApp SolidFire eSDS storage nodes running minimum Element version 12.2 or later.

* This role also requires the `nar_solidfire_sds_install` role (https://github.com/NetApp-Automation/nar_solidfire_sds_install) to install the target solidfire-element software RPM package.

* This role performs some operations directly on each storage node. So, the controller system where Ansible is run needs to have network connections to all nodes in the target storage cluster. The inventory file for this role needs host information for all nodes, the target eSDS storage cluster MVIP and credentials, and the path to the target Element solidfire-element RPM package downloaded from NetApp Support Site.

* This role requires root or superuser privilege to run.

Requirements for SolidFire all-flash cluster upgrade
====================================================
* This role requires that the target storage cluster has NetApp SolidFire all-flash storage nodes running minimum Element version 12.2 or later and is managed by a 12.2 or later  management node (mNode) running latest NetApp Hybrid Cloud Control (HCC) services (https://<mNode_IP>).

* This role uses HCC APIs to execute the Element software upgrade and hence needs the Element iso.tar.gz package (solidfire-rtfi-xxx.iso.tar.gz) downloaded from NetApp Support Site to be uploaded to HCC before starting this role. Additionally, the controller system running this role needs to have a network connection to the mNode.

* The inventory file for this role needs to include mNode IP address, HCC authentication credentials, target Element version of the solidfire-rtfi package, and the target AFA storage cluster MVIP and credentials.


Role Variables
==============

| Variable                            | Required | Default  | Description                                                                           | Cluster  |
|-------------------------------------|----------|----------|---------------------------------------------------------------------------------------|----------|
| solidfire_element_rpm               | Yes      | N/A      | The RPM package URL or local file path on the controller. See examples in note [1]    | eSDS     |
| sf_mgmt_virt_ip                     | Yes      | N/A      | The Management Virtual IP address (MVIP) of the target cluster to be upgraded         | eSDS/AFA |
| sf_cluster_admin_passwd             | Yes      | N/A      | The password for the Administrator account of the target cluster to be upgraded       | eSDS/AFA |
| sf_cluster_admin_username           | Yes      | N/A      | The username for the Administrator account of the target cluster to be upgraded       | eSDS/AFA |
| mnode_ip                            | Yes      | N/A      | The management node (mNode) IP address                                                | AFA      |
| solidfire_rtfi_pkg_version          | Yes      | N/A      | The target Element version of the solidfire-rtfi package for upgrade, e.g. 12.3.0.777 | AFA      |
| hcc_auth_passwd                     | Yes      | N/A      | The password for the Administrator account of the authentication cluster used by HCC  | AFA      |
| hcc_auth_username                   | Yes      | N/A      | The username for the Administrator account of the authentication cluster used by HCC  | AFA      |
| sf_allow_resume_paused_upgrade      | No       | False    | "True" will allow resumption of a paused upgrade operation                            | AFA      |
| sf_cluster_connect_timeout          | No       | 20       | The API connection timeout value in seconds                                           | eSDS/AFA |
| sf_api_version                      | No       | 12.2     | The version of cluster API (should not be modified)                                   | eSDS/AFA |
| yes_i_want_to_ignore_cluster_faults | No       | False    | "True" will allow upgrade to proceed even if blocking faults exit, not recommended    | eSDS/AFA |
| sf_use_proxy                        | No       | True     | Whether to use proxy configuration on the target host. See note [2]                   | eSDS/AFA |
| sf_validate_certs                   | No       | False    | Whether to validate SSL/TLS certificates                                              | eSDS/AFA |
| sf_pip_extra_args                   | No       | ""       | Specify extra Python pip arguments when installing controller lib/modules             | eSDS/AFA |
| sf_maint_mode_duration              | No       | 02:00:00 | Max time duration in HH:MM:SS format that a node will stay in maintenance mode        | eSDS     |
| sf_allow_cluster_subset_upgrade     | No       | False    | "True" allows upgrading a subset of cluster nodes without updating the                | eSDS     |
|                                     |          |          | cluster version until all cluster nodes have been upgraded                            |          |

Notes
-----
\[1\]: Example RPM path:
```
Example of URL : `http://<server><:port>/<path>/solidfire-element-W.X.Y.Z-N.el{7,8}.x86_64.rpm`
Example of local path : `/<downloaded rpm path on Control node >/solidfire-element-<version>.<platform>.<arch>.rpm`
```
\[2\]:
The "False" value might have no effect due to a bug in Ansible version 2.10. Instead, it is recommended that you use
 the environment variable "no_proxy" in a playbook as a workaround until the bug is fixed by Ansible.
```
   environment:
     no_proxy: <target_ip_address>
```


Dependencies
------------
The `nar_solidfire_sds_install` role, available on GitHub (https://github.com/NetApp-Automation/nar_solidfire_sds_install) provides a few of the tasks used by this `nar_solidfire_sds_upgrade` role.


Example Playbooks
-----------------
Playbook file name: `update-eSDS-cluster.yml`

```
- name: Rolling Upgrade of SolidFire Enterprise SDS
  remote_user: <root or superuser ID>
  gather_facts: True
  hosts: all
  roles:
    - role: nar_solidfire_sds_upgrade
```

Playbook file name: 'update-afa-cluster.yml'
```
- name: Upgrade AFA cluster via HCC
  remote_user: <root or superuser ID>
  gather_facts: True
  hosts: localhost
  roles:
    - role: nar_solidfire_sds_upgrade
```

Example Inventory
-----------------
```
all:
  hosts:
    host1:
      ansible_host: <host1 IP>
    host2:
      ansible_host: <host2 IP>
    host3:
      ansible_host: <host3 IP>
    host4:
      ansible_host: <host4 IP>
  vars:
    # variables needed by both eSDS and AFA clusters
    ansible_python_interpreter: <full path on target hosts to a python interpreter, such as /usr/libexec/platform-python>
    become: true
    ansible_password: <login user password> # prefer vault key
    ansible_become_pass: <privilege escalation password> # or --ask-become-pass on command line to prompt for input

    sf_mgmt_virt_ip: <IP address for the cluster management interface>
    sf_cluster_admin_username: <username to communicate with cluster management interface>
    sf_cluster_admin_passwd: <password to communicate with clsuter management interface>

    # variables for eSDS cluster only
    solidfire_element_rpm: <RPM package file path on controller or remote server>

    # variables for AFA cluster only
    mnode_ip: <the management node IP address>
    solidfire_rtfi_pkg_version: <a SolidFire Element package version, such as 12.3.1.959>
    hcc_auth_username: <HCI UI quthentication username>
    hcc_auth_passwd: <HCI UI authentication password>
```

License
-------
MIT

Author Information
------------------
NetApp
https://www.netapp.com
