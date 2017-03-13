ansible-role-kubernetes-controller
==================================

This playbook is used in [Kubernetes the not so hard way with Ansible (at scaleway) - Part 6 - Control plane](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-6/). It installes the Kubernetes API server, scheduler and controller manager.

Requirements
------------

This playbook requires that you already created some certificates for Kubernetes API server (see [Kubernetes the not so hard way with Ansible (at Scaleway) - Part 4 - Certificate authority (CA)](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-4/)). The playbook copies the certificates from `local_cert_dir` on the host this playbook runs to the destination host.

Role Variables
--------------

```
local_cert_dir: /etc/cfssl

etcd_client_port: 2379
etcd_interface: tap0

etcd_certificates:
  - ca-etcd.pem
  - ca-etcd-key.pem
  - cert-etcd.pem
  - cert-etcd-key.pem

k8s_conf_dir: /var/lib/kubernetes
k8s_bin_dir: /usr/local/bin
k8s_release: 1.5.1
k8s_interface: tap0
k8s_certificates:
  - ca-k8s-apiserver.pem
  - ca-k8s-apiserver-key.pem
  - cert-k8s-apiserver.pem
  - cert-k8s-apiserver-key.pem
k8s_binaries:
  - kube-apiserver
  - kube-controller-manager
  - kube-scheduler
  - kubectl
k8s_auth_tokens:
  - chAng3m3,admin,admin
  - chAng3m3,scheduler,scheduler
  - chAng3m3,kubelet,kubelet
```

Example Playbook
----------------

```
- hosts: k8s-controller
  roles:
    - githubixx.kubernetes-controller
```

License
-------

GNU GENERAL PUBLIC LICENSE Version 3

Author Information
------------------

[http://www.tauceti.blog](http://www.tauceti.blog)
