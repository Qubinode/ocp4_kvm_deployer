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

Example Usage:

```
ansible-playbook rhcos.yml -t setup
ansible-playbook rhcos.yml -t tools
ansible-playbook rhcos.yml -t podman
ansible-playbook rhcos.yml -t podman --skip-tags pkg
ansible-playbook rhcos.yml -t ignitions
ansible-playbook rhcos.yml -t lb
ansible-playbook rhcos.yml -t webserver
ansible-playbook rhcos.yml -t download
ansible-playbook rhcos.yml -t libvirt_net
ansible-playbook rhcos.yml -t node_profile
ansible-playbook rhcos.yml -t idm
ansible-playbook rhcos.yml -t rhcos

ansible-playbook rhcos.yml -t webserver
ansible-playbook rhcos.yml -t lb
ansible-playbook rhcos.yml -t libvirt_net
ansible-playbook rhcos.yml -t deploy_vms
ansible-playbook rhcos.yml -t nfs --extra-vars "configure_nfs_storage=true" --extra-vars "cluster_deployed_msg=deployed
ansible-playbook rhcos.yml -t localstorage
```
Dependancy roles:
  - openshift-4-loadbalancer
  - nfs-provisioner-role

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
