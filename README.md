ansible-role-kubernetes-controller
==================================

This role is used in [Kubernetes the not so hard way with Ansible (at scaleway) - Part 6 - Control plane](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-6/). It installes the Kubernetes API server, scheduler and controller manager. For more information about this role please have a look at [Kubernetes the not so hard way with Ansible (at scaleway) - Part 6 - Control plane](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-6/).

Versions
--------

I tag every release and try to stay with [semantic versioning](http://semver.org). If you want to use the role I recommend to checkout the latest tag. The master branch is basically development while the tags mark stable releases. But in general I try to keep master in good shape too. A tag `r1.0.0_v1.8.0` means this is release 1.0.0 of this role and it's meant to be used with Kubernetes version 1.8.0. If the role itself changes `rX.Y.Z` will increase. If the Kubernetes version changes `vX.Y.Z` will increase. This allows to tag bugfixes and new major versions of the role while it's still developed for a specific Kubernetes release.

Requirements
------------

This role requires that you already created some certificates for Kubernetes API server (see [Kubernetes the not so hard way with Ansible (at Scaleway) - Part 4 - Certificate authority (CA)](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-4/)). The role copies the certificates from `k8s_ca_conf_directory` to the destination host.

Role Variables
--------------

```
k8s_conf_dir: "/var/lib/kubernetes"
k8s_bin_dir: "/usr/local/bin"
k8s_release: "1.8.0"
k8s_interface: "tap0"

k8s_ca_conf_directory: "/etc/k8s/certs"
k8s_config_directory: "/etc/k8s/configs"

k8s_controller_binaries:
  - kube-apiserver
  - kube-controller-manager
  - kube-scheduler
  - kubectl

k8s_certificates:
  - ca-k8s-apiserver.pem
  - ca-k8s-apiserver-key.pem
  - cert-k8s-apiserver.pem
  - cert-k8s-apiserver-key.pem

k8s_apiserver_admission_control: "Initializers,NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota"
k8s_apiserver_allow_privileged: "true"
k8s_apiserver_apiserver_count: "3"
k8s_apiserver_authorization_mode: "Node,RBAC"
k8s_apiserver_audit_log_maxage: "30"
k8s_apiserver_audit_log_maxbackup: "3"
k8s_apiserver_audit_log_maxsize: "100"
k8s_apiserver_audit_log_path: "/var/log/audit.log"
k8s_apiserver_enable_swagger_ui: "true"
k8s_apiserver_event_ttl: "1h"
k8s_apiserver_kubelet_https: "true"
k8s_apiserver_runtime_config: "api/all"
k8s_apiserver_service_cluster_ip_range: "10.32.0.0/16"
k8s_apiserver_service_node_port_range: "30000-32767"

k8s_controller_manager_cluster_cidr: "10.200.0.0/16"
k8s_controller_manager_cluser_name: "kubernetes"
k8s_controller_manager_leader_elect: "true"
k8s_controller_manager_conf_dir: "{{k8s_conf_dir}}"
k8s_controller_manager_service_cluster_ip_range: "{{k8s_apiserver_service_cluster_ip_range}}"

k8s_scheduler_leader_elect: "{{k8s_controller_manager_leader_elect}}"

etcd_client_port: "2379"
etcd_interface: "tap0"

etcd_certificates:
  - ca-etcd.pem
  - ca-etcd-key.pem
  - cert-etcd.pem
  - cert-etcd-key.pem
```

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
