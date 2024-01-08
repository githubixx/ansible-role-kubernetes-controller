# ansible-role-kubernetes-controller

This role is used in [Kubernetes the not so hard way with Ansible - Control plane](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-control-plane/). It installs the Kubernetes API server, scheduler and controller manager. For more information about this role please have a look at [Kubernetes the not so hard way with Ansible - Control plane](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-control-plane/).

## Versions

I tag every release and try to stay with [semantic versioning](http://semver.org). If you want to use the role I recommend to checkout the latest tag. The master branch is basically development while the tags mark stable releases. But in general I try to keep master in good shape too. A tag `21.0.0+1.27.8` means this is release `21.0.0` of this role and it's meant to be used with Kubernetes version `1.27.8` (but should work with any K8s 1.27.x release of course). If the role itself changes `X.Y.Z` before `+` will increase. If the Kubernetes version changes `X.Y.Z` after `+` will increase too. This allows to tag bugfixes and new major versions of the role while it's still developed for a specific Kubernetes release. That's especially useful for Kubernetes major releases with breaking changes.

## Requirements

This role requires that you already created some certificates for Kubernetes API server (see [Kubernetes the not so hard way with Ansible - Certificate authority (CA)](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-certificate-authority/)). The role copies the certificates from `k8s_ctl_ca_conf_directory` (which is by default the same as `k8s_ca_conf_directory` used by `githubixx.kubernetes_ca` role) to the destination host. You should also setup a fully meshed VPN with e.g. WireGuard (see [Kubernetes the not so hard way with Ansible - WireGuard](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-wireguard/)) and of course an etcd cluster (see [Kubernetes the not so hard way with Ansible - etcd cluster](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-etcd/)). The WireGuard VPN Mesh is not a requirement but increases security as all traffic between the K8s hosts is encrypted by default. But as long as all hosts included have an interface where they can communicate with each other it's fine.

## Supported OS

- Ubuntu 20.04 (Focal Fossa)
- Ubuntu 22.04 (Jammy Jellyfish)

## Changelog

**Change history:**

See full [CHANGELOG.md](https://github.com/githubixx/ansible-role-kubernetes-controller/blob/master/CHANGELOG.md)

**Recent changes:**

## 23.0.0+1.27.5

- update `k8s_release` to `1.28.5`

## 22.0.0+1.27.8

### PLEASE READ CAREFULLY

This release contains quite a few potential breaking changes! So review carefully before rolling out the new version of this role! A bigger part of the whole changes are related to increase security. While most of the new variables
 and defaults should be just fine and should just work out of the box side effects might occur.

All the newly introduced or changed variables have detailed comments in [README](https://github.com/githubixx/ansible-role-kubernetes-controller/blob/master/README.md). So please read them carefully!

This refactoring was needed to make it possible to have `githubixx.kubernetes_controller` and `githubixx.kubernetes_worker` deployed on the same host e.g. They were some intersections between the two roles that had to be fixed.

**Please remove** `/var/lib/kubernetes/admin.kubeconfig` on the K8s controller nodes (if you didn't change the default directory for this file). Older versions of this role created this file. It's no longer needed. It contains the
`kubeconfig` (so basically the credentials file) for the `admin` user. This is a very powerful user (actually the user with the most permissions). So use with care and store the file in a secure place! `admin.kubeconfig` should onl
y be used at the very beginning to create a new user with less permissions.

### UPDATE

- update `k8s_release` to `1.27.8`

### BREAKING

- Rename variable `k8s_conf_dir` to `k8s_ctl_conf_dir`. Additionally the default value changed from `/usr/lib/kubernetes` to `/etc/kubernetes/controller`.
- Introduce variable `k8s_admin_conf_dir`. Currently it only stores `admin.kubeconfig` which is basically the credentials file of the `admin` "user". Formerly this file was stored in the directory specified in the (now removed) `k8
s_config_directory` variable. The default value of `k8s_admin_conf_dir` is the same as the removed `k8s_config_directory`. Additionally to set permissions for `k8s_admin_conf_dir` the following variables were introduced: `k8s_admin
_conf_dir_perm`, `k8s_admin_conf_owner` and `k8s_admin_conf_group`.
- Introduce variable `k8s_ctl_pki_dir`. All certificate files specified in `k8s_ctl_certificates` and `k8s_ctl_etcd_certificates` (see `vars/main.yml`) will be stored here. Related to this: Certificate related settings in `k8s_apiserver_settings` used `k8s_conf_dir` before and now use `k8s_ctl_pki_dir`. That's `client-ca-file`, `etcd-cafile`, `etcd-certfile`, `etcd-keyfile`, `kubelet-certificate-authority`, `kubelet-client-certificate`, `kubelet-client-key`,
 `service-account-key-file`, `service-account-signing-key-file`, `tls-cert-file` and `tls-private-key-file`. For `k8s_controller_manager_settings` that's: `cluster-signing-cert-file`, `cluster-signing-key-file`, `root-ca-file`, `re
questheader-client-ca-file` and `service-account-private-key-file`. And for `k8s_scheduler_settings` that's: `requestheader-client-ca-file`.
- Rename variable `k8s_bin_dir` to `k8s_ctl_bin_dir`.
- Rename variable `k8s_release` to `k8s_ctl_release`.
- The default value for `k8s_interface` changed from `tap0` to `eth0`.
- Rename variable `k8s_controller_binaries` to `k8s_ctl_binaries`. Additionally this variable is no longer defined in `defaults/main.yml` but in `vars/main.yml`. Since this list is fixed anyways it makes no sense to allow to modify
 this list.
- Rename variable `k8s_certificates` to `k8s_ctl_certificates`.
- Rename variable `etcd_certificates` to `k8s_ctl_etcd_certificates`. Additionally this variable is no longer defined in `defaults/main.yml` but in `vars/main.yml`. Since this list is fixed anyways it makes no sense to allow to mod
ify this list.
- Rename variable `etcd_client_port` to `k8s_ctl_etcd_client_port`.
- Rename variable `etcd_interface` to `k8s_ctl_etcd_interface`. Additionally the default value changed from `tap0` to `eth0`.
- Use `k8s_ctl_etcd_interface` variable instead of `k8s_interface` for `--etcd-servers` option in `templates/etc/systemd/system/kube-apiserver.service.j2`. Normally `etcd` and `kube-apiserver` listen on the same interface. But if someone specified `k8s_ctl_etcd_interface` (formerly `etcd_interface` in the context of this role) it was basically ignored as the value of `k8s_interface` was used instead. That's fixed now.
- Rename variable `k8s_controller_delegate_to` to `k8s_ctl_delegate_to`.
- Introduce `k8s_apiserver_conf_dir` variable. `encryption-config.yaml` is now located in `k8s_apiserver_conf_dir`.
- Change default value of `k8s_controller_manager_conf_dir` to `"{{ k8s_ctl_conf_dir }}/kube-controller-manager"`.
- Change default value of `k8s_scheduler_conf_dir` to `"{{ k8s_ctl_conf_dir }}/kube-scheduler"`.
- Rename `kube-controller-manager.kubeconfig` to `kubeconfig`. This affects `k8s_controller_manager_settings` and the following settings: `authentication-kubeconfig`, `authorization-kubeconfig` and `kubeconfig`.
- Rename `kube-scheduler.kubeconfig` to `kubeconfig`. This affects `k8s_scheduler_settings` and the following settings: `authentication-kubeconfig"` and `authorization-kubeconfig`.
- Added new option `encryption-provider-config-automatic-reload: "true"` to `k8s_apiserver_settings`. In case the file specified in `encryption-provider-config` changes `kube-apiserver` will automatically reload that file. This is
handy if one wants to change the encryption provider (also see new variable `k8s_apiserver_encryption_provider_config`)
- Introduce `k8s_ctl_service_options` variable. As mentioned above already previously `kube-apiserver`, `kube-controller-manager` and `kube-scheduler` were running as user `root`. Now these services will run as `k8s_run_as_user` an
d `k8s_run_as_group`. Additionally `systemd` allows to limit the exposure of the system towards the unit's processes. Basically all settings below `RestartSec=5` are related to increase security and limit what the process allowed t
o do. So these settings reduce the attack surface quite a bit already. They're not perfect but a starting point. If you want the previous behavior just remove all settings besides `Restart` and `RestartSec`. But that also means tha
t they'll run again as `root` user.
- The following variables are no longer used and can be removed: `k8s_encryption_config_directory`, `k8s_encryption_config_owner`, `k8s_encryption_config_group`, `k8s_encryption_config_directory_perm` and `k8s_encryption_config_fil
e_perm`. Previously these values were needed to specify permissions on the Ansible controller for this file/directory. Since this file is now generated directly on the K8s controller nodes they're no longer needed. That also means
you can remove `encryption-config.yaml` from Ansible controller node (like your workstation e.g.)
- The variable `k8s_config_directory` is gone. It's no longer in use. After the upgrade to this release you can delete this directory (if you accept the new default!) and it's content (make a backup esp. of `admin.kubeconfig` file
- just in case!)
- Rename `k8s_ca_conf_directory` to `k8s_ctl_ca_conf_directory`

### FEATURE

- Introduce `k8s_run_as_user` variable. Previously all control plane services like `kube-apiserver`, `kube-scheduler` and `kube-controller-manager` run as user `root`. Securitywise that's not optimal. There is just no need to run t
hem as `root` as long they use a listening port > `1024` which they do by default. In this version all these services will run as the user specified with `k8s_run_as_user` which is `k8s` by default. Related to this variable are the
 new variables `k8s_run_as_user_shell`, `k8s_run_as_user_system`, `k8s_run_as_group` and `k8s_run_as_group_system`. See [README](https://github.com/githubixx/ansible-role-kubernetes-controller/blob/master/README.md) for further information about this variables. The defaults should be just fine even for upgrading from a previous version of this role.
- Introduce `k8s_ctl_api_endpoint_host` and `k8s_ctl_api_endpoint_port` variables. Previously `kube-scheduler` and `kube-controller-manager` where configured to connect to the first host in the Ansible `k8s_controller` group and communicate with the `kube-apiserver` that was running there. This was hard-coded and couldn't be changed. If that host was down the K8s worker nodes didn't receive any updates. Now one can install and use a load balancer like `haproxy` e.g. that distributes requests between all `kube-apiserver`'s and takes a `kube-apiserver` out of rotation if that one is down (also see my Ansible [haproxy role](https://github.com/githubixx/ansible-role-haproxy) for that use
case). The default is still to use the first host/kube-apiserver in the Ansible `k8s_controller` group. So behaviorwise nothing changed basically.
- Introduce `k8s_admin_api_endpoint_host` and `k8s_admin_api_endpoint_port` variables. For these two variables the same is basically true as for `k8s_ctl_api_endpoint_host` and `k8s_ctl_api_endpoint_port` variables above. But these
 settings are meant to be used by the `admin` user that this role creates by default. These settings are written into `admin.kubeconfig`. So it's possible to configure another host/load balancer for the `admin` user as for the K8s
control plane services mentioned in the previous paragraph.
- Introduce `k8s_ctl_log_base_dir` and `k8s_ctl_log_base_dir_mode`. Normally  `kube-apiserver`, `kube-controller-manager` and `kube-scheduler` log to `journald`. But there are exceptions like the audit log. For this kind of log fil
es this directory will be used as a base path.
- Introduce `k8s_apiserver_audit_log_dir`. Directory to store kube-apiserver audit logs.
- Introduce `k8s_apiserver_encryption_provider_config` variable. Previously the content of this file was hard-coded. Now it's exposed via this variable. The content of that variable and the previously hard-coded value are the same.
 So if you keep the default when upgrading everything stays the same in that regards. **NOTE**: Changing this configuration and deploy the changes can potentially cause quite some problems! Make sure to read [Encrypting Confidential Data at Rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/) and esp. [Rotating a decryption key](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#rotating-a-decryption-key)!
- Add task to generate `kubeconfig` for `admin` user (previously this was a separate [playbook](https://github.com/githubixx/ansible-kubernetes-playbooks/tree/v15.0.0_r1.27.5/kubeauthconfig)).
- Add task to generate `kubeconfig` for `kube-controller-manager` service (previously this was a separate [playbook](https://github.com/githubixx/ansible-kubernetes-playbooks/tree/v15.0.0_r1.27.5/kubeauthconfig)).
- Add task to generate `kubeconfig` for `kube-scheduler` service (previously this was a separate [playbook](https://github.com/githubixx/ansible-kubernetes-playbooks/tree/v15.0.0_r1.27.5/kubeauthconfig)).
- When downloading the Kubernetes binaries the task checks the SHA512 checksum
- Make `kube-scheduler` and `kube-controller-manager` wait until `kube-apiserver` has started and is listening on a port

### OTHER CHANGES

- Extend `k8s_ctl_certificates` (formerly `k8s_certificates`) list by `cert-k8s-scheduler` and `cert-k8s-controller-manager` files. This was needed as the `kubeconfig` files are now generated on the K8s controller nodes and no long
er on the Ansible controller host. Previously it was needed to prepare these files upfront before installing this role. That's no longer needed. Also see **FEATURES** list above.
- Use `kubernetes.core.*` modules instead of `kubectl` binary
- Fix some `ansible-lint` issues

### MOLECULE

- Updated all files to reflect the changes introduces with this version
- Tasks for creating `kubeconfig` for `kube-controller-manager`, `kube-scheduler`, `admin` user and `encryption configuration` are no longer needed as they're now part of `kubernetes_controller` role
- Add `haproxy` to Ubuntu 22 hosts to test new `k8s_ctl_api_endpoint_host` and `k8s_ctl_api_endpoint_port` settings
- Add tasks to install [ansible-role-cni](https://github.com/githubixx/ansible-role-cni) and [ansible-role-runc](https://github.com/githubixx/ansible-role-runc)
- Use `kubernetes.core.k8s_info` module instead of calling `kubectl` binary

21.1.3+1.27.5

- rename `githubixx.harden-linux` to `githubixx.harden_linux`

## Installation

- Directly download from Github (Change into Ansible role directory before cloning. You can figure out the role path by using `ansible-config dump | grep DEFAULT_ROLES_PATH` command):
`git clone https://github.com/githubixx/ansible-role-kubernetes-controller.git githubixx.kubernetes_controller`

- Via `ansible-galaxy` command and download directly from Ansible Galaxy:
`ansible-galaxy install role githubixx.kubernetes_controller`

- Create a `requirements.yml` file with the following content (this will download the role from Github) and install with
`ansible-galaxy role install -r requirements.yml` (change `version` if needed):

```yaml
---
roles:
  - name: githubixx.kubernetes_controller
    src: https://github.com/githubixx/ansible-role-kubernetes-controller.git
    version: 22.0.0+1.27.8
```

## Role (default) variables

```yaml
# The base directory for Kubernetes configuration and certificate files for
# everything control plane related. After the playbook is done this directory
# contains various sub-folders.
k8s_ctl_conf_dir: "/etc/kubernetes/controller"

# All certificate files (Private Key Infrastructure related) specified in
# "k8s_ctl_certificates" and "k8s_ctl_etcd_certificates" (see "vars/main.yml")
# will be stored here. Owner of this new directory will be "root". Group will
# be the group specified in "k8s_run_as_group"`. The files in this directory
# will be owned by "root" and group as specified in "k8s_run_as_group"`. File
# permissions will be "0640".
k8s_ctl_pki_dir: "{{ k8s_ctl_conf_dir }}/pki"

# The directory to store the Kubernetes binaries (see "k8s_ctl_binaries"
# variable in "vars/main.yml"). Owner and group of this new directory
# will be "root" in both cases. Permissions for this directory will be "0755".
#
# NOTE: The default directory "/usr/local/bin" normally already exists on every
# Linux installation with the owner, group and permissions mentioned above. If
# your current settings are different consider a different directory. But make sure
# that the new directory is included in your "$PATH" variable value.
k8s_ctl_bin_dir: "/usr/local/bin"

# The Kubernetes release.
k8s_ctl_release: "1.28.5"

# The interface on which the Kubernetes services should listen on. As all cluster
# communication should use a VPN interface the interface name is
# normally "wg0" (WireGuard),"peervpn0" (PeerVPN) or "tap0".
#
# The network interface on which the Kubernetes control plane services should
# listen on. That is:
#
# - kube-apiserver
# - kube-scheduler
# - kube-controller-manager
#
k8s_interface: "eth0"

# Run Kubernetes control plane service (kube-apiserver, kube-scheduler,
# kube-controller-manager) as this user.
#
# If you want to use a "secure-port" < 1024 for "kube-apiserver" you most
# probably need to run "kube-apiserver" as user "root" (not recommended).
#
# If the user specified in "k8s_run_as_user" does not exist then the role
# will create it. Only if the user already exists the role will not create it
# but it will adjust it's UID/GID and shell if specified (see settings below).
# So make sure that UID, GID and shell matches the existing user if you don't
# want that that user will be changed.
#
# Additionally if "k8s_run_as_user" is "root" then this role wont touch the user
# at all.
k8s_run_as_user: "k8s"

# UID of user specified in "k8s_run_as_user". If not specified the next available
# UID from "/etc/login.defs" will be taken (see "SYS_UID_MAX" setting in that file).
# k8s_run_as_user_uid: "999"

# Shell for specified user in "k8s_run_as_user". For increased security keep
# the default.
k8s_run_as_user_shell: "/bin/false"

# Specifies if the user specified in "k8s_run_as_user" will be a system user (default)
# or not. If "true" the "k8s_run_as_user_home" setting will be ignored. In general
# it makes sense to keep the default as there should be no need to login as
# the user that runs kube-apiserver, kube-scheduler or kube-controller-manager.
k8s_run_as_user_system: true

# Home directory of user specified in "k8s_run_as_user". Will be ignored if
# "k8s_run_as_user_system" is set to "true". In this case no home directory will
# be created. Normally not needed.
# k8s_run_as_user_home: "/home/k8s"

# Run Kubernetes daemons (kube-apiserver, kube-scheduler, kube-controller-manager)
# as this group.
#
# Note: If the group specified in "k8s_run_as_group" does not exist then the role
# will create it. Only if the group already exists the role will not create it
# but will adjust GID if specified in "k8s_run_as_group_gid" (see setting below).
k8s_run_as_group: "k8s"

# GID of group specified in "k8s_run_as_group". If not specified the next available
# GID from "/etc/login.defs" will be take (see "SYS_GID_MAX" setting in that file).
# k8s_run_as_group_gid: "999"

# Specifies if the group specified in "k8s_run_as_group" will be a system group (default)
# or not.
k8s_run_as_group_system: true

# By default all tasks that needs to communicate with the Kubernetes
# cluster are executed on local host (127.0.0.1). But if that one
# doesn't have direct connection to the K8s cluster or should be executed
# elsewhere this variable can be changed accordingly.
k8s_ctl_delegate_to: "127.0.0.1"

# The IP address or hostname of the Kubernetes API endpoint. This variable
# is used by "kube-scheduler" and "kube-controller-manager" to connect
# to the "kube-apiserver" (Kubernetes API server).
#
# By default the first host in the Ansible group "k8s_controller" is
# specified here. NOTE: This setting is not fault tolerant! That means
# if the first host in the Ansible group "k8s_controller" is down
# the worker node and its workload continue working but the worker
# node doesn't receive any updates from Kubernetes API server.
#
# If you have a loadbalancer that distributes traffic between all
# Kubernetes API servers it should be specified here (either its IP
# address or the DNS name). But you need to make sure that the IP
# address or the DNS name you want to use here is included in the
# Kubernetes API server TLS certificate (see "k8s_apiserver_cert_hosts"
# variable of https://github.com/githubixx/ansible-role-kubernetes-ca
# role). If it's not specified you'll get certificate errors in the
# logs of the services mentioned above.
k8s_ctl_api_endpoint_host: "{% set controller_host = groups['k8s_controller'][0] %}{{ hostvars[controller_host]['ansible_' + hostvars[controller_host]['k8s_interface']].ipv4.address }}"

# As above just for the port. It specifies on which port the
# Kubernetes API servers are listening. Again if there is a loadbalancer
# in place that distributes the requests to the Kubernetes API servers
# put the port of the loadbalancer here.
k8s_ctl_api_endpoint_port: "6443"

# Normally  "kube-apiserver", "kube-controller-manager" and "kube-scheduler" log
# to "journald". But there are exceptions like the audit log. For this kind of
# log files this directory will be used as a base path. The owner and group
# of this directory will be the one specified in "k8s_run_as_user" and "k8s_run_as_group"
# as these services run as this user and need permissions to create log files
# in this directory.
k8s_ctl_log_base_dir: "/var/log/kubernetes"

# Permissions for directory specified in "k8s_ctl_log_base_dir"
k8s_ctl_log_base_dir_mode: "0770"

# The port the control plane components should connect to etcd cluster
k8s_ctl_etcd_client_port: "2379"

# The interface the etcd cluster is listening on
k8s_ctl_etcd_interface: "eth0"

# The location of the directory where the Kubernetes certificates are stored.
# These certificates were generated by the "kubernetes_ca" Ansible role if you
# haven't used a different method to generate these certificates. So this
# directory is located on the Ansible controller host. That's normally the
# host where "ansible-playbook" gets executed. "k8s_ca_conf_directory" is used
# by the "kubernetes_ca" Ansible role to store the certificates. So it's
# assumed that this variable is already set.
k8s_ctl_ca_conf_directory: "{{ k8s_ca_conf_directory }}"

# Directory where "admin.kubeconfig" (the credentials file) for the "admin" user
# is stored. By default this directory (and it's "kubeconfig" file) will
# be stored on the host specified in "k8s_ctl_delegate_to". By default this
# is "127.0.0.1". So if you run "ansible-playbook" locally e.g. the directory
# and file will be created there.
#
# By default the value of this variable will expand to the user's local $HOME
# plus "/k8s/certs". That means if the user's $HOME directory is e.g.
# "/home/da_user" then "k8s_admin_conf_dir" will have a value of
# "/home/da_user/k8s/certs".
k8s_admin_conf_dir: "{{ '~/k8s/configs' | expanduser }}"

# Permissions for the directory specified in "k8s_admin_conf_dir"
k8s_admin_conf_dir_perm: "0700"

# Owner of the directory specified in "k8s_admin_conf_dir" and for
# "admin.kubeconfig" stored in this directory.
k8s_admin_conf_owner: "root"

# Group of the directory specified in "k8s_admin_conf_dir" and for
# "admin.kubeconfig" stored in this directory.
k8s_admin_conf_group: "root"

# Host where the "admin" user connects to administer the K8s cluster. 
# This setting is written into "admin.kubeconfig". This allows to use
# a different host/loadbalancer as the K8s services which might use an internal
# loadbalancer while the "admin" user connects to a different host/loadbalancer
# that distributes traffic to the "kube-apiserver" e.g.
#
# Besides that basically the same comments as for "k8s_ctl_api_endpoint_host"
# variable apply.
k8s_admin_api_endpoint_host: "{% set controller_host = groups['k8s_controller'][0] %}{{ hostvars[controller_host]['ansible_' + hostvars[controller_host]['k8s_interface']].ipv4.address }}"

# As above just for the port.
k8s_admin_api_endpoint_port: "6443"

# Directory to store "kube-apiserver" audit logs (if enabled). The owner and
# group of this directory will be the one specified in "k8s_run_as_user"
# and "k8s_run_as_group".
k8s_apiserver_audit_log_dir: "{{ k8s_ctl_log_base_dir }}/kube-apiserver"

# The directory to store "kube-apiserver" configuration.
k8s_apiserver_conf_dir: "{{ k8s_ctl_conf_dir }}/kube-apiserver"

# "kube-apiserver" daemon settings (can be overridden or additional added by defining
# "k8s_apiserver_settings_user")
k8s_apiserver_settings:
  "advertise-address": "{{ hostvars[inventory_hostname]['ansible_' + k8s_interface].ipv4.address }}"
  "bind-address": "{{ hostvars[inventory_hostname]['ansible_' + k8s_interface].ipv4.address }}"
  "secure-port": "6443"
  "enable-admission-plugins": "NodeRestriction,NamespaceLifecycle,LimitRanger,ServiceAccount,TaintNodesByCondition,Priority,DefaultTolerationSeconds,DefaultStorageClass,PersistentVolumeClaimResize,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota,PodSecurity,Priority,StorageObjectInUseProtection,RuntimeClass,CertificateApproval,CertificateSigning,ClusterTrustBundleAttest,CertificateSubjectRestriction,DefaultIngressClass"
  "allow-privileged": "true"
  "authorization-mode": "Node,RBAC"
  "audit-log-maxage": "30"
  "audit-log-maxbackup": "3"
  "audit-log-maxsize": "100"
  "audit-log-path": "{{ k8s_apiserver_audit_log_dir }}/audit.log"
  "event-ttl": "1h"
  "kubelet-preferred-address-types": "InternalIP,Hostname,ExternalIP"  # "--kubelet-preferred-address-types" defaults to:
                                                                       # "Hostname,InternalDNS,InternalIP,ExternalDNS,ExternalIP"
                                                                       # Needs to be changed to make "kubectl logs" and "kubectl exec" work.
  "runtime-config": "api/all=true"
  "service-cluster-ip-range": "10.32.0.0/16"
  "service-node-port-range": "30000-32767"
  "client-ca-file": "{{ k8s_ctl_pki_dir }}/ca-k8s-apiserver.pem"
  "etcd-cafile": "{{ k8s_ctl_pki_dir }}/ca-etcd.pem"
  "etcd-certfile": "{{ k8s_ctl_pki_dir }}/cert-k8s-apiserver-etcd.pem"
  "etcd-keyfile": "{{ k8s_ctl_pki_dir }}/cert-k8s-apiserver-etcd-key.pem"
  "encryption-provider-config": "{{ k8s_apiserver_conf_dir }}/encryption-config.yaml"
  "encryption-provider-config-automatic-reload": "true"
  "kubelet-certificate-authority": "{{ k8s_ctl_pki_dir }}/ca-k8s-apiserver.pem"
  "kubelet-client-certificate": "{{ k8s_ctl_pki_dir }}/cert-k8s-apiserver.pem"
  "kubelet-client-key": "{{ k8s_ctl_pki_dir }}/cert-k8s-apiserver-key.pem"
  "service-account-key-file": "{{ k8s_ctl_pki_dir }}/cert-k8s-controller-manager-sa.pem"
  "service-account-signing-key-file": "{{ k8s_ctl_pki_dir }}/cert-k8s-controller-manager-sa-key.pem"
  "service-account-issuer": "https://{{ groups.k8s_controller | first }}:6443"
  "tls-cert-file": "{{ k8s_ctl_pki_dir }}/cert-k8s-apiserver.pem"
  "tls-private-key-file": "{{ k8s_ctl_pki_dir }}/cert-k8s-apiserver-key.pem"

# This is the content of "encryption-config.yaml". Used by "kube-apiserver"
# (see "encryption-provider-config" option in "k8s_apiserver_settings").
# "kube-apiserver" will use this configuration to encrypt data before storing
# it in etcd (encrypt data at-rest).
#
# The configuration below is a usable example but might not fit your needs.
# So please review carefully! E.g. you might want to replace "aescbc" provider
# with a different one like "secretbox". As you can see this configuration only
# encrypts "secrets" at-rest. But it's also possible to encrypt other K8s
# resources. NOTE: "identity" provider doesn't encrypt anything! That means
# plain text. In the configuration below it's used as fallback.
#
# If you keep the default defined below please make sure to specify the
# variable "k8s_encryption_config_key" somewhere (e.g. "group_vars/all.yml" or
# even better use "ansible-vault" to store these kind of secrets).
# This needs to be a base64 encoded value. To create such a value on Linux
# run the following command:
#
# head -c 32 /dev/urandom | base64
#
# For a detailed description please visit:
# https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/
#
# How to rotate the encryption key or to implement encryption at-rest in
# an existing K8s cluster please visit:
# https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#rotating-a-decryption-key
k8s_apiserver_encryption_provider_config: |
  ---
  kind: EncryptionConfiguration
  apiVersion: apiserver.config.k8s.io/v1
  resources:
    - resources:
        - secrets
      providers:
        - aescbc:
            keys:
              - name: key1
                secret: {{ k8s_encryption_config_key }}
        - identity: {}

# The directory to store controller manager configuration.
k8s_controller_manager_conf_dir: "{{ k8s_ctl_conf_dir }}/kube-controller-manager"

# K8s controller manager settings (can be overridden or additional added by defining
# "k8s_controller_manager_settings_user")
k8s_controller_manager_settings:
  "bind-address": "{{ hostvars[inventory_hostname]['ansible_' + k8s_interface].ipv4.address }}"
  "secure-port": "10257"
  "cluster-cidr": "10.200.0.0/16"
  "allocate-node-cidrs": "true"
  "cluster-name": "kubernetes"
  "authentication-kubeconfig": "{{ k8s_controller_manager_conf_dir }}/kubeconfig"
  "authorization-kubeconfig": "{{ k8s_controller_manager_conf_dir }}/kubeconfig"
  "kubeconfig": "{{ k8s_controller_manager_conf_dir }}/kubeconfig"
  "leader-elect": "true"
  "service-cluster-ip-range": "10.32.0.0/16"
  "cluster-signing-cert-file": "{{ k8s_ctl_pki_dir }}/cert-k8s-apiserver.pem"
  "cluster-signing-key-file": "{{ k8s_ctl_pki_dir }}/cert-k8s-apiserver-key.pem"
  "root-ca-file": "{{ k8s_ctl_pki_dir }}/ca-k8s-apiserver.pem"
  "requestheader-client-ca-file": "{{ k8s_ctl_pki_dir }}/ca-k8s-apiserver.pem"
  "service-account-private-key-file": "{{ k8s_ctl_pki_dir }}/cert-k8s-controller-manager-sa-key.pem"
  "use-service-account-credentials": "true"

# The directory to store scheduler configuration.
k8s_scheduler_conf_dir: "{{ k8s_ctl_conf_dir }}/kube-scheduler"

# kube-scheduler settings
k8s_scheduler_settings:
  "bind-address": "{{ hostvars[inventory_hostname]['ansible_' + k8s_interface].ipv4.address }}"
  "config": "{{ k8s_scheduler_conf_dir }}/kube-scheduler.yaml"
  "authentication-kubeconfig": "{{ k8s_scheduler_conf_dir }}/kubeconfig"
  "authorization-kubeconfig": "{{ k8s_scheduler_conf_dir }}/kubeconfig"
  "requestheader-client-ca-file": "{{ k8s_ctl_pki_dir }}/ca-k8s-apiserver.pem"

# These sandbox security/sandbox related settings will be used for
# "kube-apiserver", "kube-scheduler" and "kube-controller-manager"
# systemd units. These options will be placed in the "[Service]" section.
# The default settings should be just fine for increased security of the
# mentioned services. So it makes sense to keep them if possible.
#
# For more information see:
# https://www.freedesktop.org/software/systemd/man/systemd.service.html#Options
#
# The options below "RestartSec=5" are mostly security/sandbox related settings
# and limit the exposure of the system towards the unit's processes. You can add
# or remove options as needed of course. For more information see:
# https://www.freedesktop.org/software/systemd/man/systemd.exec.html
k8s_ctl_service_options:
  - User={{ k8s_run_as_user }}
  - Group={{ k8s_run_as_group }}
  - Restart=on-failure
  - RestartSec=5
  - NoNewPrivileges=true
  - ProtectHome=true
  - PrivateTmp=true
  - PrivateUsers=true
  - ProtectSystem=full
  - ProtectClock=true
  - ProtectKernelModules=true
  - ProtectKernelTunables=true
  - ProtectKernelLogs=true
  - ProtectControlGroups=true
  - ProtectHostname=true
  - ProtectControlGroups=true
  - RestrictNamespaces=true
  - RestrictRealtime=true
  - RestrictSUIDSGID=true
  - CapabilityBoundingSet=~CAP_SYS_PTRACE
  - CapabilityBoundingSet=~CAP_KILL
  - CapabilityBoundingSet=~CAP_MKNOD
  - CapabilityBoundingSet=~CAP_SYS_CHROOT
  - CapabilityBoundingSet=~CAP_SYS_ADMIN
  - CapabilityBoundingSet=~CAP_SETUID
  - CapabilityBoundingSet=~CAP_SETGID
  - CapabilityBoundingSet=~CAP_SETPCAP
  - CapabilityBoundingSet=~CAP_CHOWN
  - SystemCallFilter=@system-service
  - ReadWritePaths=-/usr/libexec/kubernetes
```

The kube-apiserver settings defined in `k8s_apiserver_settings` can be overridden by defining a variable called `k8s_apiserver_settings_user`. You can also add additional settings by using this variable. E.g. to override `audit-log-maxage` and `audit-log-maxbackup` default values and add `watch-cache` add the following settings to `group_vars/k8s.yml`:

```yaml
k8s_apiserver_settings_user:
  "audit-log-maxage": "40"
  "audit-log-maxbackup": "4"
  "watch-cache": "false"
```

The same is true for the `kube-controller-manager` by adding entries to `k8s_controller_manager_settings_user` variable. For `kube-scheduler` add entries to `k8s_scheduler_settings_user` variable to override/add settings in `k8s_scheduler_settings` dictionary.

## Example Playbook

```yaml
- hosts: k8s_controller
  roles:
    - githubixx.kubernetes_controller
```

## Testing

This role has a small test setup that is created using [Molecule](https://github.com/ansible-community/molecule), libvirt (vagrant-libvirt) and QEMU/KVM. Please see my blog post [Testing Ansible roles with Molecule, libvirt (vagrant-libvirt) and QEMU/KVM](https://www.tauceti.blog/posts/testing-ansible-roles-with-molecule-libvirt-vagrant-qemu-kvm/) how to setup. The test configuration is [here](https://github.com/githubixx/ansible-role-kubernetes-controller/tree/master/molecule/default).

Afterwards Molecule can be executed:

```bash
molecule converge
```

This will setup a few virtual machines (VM) with supported Ubuntu OS and installs an Kubernetes cluster but without the worker nodes (so no nodes with `kube-proxy` and `kubelet` installed). A small verification step is also included:

```bash
molecule verify
```

To clean up run

```bash
molecule destroy
```

## License

GNU GENERAL PUBLIC LICENSE Version 3

## Author Information

[http://www.tauceti.blog](http://www.tauceti.blog)
