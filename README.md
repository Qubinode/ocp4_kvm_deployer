![Ansible Lint](https://github.com/Qubinode/ocp4_kvm_deployer/workflows/Ansible%20Lint/badge.svg?branch=dev)

ocp4_kvm_deployer
=================


This role will deploy OCP4 and OKD4 on a kvm host. It was develop to support the [Qubinode Installer](https://github.com/Qubinode/qubinode-installer).

Requirements
------------

This role depends on Red Hat IdM as the DNS server. PR's are welcome to support other DNS servers.

Role Variables
--------------

Please see example playbook.

Dependencies
------------

Roles:

  - [openshift-4-loadbalancer](https://github.com/Qubinode/openshift-4-loadbalancer)
  - [ocp4-bootstrap-webserver](https://github.com/Qubinode/ocp4-bootstrap-webserver)

Example Playbook Deploying OpenShift 4 Cluster
----------------

```
  - name: Deploy OpenShift 4.x Cluster
    hosts: localhost
    become: yes
    vars:
      local_user_account: admin
      ocp4_okd4: ocp4
      ocp4_release: 4.4.6
      family_tree: "Red Hat CoreOS"
      ocp4_dependencies_version: "{{ ocp4_release[:3] }}"
      ocp4_image_version: "{{ ocp4_release[:3] + '.0' }}"
      installation_working_dir: /home/admin/qubinode-installer
      pull_secret: "{{ installation_working_dir }}/pull-secret.txt"
      vm_public_key: "/home/{{ local_user_account }}/.ssh/id_rsa.pub"
      openshift_install_folder: ocp4
      openshift_install_dir: "{{ installation_working_dir }}/{{ openshift_install_folder }}"
      ignition_files_dir: "{{ openshift_install_dir }}"
      ssh_ocp4_public_key: "{{ lookup('file', vm_public_key) }}"
      podman_webserver: qbn-httpd
      rhcos_webserver_img_name: rhcos-webserver
      dest_ignitions_web_directory: "{{ webserver_directory }}/{{ ocp4_dependencies_version }}/ignitions/"
      webserver_directory: /opt/qubinode_webserver
      webserver_dependencies: "{{ webserver_directory }}/{{ ocp4_dependencies_version }}"
      webserver_images: "{{ webserver_directory }}/{{ ocp4_dependencies_version }}/images"
      coreos_installer_kernel: "fedora-coreos-{{ major_version }}-live-kernel-x86_64"   
      coreos_installer_initramfs: "fedora-coreos-{{ major_version }}-live-initramfs.x86_64.img"
      coreos_metal_bios: "fedora-coreos-{{ major_version }}-metal.x86_64.raw.xz"
      openshift_mirror: "https://builds.coreos.fedoraproject.org/prod/streams/stable/builds/{{ major_version }}/x86_64"
      coreos_tmp_dir: /tmp/build_coreos_container
      tear_down: false
      virtinstall_dir: "{{ installation_working_dir }}/rhcos-install/"
      internal_domain_name: lunchnet.example
      external_domain_name: "{{ internal_domain_name }}"
      ocp4_cluster_domain: "cloud.{{ internal_domain_name }}"
      idm_server_shortname: qbn-dns01
      user_idm_admin: admin
      user_idm_password: "password"
      idm_server_ipaddr: 192.168.11.1
      dns_teardown: false
      idm_dns_forward_zone: "{{ ocp4_cluster_domain }}"
      idm_dns_reverse_zone: "50.168.192.in-addr.arpa."
      idm_server_fqdn: qbn-dns01.lunchnet.example
      dns_wildcard: "*.apps.{{ cluster_name }}"
      nat_gateway: "192.168.50.1"
      localstorage_version: '4.3'
      localstorage_filesystem: true
      localstorage_block: false
      localstorage_block_disk: /dev/vdc
      localstorage_fs_disk: /dev/vdb

    environment:
      IPA_HOST: "{{idm_server_shortname}}.{{ internal_domain_name }}"
      IPA_USER: "{{ user_idm_admin }}"
      IPA_PASS: "{{ user_idm_password }}"

    tasks:
    - name: run the role ocp4_kvm_deployer
      import_role:
        name: ocp4_kvm_deployer
```

Example Playbook Deploying OKD 4 Cluster
----------------

```
  - name: Deploy OKD 4.x Cluster
    hosts: localhost
    become: yes
    vars:
      local_user_account: admin
      ocp4_okd4: okd4
      ocp4_release: 4.4.0-0.okd-2020-05-23-055148-beta5
      ocp4_dependencies_version: "{{ major_version }}"
      ocp4_image_version: "{{ major_version }}"
      installation_working_dir: /home/admin/qubinode-installer
      ocp4_release_url: "https://github.com/openshift/okd/releases/download/{{ ocp4_release }}"
      family_tree: "Fedora CoreOS"
      pull_secret: "{{ installation_working_dir }}/pull-secret.txt"
      vm_public_key: "/home/{{ local_user_account }}/.ssh/id_rsa.pub"
      openshift_install_folder: ocp4
      openshift_install_dir: "{{ installation_working_dir }}/{{ openshift_install_folder }}"
      ignition_files_dir: "{{ openshift_install_dir }}"
      ssh_ocp4_public_key: "{{ lookup('file', vm_public_key) }}"
      podman_webserver: qbn-httpd
      rhcos_webserver_img_name: rhcos-webserver
      dest_ignitions_web_directory: "{{ webserver_directory }}/{{ ocp4_dependencies_version }}/ignitions/"
      webserver_directory: /opt/qubinode_webserver
      webserver_dependencies: "{{ webserver_directory }}/{{ ocp4_dependencies_version }}"
      webserver_images: "{{ webserver_directory }}/{{ ocp4_dependencies_version }}/images"
      coreos_installer_kernel: "rhcos-{{ ocp4_image_version }}-x86_64-installer-kernel"
      coreos_installer_initramfs: "rhcos-{{ ocp4_image_version }}-x86_64-installer-initramfs.img"
      coreos_metal_bios: "rhcos-{{ ocp4_image_version }}-x86_64-metal.raw.gz"
      openshift_mirror: http://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/{{ ocp4_dependencies_version }}/{{ ocp4_image_version }}
      coreos_tmp_dir: /tmp/build_coreos_container
      tear_down: false
      virtinstall_dir: "{{ installation_working_dir }}/rhcos-install/"
      internal_domain_name: "{{ domain }}"
      external_domain_name: "{{ domain }}"
      libivrt_network_domain: "{{ cluster_name }}.{{ ocp4_subdomain }}.{{ domain }}"
      ocp4_cluster_domain: "cloud.{{ internal_domain_name }}"
      idm_server_shortname: qbn-dns01
      user_idm_admin: admin
      user_idm_password: "password"
      idm_server_ipaddr: 192.168.11.1
      dns_teardown: false
      idm_dns_forward_zone: "{{ ocp4_cluster_domain }}"
      idm_dns_reverse_zone: "50.168.192.in-addr.arpa."
      idm_server_fqdn: qbn-dns01.lunchnet.example
      dns_wildcard: "*.apps.{{ cluster_name }}"
      nat_gateway: "192.168.50.1"
      localstorage_version: '4.3'
      localstorage_filesystem: true
      localstorage_block: false
      localstorage_block_disk: /dev/vdc
      localstorage_fs_disk: /dev/vdb

    environment:
      IPA_HOST: "{{idm_server_shortname}}.{{ internal_domain_name }}"
      IPA_USER: "{{ user_idm_admin }}"
      IPA_PASS: "{{ user_idm_password }}"

    tasks:
    - name: run the role ocp4_kvm_deployer
      import_role:
        name: ocp4_kvm_deployer
```

For OKD 4 Deployments use the following pull secert
```
$ cat pull-secret.txt 
{"auths":{"fake":{"auth": "bar"}}}
```

Additional Details About The Role
----------------------------------
The role has three main tasks file:

* pre_tasks.yml
* tear_down.yml
* deploy.yml

### **TASK: pre_tasks.yml**

- Sets up global variables for load balancer contaier
- Ensure the jq command is installed

### **TASK: tear_down.yml**

The tasks is responsible for tearing down the OCP cluster. It will remove all DNS entries, remove the VMs, remove the containers and any other files created by the role.

Example Usage:

```sh
# Teardown the entire cluster
ansible-playbook rhcos.yml -e "tear_down=true"

# Remove just the webserver container
ansible-playbook rhcos.yml -e "tear_down=true" -t webserver

```

Dependancy roles:
  - openshift-4-loadbalancer
  - ocp4-bootstrap-webserver

### **TASK: deploy.yml**

1. check for ocp4 pull secret and fail if it's not found
2. ensure the kvm host firewall ports are open
3. install_podman.yml: ensure the kvm host is setup to run podman containers
4. ocp4_tools.yml: ensure the ocp4 tools such as the oc command and the openshift installer are installed
5. create_ignitions.yml: generate ignitions files based on the number of ctrlplane and computes specified for this cluster
6. deploy the container load balancer
7. deploy the container webserver that servers up the files for bootstrap
8. download_rhcos_files.yml: download the files required to bootstrap rhcos
9. create directory to store the generated virt-install scripts for rhcos
10. deploy a generated treeinfo that defines the location of the kernel and ramdisks
11. deploy_libvirt_network.yml: create libvirt nat network for rhcos vms
12. configure_dns_entries.yml: populate Red Hat IdM with the dns entries for OCP4
13. build_cluster_nodes_profile.yml: generate the virt-install scripts for building the rhcos VMs
14. build_vm_list.yml: create the variable *ocp4_nodes* with a list of all the ocp4 required for the cluster
15. deploy the ocp4 rhcos node vms
16. wait_for_vm_shutdown.yml: OCP nodes shutdown after the initial boostrap, this tasks waits for all the VMs to shutdown
17. start up the RHCOS vms to continue the OCP deployment
18. run /usr/local/bin/openshift-install wait-for bootstrap-complete
19. contine install if bootstrap returns `It is now safe to remove the bootstrap resources`
20. get all running VMS
21. shutdown the bootstrap node
22. wait_for_vm_shutdown.yml: wait for the bootstrap node to shutdown
23. destroy the bootstrap node




Configure nfs provisioner
```
ansible-playbook playbooks/deploy_ocp4.yml --start-at-task="Waiting for Installation to Complete"
```

Available Ansible Tags
----------------------

**IdM dns**

 * idm - Create all IdM dns entries
 * dns - Same as above
 * bootstrap_dns - Only bootstrap node
 * api_dns - Only api entries
 * ctrlplane_dns - Only control nodes
 * compute_dns - Only compute nodes
 * wildcard_dns - Create wild card *.apps
 * routes - same as above
 * fwd_zone
 * ptr_zone
 
Examples 

* Populate dns for worker/compute nodes

```
ansible-playbook playbooks/deploy_ocp4.yml -t fwd_zone,ptr_zone,compute_dns
```

* Delete all A and PTR records

```
ansible-playbook playbooks/deploy_ocp4.yml -t idm -e "idm_record_state=absent"
```

* Delete the forward zone

```
ansible-playbook playbooks/deploy_ocp4.yml -t idm -e "idm_fwd_zone_state=absent"
```

**Create Libvirt NAT network**

```
ansible-playbook playbooks/deploy_ocp4.yml -t libvirt_nat
```

**Setup Firewall**

```
ansible-playbook playbooks/deploy_ocp4.yml -t firewall
```

**Setup OpenShift client tools**

```
ansible-playbook playbooks/deploy_ocp4.yml -t tools
```

**Create OpenShift Ignitions**

```
ansible-playbook playbooks/deploy_ocp4.yml -t ignitions
```

**Download RHCOS kernel, initramfs, rootfs**

```
ansible-playbook playbooks/deploy_ocp4.yml -t rhcos_files
```

**Generate virt-install scripts for deploying RHCOS nodes**

```
ansible-playbook playbooks/deploy_ocp4.yml -t node_profile
```

**Start required containers**

```
ansible-playbook playbooks/deploy_ocp4.yml -t containers -e "container_exist=no" -e "tear_down=no"
```

**Deploy RHCOS VMs**

```
ansible-playbook playbooks/deploy_ocp4.yml -t deploy_nodes -e "container_exist=no" -e "tear_down=no" -e "check_existing_cluster=no"
```

**Bootstrap the OCP4 Cluster**
```
ansible-playbook playbooks/deploy_ocp4.yml -t bootstrap_cluster -e "container_exist=no" -e "tear_down=no" -e "check_existing_cluster=no" -e "cluster_install_status=no" -e "bootstrap_precheck=yes"
```

**Deploy Registry**
```
ansible-playbook playbooks/deploy_ocp4.yml -t configure_registry -e "container_exist=no" -e "tear_down=no" -e "check_existing_cluster=no" -e "cluster_install_status=no" -e "bootstrap_precheck=yes" -e "bootstrap_complete=yes"
```

**Complete OpenShift Install**
```
ansible-playbook playbooks/deploy_ocp4.yml -t complete_cluster_install -e "tear_down=no" -e "check_existing_cluster=no" -e "cluster_install_status=no" -e "bootstrap_precheck=yes" -e "bootstrap_complete=yes"
```

**Deploy nodes on External network**
```
export PLAYBOOK_NAME=playbooks/deploy_ocp4.yml #PLAYBOOK_NAME=rhcos.yml

ansible-playbook $PLAYBOOK_NAME  -t setup
ansible-playbook $PLAYBOOK_NAME  -t tools
ansible-playbook $PLAYBOOK_NAME  -t podman
ansible-playbook $PLAYBOOK_NAME  -t podman --skip-tags pkg
ansible-playbook $PLAYBOOK_NAME -t ignitions --extra-vars "download_ocp_tools=true"
ansible-playbook $PLAYBOOK_NAME -t idm,containers,lb --extra-vars "tear_down=false"
ansible-playbook  $PLAYBOOK_NAME  -t webserver
ansible-playbook $PLAYBOOK_NAME -t download
#ansible-playbook $PLAYBOOK_NAME -t libvirt_net
ansible-playbook $PLAYBOOK_NAME -t deploy_nodes 
```

**Deploy nodes on Internal libvirt network**
```
WIP
```
**Configure NFS**
```
ansible-playbook rhcos.yml -t nfs --extra-vars "configure_nfs_storage=true" --extra-vars "bootstrap_complete=true" 
```
**Remove NFS**
```
ansible-playbook playbooks/deploy_ocp4.yml  -t nfs --extra-vars "configure_nfs_storage=false" --extra-vars "bootstrap_complete=true"  --extra-vars "delete_deployment=true"
```

**Configure Local strorage**
```
ansible-playbook rhcos.yml -t localstorage  --extra-vars "bootstrap_complete=true" 
```
**Configure OCS**
```
ansible-playbook rhcos.yml -t ocs --extra-vars "configure_openshift_storage=true"  --extra-vars "bootstrap_complete=true" 
```
Dependancy roles:
  - openshift-4-loadbalancer
  - nfs-provisioner-role


### add worker external worker  
*Requirement: Openshift must be deployed on external network*
* update worker count
```
vim playbooks/vars/ocp4.yml
compute_count: 6
```

* remove lb 
```
export PLAYBOOK_NAME=playbooks/deploy_ocp4.yml #PLAYBOOK_NAME=rhcos.yml
ansible-playbook $PLAYBOOK_NAME -t lb --extra-vars "tear_down=true"
```
* Add lb
```
ansible-playbook $PLAYBOOK_NAME -t idm,containers,lb --extra-vars "tear_down=false"
```

Example PXE boot Menu

*If you are using a sepreate pxe server* 
*Add the kernel and initramfs files to webserver*
```
# on qubinode box
cp rhcos-install/rhcos-live-* /opt/qubinode_webserver/4.7/images/
sudo chmod 775 -R /opt/qubinode_webserver/4.7/images/
```

Download  kernel and initramfs to pxe server
```
mkdir -p /mnt/data/netboot/boot/amd64/coreos/4.7.7/
cd /mnt/data/netboot/boot/amd64/coreos/4.7.7/

curl  -OL http://192.168.1.10:8080/pub/4.7/images/rhcos-live-kernel-x86_64
curl -OL http://192.168.1.10:8080/pub/4.7/images/rhcos-live-initramfs.x86_64.img
```

Example PXE config
*Double check idm server for ips*
```
        MENU BEGIN CoreOS
        MENU TITLE CoreOS
            LABEL compute3
                MENU LABEL ^Computess 3 Install
                KERNEL ::boot/amd64/coreos/4.7.7/rhcos-live-kernel-x86_64
                INITRD ::boot/amd64/coreos/4.7.7/rhcos-live-initramfs.x86_64.img
                APPEND coreos.live.rootfs_url=http://192.168.1.10:8080/pub/4.7/images/rhcos-4.7.7-x86_64-live-rootfs.x86_64.img coreos.inst.install_dev=/dev/sda coreos.inst.ignition_url=http://192.168.1.10:8080/pub/4.7/ignitions/worker.ign ip=192.168.1.88::192.168.1.1:255.255.255.0:qbn-ocp4-compute-3.qub593.cloud.qubinodeexample.com:ens192:none nameserver=192.168.1.150 

            LABEL compute4
                MENU LABEL ^Compute 4 Install
                KERNEL ::boot/amd64/coreos/4.7.7/rhcos-live-kernel-x86_64
                INITRD ::boot/amd64/coreos/4.7.7/rhcos-live-initramfs.x86_64.img
                APPEND coreos.live.rootfs_url=http://192.168.1.10:8080/pub/4.7/images/rhcos-4.7.7-x86_64-live-rootfs.x86_64.img coreos.inst.install_dev=/dev/sda coreos.inst.ignition_url=http://192.168.1.10:8080/pub/4.7/ignitions/worker.ign ip=192.168.1.89::192.168.1.1:255.255.255.0:qbn-ocp4-compute-4 .qub593.cloud.qubinodeexample.com:ens192:none nameserver=192.168.1.150 
            
            LABEL compute5
                MENU LABEL ^Compute 5 Install
                KERNEL ::boot/amd64/coreos/4.7.7/rhcos-live-kernel-x86_64
                INITRD ::boot/amd64/coreos/4.7.7/rhcos-live-initramfs.x86_64.img
                APPEND coreos.live.rootfs_url=http://192.168.1.10:8080/pub/4.7/images/rhcos-4.7.7-x86_64-live-rootfs.x86_64.img coreos.inst.install_dev=/dev/sda coreos.inst.ignition_url=http://192.168.1.10:8080/pub/4.7/ignitions/worker.ign ip=192.168.1.89::192.168.1.1:255.255.255.0:qbn-ocp4-compute-5 .qub593.cloud.qubinodeexample.com:ens192:none nameserver=192.168.1.150 
            MENU END

```


* boot new node using the pxe server
* approve csr and wait for node to configure
```
watch -n 10 oc get csr
oc get csr -ojson | jq -r '.items[] | select(.status == {} ) | .metadata.name' | xargs oc adm certificate approve
```


To-Do List / Known Limitations
------------------------------
>List of to-do's if anyone would like to contriube to the role
- ocp4-bootstrap-webserver does not need to download any coreos images
- Currently disconnected installation is not supported in OCP4/OKD4 role
- localstorage - Role has not been tested on OKD
- Role currently does not support IPI install on OCP or OKD.
- Role should be able to run on Fedora or Centos but has not been tested.

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
