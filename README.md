ansible-role-kubernetes-controller
==================================

This role is used in [Kubernetes the not so hard way with Ansible - Control plane](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-control-plane/). It installes the Kubernetes API server, scheduler and controller manager. For more information about this role please have a look at [Kubernetes the not so hard way with Ansible - Control plane](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-control-plane/).

Versions
--------

I tag every release and try to stay with [semantic versioning](http://semver.org). If you want to use the role I recommend to checkout the latest tag. The master branch is basically development while the tags mark stable releases. But in general I try to keep master in good shape too. A tag `8.0.0+1.14.2` means this is release `8.0.0` of this role and it's meant to be used with Kubernetes version `1.14.2` (but should work with any K8s 1.14.x release of course). If the role itself changes `X.Y.Z` before `+` will increase. If the Kubernetes version changes `X.Y.Z` after `+` will increase too. This allows to tag bugfixes and new major versions of the role while it's still developed for a specific Kubernetes release. That's especially useful for Kubernetes major releases with breaking changes.

Requirements
------------

This role requires that you already created some certificates for Kubernetes API server (see [Kubernetes the not so hard way with Ansible - Certificate authority (CA)](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-certificate-authority/)). The role copies the certificates from `k8s_ca_conf_directory` to the destination host. You should also setup a fully meshed VPN with e.g. WireGuard (see [Kubernetes the not so hard way with Ansible - WireGuard](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-wireguard/) and of course a etcd cluster (see [Kubernetes the not so hard way with Ansible - etcd cluster](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-etcd/)

Changelog
---------

see [CHANGELOG.md](https://github.com/githubixx/ansible-role-kubernetes-controller/blob/master/CHANGELOG.md)

Role (default) variables
------------------------

```
# The directory to store the K8s certificates and other configuration
k8s_conf_dir: "/var/lib/kubernetes"
# The directory to store the K8s binaries
k8s_bin_dir: "/usr/local/bin"
# K8s release
k8s_release: "1.16.8"
# The interface on which the K8s services should listen on. As all cluster
# communication should use a VPN interface the interface name is
# normally "wg0" (WireGuard),"peervpn0" (PeerVPN) or "tap0".
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

# K8s kube-(apiserver|controller-manager-sa) certificates
k8s_certificates:
  - ca-k8s-apiserver.pem
  - ca-k8s-apiserver-key.pem
  - cert-k8s-apiserver.pem
  - cert-k8s-apiserver-key.pem
  - cert-k8s-controller-manager-sa.pem
  - cert-k8s-controller-manager-sa-key.pem

k8s_apiserver_secure_port: "6443"

# K8s API daemon settings (can be overriden or additional added by defining
# "k8s_apiserver_settings_user")
k8s_apiserver_settings:
  "advertise-address": "{{hostvars[inventory_hostname]['ansible_' + k8s_interface].ipv4.address}}"
  "bind-address": "{{hostvars[inventory_hostname]['ansible_' + k8s_interface].ipv4.address}}"
  "secure-port": "{{k8s_apiserver_secure_port}}"
  "enable-admission-plugins": "NodeRestriction,NamespaceLifecycle,LimitRanger,ServiceAccount,TaintNodesByCondition,Priority,DefaultTolerationSeconds,DefaultStorageClass,PersistentVolumeClaimResize,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota"
  "allow-privileged": "true"
  "apiserver-count": "3"
  "authorization-mode": "Node,RBAC"
  "audit-log-maxage": "30"
  "audit-log-maxbackup": "3"
  "audit-log-maxsize": "100"
  "audit-log-path": "/var/log/audit.log"
  "event-ttl": "1h"
  "kubelet-https": "true"
  "kubelet-preferred-address-types": "InternalIP,Hostname,ExternalIP" # "--kubelet-preferred-address-types" defaults to:
                                                                      # "Hostname,InternalDNS,InternalIP,ExternalDNS,ExternalIP"
                                                                      # Needs to be changed to make "kubectl logs" and "kubectl exec" work.
  "runtime-config": "api/all"
  "service-cluster-ip-range": "10.32.0.0/16"
  "service-node-port-range": "30000-32767"
  "client-ca-file": "{{k8s_conf_dir}}/ca-k8s-apiserver.pem"
  "etcd-cafile": "{{k8s_conf_dir}}/ca-etcd.pem"
  "etcd-certfile": "{{k8s_conf_dir}}/cert-etcd.pem"
  "etcd-keyfile": "{{k8s_conf_dir}}/cert-etcd-key.pem"
  "encryption-provider-config": "{{k8s_conf_dir}}/encryption-config.yaml"
  "kubelet-certificate-authority": "{{k8s_conf_dir}}/ca-k8s-apiserver.pem"
  "kubelet-client-certificate": "{{k8s_conf_dir}}/cert-k8s-apiserver.pem"
  "kubelet-client-key": "{{k8s_conf_dir}}/cert-k8s-apiserver-key.pem"
  "service-account-key-file": "{{k8s_conf_dir}}/cert-k8s-controller-manager-sa.pem"
  "tls-cert-file": "{{k8s_conf_dir}}/cert-k8s-apiserver.pem"
  "tls-private-key-file": "{{k8s_conf_dir}}/cert-k8s-apiserver-key.pem"

# The directory to store controller manager configuration.
k8s_controller_manager_conf_dir: "/var/lib/kube-controller-manager"
# K8s controller manager settings (can be overriden or additional added by defining
# "k8s_controller_manager_settings_user")
k8s_controller_manager_settings:
  "bind-address": "{{hostvars[inventory_hostname]['ansible_' + k8s_interface].ipv4.address}}"
  "port": "0"
  "cluster-cidr": "10.200.0.0/16"
  "cluster-name": "kubernetes"
  "kubeconfig": "{{k8s_controller_manager_conf_dir}}/kube-controller-manager.kubeconfig"
  "leader-elect": "true"
  "service-cluster-ip-range": "10.32.0.0/16"
  "cluster-signing-cert-file": "{{k8s_conf_dir}}/ca-k8s-apiserver.pem"
  "cluster-signing-key-file": "{{k8s_conf_dir}}/cert-k8s-apiserver-key.pem"
  "root-ca-file": "{{k8s_conf_dir}}/ca-k8s-apiserver.pem"
  "service-account-private-key-file": "{{k8s_conf_dir}}/cert-k8s-controller-manager-sa-key.pem"
  "use-service-account-credentials": "true"

# The directory to store scheduler configuration.
k8s_scheduler_conf_dir: "/var/lib/kube-scheduler"
# kube-scheduler settings (only --config left,
# see https://github.com/kubernetes/kubernetes/pull/62515)
k8s_scheduler_settings:
  "bind-address": "{{hostvars[inventory_hostname]['ansible_' + k8s_interface].ipv4.address}}"
  "config": "{{k8s_scheduler_conf_dir}}/kube-scheduler.yaml"

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
