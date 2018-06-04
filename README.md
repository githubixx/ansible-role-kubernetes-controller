ansible-role-kubernetes-controller
==================================

This role is used in [Kubernetes the not so hard way with Ansible (at scaleway) - Part 6 - Control plane](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-6/). It installes the Kubernetes API server, scheduler and controller manager. For more information about this role please have a look at [Kubernetes the not so hard way with Ansible (at scaleway) - Part 6 - Control plane](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-6/).

Versions
--------

I tag every release and try to stay with [semantic versioning](http://semver.org) (well kind of...). If you want to use the role I recommend to checkout the latest tag. The master branch is basically development while the tags mark stable releases. But in general I try to keep master in good shape too. A tag `r1.0.0_v1.8.0` means this is release 1.0.0 of this role and it's meant to be used with Kubernetes version 1.8.0 (but will work with any 1.8.x release of course). If the role itself changes `rX.Y.Z` will increase. If the Kubernetes version changes `vX.Y.Z` will increase. This allows to tag bugfixes and new major versions of the role while it's still developed for a specific Kubernetes release. That's especially useful for Kubernetes major releases with breaking changes.

Requirements
------------

This role requires that you already created some certificates for Kubernetes API server (see [Kubernetes the not so hard way with Ansible (at Scaleway) - Part 4 - Certificate authority (CA)](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-4/)). The role copies the certificates from `k8s_ca_conf_directory` to the destination host. You should also setup PeerVPN (see [Kubernetes the not so hard way with Ansible (at Scaleway) - Part 3 - Peervpn](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-3/) and of course a etcd cluster (see [Kubernetes the not so hard way with Ansible (at Scaleway) - Part 5 - etcd cluster](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-5/)

Changelog
---------

**r4.0.0_v1.10.3**

- update `k8s_release` to `1.10.3`
- removed deprecated kube-apiserver parameter `insecure-bind-address` (see: [#59018](https://github.com/kubernetes/kubernetes/pull/59018))
- added variable `k8s_apiserver_secure_port: 6443`
- added parameter `secure-port` to `k8s_apiserver_settings` parameter list
- added kube-scheduler/kube-controller-manager certificate files to k8s_certificates list

**r3.0.0_v1.9.8**

- update `k8s_release` to `1.9.8`

**r3.0.0_v1.9.3**

- update `k8s_release` to `1.9.3`

**r3.0.0_v1.9.1**

- move advertise-address,bind-address,insecure-bind-address out of kube-apiserver.service.j2 template
- move address,master settings out of kube-controller-manager.service.j2 template / fix variable bug in `k8s_apiserver_settings`
- move address,master settings out of kube-scheduler.service.j2 template
- fix: use `k8s_etcd` hosts group instead of `k8s_controller` group to generate etcd server list
- we need to wait for kube-apiserver port 8080 to become ready before running kubectl tasks

**r2.0.2_v1.9.1**

- update to Kubernetes v1.9.1

**r2.0.1_v1.9.0**

- removed duplicate key cluster-signing-cert-file from `k8s_controller_manager_settings` dictionary

**r2.0.0_v1.9.0**

- introduce flexible parameter settings for API server via `k8s_apiserver_settings/k8s_apiserver_settings_user`
- introduce flexible parameter settings for controller manager via `k8s_controller_manager_settings/k8s_controller_manager_settings_user`
- introduce flexible parameter settings for kube-scheduler via `k8s_scheduler_settings/k8s_scheduler_settings_user`
- change defaults for `k8s_ca_conf_directory` and `k8s_config_directory` variables
- update to Kubernetes v1.9.0


No changelog for releases < r2.0.0_v1.9.0 (see commit history if needed)


Role (default) variables
------------------------

```
# The directory to store the K8s certificates and other configuration
k8s_conf_dir: "/var/lib/kubernetes"
# The directory to store the K8s binaries
k8s_bin_dir: "/usr/local/bin"
# K8s release
k8s_release: "1.9.8"
# The interface on which the K8s services should listen on. As all cluster
# communication should use the PeerVPN interface the interface name is
# normally "tap0" or "peervpn0".
k8s_interface: "tap0"

# The directory from where to copy the K8s certificates. By default this
# will expand to user's LOCAL $HOME (the user that run's "ansible-playbook ..."
# plus "/k8s/certs". That means if the user's $HOME directory is e.g.
# "/home/da_user" then "k8s_ca_conf_directory" will have a value of
# "/home/da_user/k8s/certs".
k8s_ca_conf_directory: "{{ '~/k8s/certs' | expanduser }}"
# Directory where kubeconfig for Kubernetes worker nodes and kube-proxy
# is stored among other configuration files. Same variable expansion
# rule applies as with "k8s_ca_conf_directory"
k8s_config_directory: "{{ '~/k8s/configs' | expanduser }}"

# K8s control plane binaries to download
k8s_controller_binaries:
  - kube-apiserver
  - kube-controller-manager
  - kube-scheduler
  - kubectl

# K8s API daemon certificates
k8s_certificates:
  - ca-k8s-apiserver.pem
  - ca-k8s-apiserver-key.pem
  - cert-k8s-apiserver.pem
  - cert-k8s-apiserver-key.pem
  - cert-k8s-controller-manager.pem
  - cert-k8s-controller-manager-key.pem
  - cert-k8s-scheduler.pem
  - cert-k8s-scheduler-key.pem

k8s_apiserver_secure_port: "6443"

# kube-apiserver settings (can be overriden or additional added by defining
# "k8s_apiserver_settings_user" - see text below)
k8s_apiserver_settings:
  "advertise-address": "hostvars[inventory_hostname]['ansible_' + k8s_interface].ipv4.address"
  "bind-address": "hostvars[inventory_hostname]['ansible_' + k8s_interface].ipv4.address"
  "secure-port": "{{k8s_apiserver_secure_port}}"
  "admission-control": "Initializers,NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota"
  "allow-privileged": "true"
  "apiserver-count": "3"
  "authorization-mode": "Node,RBAC"
  "audit-log-maxage": "30"
  "audit-log-maxbackup": "3"
  "audit-log-maxsize": "100"
  "audit-log-path": "/var/log/audit.log"
  "enable-swagger-ui": "true"
  "event-ttl": "1h"
  "kubelet-https": "true"
  "kubelet-preferred-address-types": "InternalIP,Hostname,ExternalIP"
  "runtime-config": "api/all"
  "service-cluster-ip-range": "10.32.0.0/16"
  "service-node-port-range": "30000-32767"
  "client-ca-file": "{{k8s_conf_dir}}/ca-k8s-apiserver.pem"
  "etcd-cafile": "{{k8s_conf_dir}}/ca-etcd.pem"
  "etcd-certfile": "{{k8s_conf_dir}}/cert-etcd.pem"
  "etcd-keyfile": "{{k8s_conf_dir}}/cert-etcd-key.pem"
  "experimental-encryption-provider-config": "{{k8s_conf_dir}}/encryption-config.yaml"
  "kubelet-certificate-authority": "{{k8s_conf_dir}}/ca-k8s-apiserver.pem"
  "kubelet-client-certificate": "{{k8s_conf_dir}}/cert-k8s-apiserver.pem"
  "kubelet-client-key": "{{k8s_conf_dir}}/cert-k8s-apiserver-key.pem"
  "service-account-key-file": "{{k8s_conf_dir}}/cert-k8s-apiserver-key.pem"
  "tls-ca-file": "{{k8s_conf_dir}}/ca-k8s-apiserver.pem"
  "tls-cert-file": "{{k8s_conf_dir}}/cert-k8s-apiserver.pem"
  "tls-private-key-file": "{{k8s_conf_dir}}/cert-k8s-apiserver-key.pem"

# The directory to store controller manager configuration.
k8s_controller_manager_conf_dir: "{{k8s_conf_dir}}"

# kube-controller-manager settings (can be overriden or additional added by defining
# "k8s_controller_manager_settings_user" - see text below)
k8s_controller_manager_settings:
  "address": "{{hostvars[inventory_hostname]['ansible_' + k8s_interface].ipv4.address}}"
  "master": "{{'http://' + hostvars[inventory_hostname]['ansible_' + k8s_interface].ipv4.address + ':8080'}}"
  "cluster-cidr": "10.200.0.0/16"
  "cluster-name": "kubernetes"
  "leader-elect": "true"
  "service-cluster-ip-range": "10.32.0.0/16"
  "cluster-signing-cert-file": "{{k8s_controller_manager_conf_dir}}/ca-k8s-apiserver.pem"
  "cluster-signing-key-file": "{{k8s_controller_manager_conf_dir}}/cert-k8s-apiserver-key.pem"
  "root-ca-file": "{{k8s_controller_manager_conf_dir}}/ca-k8s-apiserver.pem"
  "cluster-signing-cert-file": "{{k8s_controller_manager_conf_dir}}/ca-k8s-apiserver.pem"
  "service-account-private-key-file": "{{k8s_controller_manager_conf_dir}}/cert-k8s-apiserver-key.pem"

# kube-scheduler settings (can be overriden or additional added by defining
# "k8s_scheduler_settings_user" - see text below)
k8s_scheduler_settings:
  "address": "{{hostvars[inventory_hostname]['ansible_' + k8s_interface].ipv4.address}}"
  "master": "{{'http://' + hostvars[inventory_hostname]['ansible_' + k8s_interface].ipv4.address + ':8080'}}"
  "leader-elect": "true"

# The port the control plane componentes should connect to etcd cluster
etcd_client_port: "2379"
# The interface the etcd cluster is listening on
etcd_interface: "tap0"

# The etcd certificates needed for the control plane componentes to be able
# to connect to the etcd cluster.
etcd_certificates:
  - ca-etcd.pem
  - ca-etcd-key.pem
  - cert-etcd.pem
  - cert-etcd-key.pem
```

The kube-apiserver settings defined in `k8s_apiserver_settings` can be overriden by defining a variable called `k8s_apiserver_settings_user`. You can also add additional settings by using this variable. E.g. to override `audit-log-maxage` and `audit-log-maxbackup` default values and add `watch-cache` add the following settings to `group_vars/k8s.yml`:

```
k8s_apiserver_settings_user:
  "audit-log-maxage": "40"
  "audit-log-maxbackup": "4"
  "watch-cache": "false"
```

The same is true for the `kube-controller-manager` by adding entries to `k8s_controller_manager_settings_user` variable. For `kube-scheduler` add entries to `k8s_scheduler_settings_user` variable to override/add settings in `k8s_scheduler_settings` dictionary.

Example Playbook
----------------

```
- hosts: k8s_controller
  roles:
    - githubixx.kubernetes-controller
```

License
-------

GNU GENERAL PUBLIC LICENSE Version 3

Author Information
------------------

[http://www.tauceti.blog](http://www.tauceti.blog)
