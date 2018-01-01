ansible-role-kubernetes-controller
==================================

This role is used in [Kubernetes the not so hard way with Ansible (at scaleway) - Part 6 - Control plane](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-6/). It installes the Kubernetes API server, scheduler and controller manager. For more information about this role please have a look at [Kubernetes the not so hard way with Ansible (at scaleway) - Part 6 - Control plane](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-6/).

Versions
--------

I tag every release and try to stay with [semantic versioning](http://semver.org). If you want to use the role I recommend to checkout the latest tag. The master branch is basically development while the tags mark stable releases. But in general I try to keep master in good shape too. A tag `r1.0.0_v1.8.0` means this is release 1.0.0 of this role and it's meant to be used with Kubernetes version 1.8.0. If the role itself changes `rX.Y.Z` will increase. If the Kubernetes version changes `vX.Y.Z` will increase. This allows to tag bugfixes and new major versions of the role while it's still developed for a specific Kubernetes release.

Requirements
------------

This role requires that you already created some certificates for Kubernetes API server (see [Kubernetes the not so hard way with Ansible (at Scaleway) - Part 4 - Certificate authority (CA)](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-4/)). The role copies the certificates from `k8s_ca_conf_directory` to the destination host. You should also setup PeerVPN (see [Kubernetes the not so hard way with Ansible (at Scaleway) - Part 3 - Peervpn](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-3/) and of course a etcd cluster (see [Kubernetes the not so hard way with Ansible (at Scaleway) - Part 5 - etcd cluster](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-5/)

Role Variables
--------------

```
# The directory to store the K8s certificates and other configuration
k8s_conf_dir: "/var/lib/kubernetes"
# The directory to store the K8s binaries
k8s_bin_dir: "/usr/local/bin"
# K8s release
k8s_release: "1.8.4"
# The interface on which the K8s services should listen on. As all cluster
# communication should use the PeerVPN interface the interface name is
# normally "tap0" or "peervpn0".
k8s_interface: "tap0"

# The directory from where to copy the K8s certificates (needs to be
# accessable from the host you run Ansible on)
k8s_ca_conf_directory: "/etc/k8s/certs"
# Directory where kubeconfig for Kubernetes worker nodes and kube-proxy
# is stored among other configuration files.
k8s_config_directory: "/etc/k8s/configs"

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

# kube-apiserver settings (can be overriden or additional added by defining
# "k8s_apiserver_settings_user" - see text below)
k8s_apiserver_settings:
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
  "cluster-cidr": "10.200.0.0/16"
  "cluster-name": "kubernetes"
  "leader-elect": "true"
  "service-cluster-ip-range": "10.32.0.0/16"
  "cluster-signing-cert-file": "{{k8s_controller_manager_conf_dir}}/ca-k8s-apiserver.pem"
  "cluster-signing-key-file": "{{k8s_controller_manager_conf_dir}}/cert-k8s-apiserver-key.pem"
  "root-ca-file": "{{k8s_controller_manager_conf_dir}}/ca-k8s-apiserver.pem"
  "cluster-signing-cert-file": "{{k8s_controller_manager_conf_dir}}/ca-k8s-apiserver.pem"
  "service-account-private-key-file": "{{k8s_controller_manager_conf_dir}}/cert-k8s-apiserver-key.pem"

# kube-scheduler settings
k8s_scheduler_leader_elect: "{{k8s_controller_manager_leader_elect}}"

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

The kube-apiserver settings defined in `k8s_apiserver_settings` can be overriden by defining a variable called `k8s_apiserver_settings_user`. You can alse add additional settings by using this variable. E.g. to override `audit-log-maxage` and `audit-log-maxbackup` default values and add `watch-cache` add the following settings to `group_vars/k8s.yml`:

```
k8s_apiserver_settings_user:
  "audit-log-maxage": "40"
  "audit-log-maxbackup": "4"
  "watch-cache": "false"
```

The same is true for the `kube-controller-manager` by changing `k8s_controller_manager_settings` and `k8s_controller_manager_settings_user` variables.

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
