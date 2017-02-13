ansible-role-kubernetes-controller
==================================

This playbook installes the Kubernetes API server, scheduler and controller manager. 

Requirements
------------

This playbook requires that you already created some certificates for kubernetes-controller (see [ansible-role-cfssl](https://github.com/githubixx/ansible-role-cfssl)). The playbook searches the certificates in `local_cert_dir` on the host this playbook runs.

Role Variables
--------------

```
local_cert_dir: /etc/cfssl

etcd_client_port: 2379
etcd_interface: tap0

k8s_conf_dir: /var/lib/kubernetes
k8s_bin_dir: /usr/local/bin
k8s_release: 1.5.1
k8s_interface: tap0
k8s_certificates:
  - ca.pem
  - kubernetes-key.pem
  - kubernetes.pem
k8s_binaries:
  - kube-apiserver
  - kube-controller-manager
  - kube-scheduler
  - kubectl
```

Example Playbook
----------------

```
- hosts: kube-controller

  roles:
    - githubixx.kube-controller
```

License
-------

GNU GENERAL PUBLIC LICENSE Version 3

Author Information
------------------

[http://www.tauceti.blog](http://www.tauceti.blog)
