# Changelog

## 23.0.0+1.28.5

### UPDATE

- update `k8s_release` to `1.28.5`

### BREAKING

- Extend `enable-admission-plugins` in `k8s_apiserver_settings` by: `PodSecurity,Priority,StorageObjectInUseProtection,RuntimeClass,CertificateApproval,CertificateSigning,ClusterTrustBundleAttest,CertificateSubjectRestriction,DefaultIngressClass`. These are enabled by default if this flag is not specified (see [Admission Controllers Reference](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/) for more information).

## 22.0.0+1.27.8

### PLEASE READ CAREFULLY

This release contains quite a few potential breaking changes! So review carefully before rolling out the new version of this role! A bigger part of the whole changes are related to increase security. While most of the new variables and defaults should be just fine and should just work out of the box side effects might occur.

All the newly introduced or changed variables have detailed comments in [README](https://github.com/githubixx/ansible-role-kubernetes-controller/blob/master/README.md). So please read them carefully!

This refactoring was needed to make it possible to have `githubixx.kubernetes_controller` and `githubixx.kubernetes_worker` deployed on the same host e.g. They were some intersections between the two roles that had to be fixed.

**Please remove** `/var/lib/kubernetes/admin.kubeconfig` on the K8s controller nodes (if you didn't change the default directory for this file). Older versions of this role created this file. It's no longer needed. It contains the `kubeconfig` (so basically the credentials file) for the `admin` user. This is a very powerful user (actually the user with the most permissions). So use with care and store the file in a secure place! `admin.kubeconfig` should only be used at the very beginning to create a new user with less permissions.

### UPDATE

- update `k8s_release` to `1.27.8`

### BREAKING

- Rename variable `k8s_conf_dir` to `k8s_ctl_conf_dir`. Additionally the default value changed from `/usr/lib/kubernetes` to `/etc/kubernetes/controller`.
- Introduce variable `k8s_admin_conf_dir`. Currently it only stores `admin.kubeconfig` which is basically the credentials file of the `admin` "user". Formerly this file was stored in the directory specified in the (now removed) `k8s_config_directory` variable. The default value of `k8s_admin_conf_dir` is the same as the removed `k8s_config_directory`. Additionally to set permissions for `k8s_admin_conf_dir` the following variables were introduced: `k8s_admin_conf_dir_perm`, `k8s_admin_conf_owner` and `k8s_admin_conf_group`.
- Introduce variable `k8s_ctl_pki_dir`. All certificate files specified in `k8s_ctl_certificates` and `k8s_ctl_etcd_certificates` (see `vars/main.yml`) will be stored here. Related to this: Certificate related settings in `k8s_apiserver_settings` used `k8s_conf_dir` before and now use `k8s_ctl_pki_dir`. That's `client-ca-file`, `etcd-cafile`, `etcd-certfile`, `etcd-keyfile`, `kubelet-certificate-authority`, `kubelet-client-certificate`, `kubelet-client-key`, `service-account-key-file`, `service-account-signing-key-file`, `tls-cert-file` and `tls-private-key-file`. For `k8s_controller_manager_settings` that's: `cluster-signing-cert-file`, `cluster-signing-key-file`, `root-ca-file`, `requestheader-client-ca-file` and `service-account-private-key-file`. And for `k8s_scheduler_settings` that's: `requestheader-client-ca-file`.
- Rename variable `k8s_bin_dir` to `k8s_ctl_bin_dir`.
- Rename variable `k8s_release` to `k8s_ctl_release`.
- The default value for `k8s_interface` changed from `tap0` to `eth0`.
- Rename variable `k8s_controller_binaries` to `k8s_ctl_binaries`. Additionally this variable is no longer defined in `defaults/main.yml` but in `vars/main.yml`. Since this list is fixed anyways it makes no sense to allow to modify this list.
- Rename variable `k8s_certificates` to `k8s_ctl_certificates`.
- Rename variable `etcd_certificates` to `k8s_ctl_etcd_certificates`. Additionally this variable is no longer defined in `defaults/main.yml` but in `vars/main.yml`. Since this list is fixed anyways it makes no sense to allow to modify this list.
- Rename variable `etcd_client_port` to `k8s_ctl_etcd_client_port`.
- Rename variable `etcd_interface` to `k8s_ctl_etcd_interface`. Additionally the default value changed from `tap0` to `eth0`.
- Use `k8s_ctl_etcd_interface` variable instead of `k8s_interface` for `--etcd-servers` option in `templates/etc/systemd/system/kube-apiserver.service.j2`. Normally `etcd` and `kube-apiserver` listen on the same interface. But if someone specified `k8s_ctl_etcd_interface` (formerly `etcd_interface` in the context of this role) it was basically ignored as the value of `k8s_interface` was used instead. That's fixed now.
- Rename variable `k8s_controller_delegate_to` to `k8s_ctl_delegate_to`.
- Introduce `k8s_apiserver_conf_dir` variable. `encryption-config.yaml` is now located in `k8s_apiserver_conf_dir`.
- Change default value of `k8s_controller_manager_conf_dir` to `"{{ k8s_ctl_conf_dir }}/kube-controller-manager"`.
- Change default value of `k8s_scheduler_conf_dir` to `"{{ k8s_ctl_conf_dir }}/kube-scheduler"`.
- Rename `kube-controller-manager.kubeconfig` to `kubeconfig`. This affects `k8s_controller_manager_settings` and the following settings: `authentication-kubeconfig`, `authorization-kubeconfig` and `kubeconfig`.
- Rename `kube-scheduler.kubeconfig` to `kubeconfig`. This affects `k8s_scheduler_settings` and the following settings: `authentication-kubeconfig"` and `authorization-kubeconfig`.
- Added new option `encryption-provider-config-automatic-reload: "true"` to `k8s_apiserver_settings`. In case the file specified in `encryption-provider-config` changes `kube-apiserver` will automatically reload that file. This is handy if one wants to change the encryption provider (also see new variable `k8s_apiserver_encryption_provider_config`)
- Introduce `k8s_ctl_service_options` variable. As mentioned above already previously `kube-apiserver`, `kube-controller-manager` and `kube-scheduler` were running as user `root`. Now these services will run as `k8s_run_as_user` and `k8s_run_as_group`. Additionally `systemd` allows to limit the exposure of the system towards the unit's processes. Basically all settings below `RestartSec=5` are related to increase security and limit what the process allowed to do. So these settings reduce the attack surface quite a bit already. They're not perfect but a starting point. If you want the previous behavior just remove all settings besides `Restart` and `RestartSec`. But that also means that they'll run again as `root` user.
- The following variables are no longer used and can be removed: `k8s_encryption_config_directory`, `k8s_encryption_config_owner`, `k8s_encryption_config_group`, `k8s_encryption_config_directory_perm` and `k8s_encryption_config_file_perm`. Previously these values were needed to specify permissions on the Ansible controller for this file/directory. Since this file is now generated directly on the K8s controller nodes they're no longer needed. That also means you can remove `encryption-config.yaml` from Ansible controller node (like your workstation e.g.)
- The variable `k8s_config_directory` is gone. It's no longer in use. After the upgrade to this release you can delete this directory (if you accept the new default!) and it's content (make a backup esp. of `admin.kubeconfig` file - just in case!)
- Rename `k8s_ca_conf_directory` to `k8s_ctl_ca_conf_directory`

### FEATURE

- Introduce `k8s_run_as_user` variable. Previously all control plane services like `kube-apiserver`, `kube-scheduler` and `kube-controller-manager` run as user `root`. Securitywise that's not optimal. There is just no need to run them as `root` as long they use a listening port > `1024` which they do by default. In this version all these services will run as the user specified with `k8s_run_as_user` which is `k8s` by default. Related to this variable are the new variables `k8s_run_as_user_shell`, `k8s_run_as_user_system`, `k8s_run_as_group` and `k8s_run_as_group_system`. See [README](https://github.com/githubixx/ansible-role-kubernetes-controller/blob/master/README.md) for further information about this variables. The defaults should be just fine even for upgrading from a previous version of this role.
- Introduce `k8s_ctl_api_endpoint_host` and `k8s_ctl_api_endpoint_port` variables. Previously `kube-scheduler` and `kube-controller-manager` where configured to connect to the first host in the Ansible `k8s_controller` group and communicate with the `kube-apiserver` that was running there. This was hard-coded and couldn't be changed. If that host was down the K8s worker nodes didn't receive any updates. Now one can install and use a load balancer like `haproxy` e.g. that distributes requests between all `kube-apiserver`'s and takes a `kube-apiserver` out of rotation if that one is down (also see my Ansible [haproxy role](https://github.com/githubixx/ansible-role-haproxy) for that use case). The default is still to use the first host/kube-apiserver in the Ansible `k8s_controller` group. So behaviorwise nothing changed basically.
- Introduce `k8s_admin_api_endpoint_host` and `k8s_admin_api_endpoint_port` variables. For these two variables the same is basically true as for `k8s_ctl_api_endpoint_host` and `k8s_ctl_api_endpoint_port` variables above. But these settings are meant to be used by the `admin` user that this role creates by default. These settings are written into `admin.kubeconfig`. So it's possible to configure another host/load balancer for the `admin` user as for the K8s control plane services mentioned in the previous paragraph.
- Introduce `k8s_ctl_log_base_dir` and `k8s_ctl_log_base_dir_mode`. Normally  `kube-apiserver`, `kube-controller-manager` and `kube-scheduler` log to `journald`. But there are exceptions like the audit log. For this kind of log files this directory will be used as a base path.
- Introduce `k8s_apiserver_audit_log_dir`. Directory to store kube-apiserver audit logs.
- Introduce `k8s_apiserver_encryption_provider_config` variable. Previously the content of this file was hard-coded. Now it's exposed via this variable. The content of that variable and the previously hard-coded value are the same. So if you keep the default when upgrading everything stays the same in that regards. **NOTE**: Changing this configuration and deploy the changes can potentially cause quite some problems! Make sure to read [Encrypting Confidential Data at Rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/) and esp. [Rotating a decryption key](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#rotating-a-decryption-key)!
- Add task to generate `kubeconfig` for `admin` user (previously this was a separate [playbook](https://github.com/githubixx/ansible-kubernetes-playbooks/tree/v15.0.0_r1.27.5/kubeauthconfig)).
- Add task to generate `kubeconfig` for `kube-controller-manager` service (previously this was a separate [playbook](https://github.com/githubixx/ansible-kubernetes-playbooks/tree/v15.0.0_r1.27.5/kubeauthconfig)).
- Add task to generate `kubeconfig` for `kube-scheduler` service (previously this was a separate [playbook](https://github.com/githubixx/ansible-kubernetes-playbooks/tree/v15.0.0_r1.27.5/kubeauthconfig)).
- When downloading the Kubernetes binaries the task checks the SHA512 checksum
- Make `kube-scheduler` and `kube-controller-manager` wait until `kube-apiserver` has started and is listening on a port

### OTHER CHANGES

- Extend `k8s_ctl_certificates` (formerly `k8s_certificates`) list by `cert-k8s-scheduler` and `cert-k8s-controller-manager` files. This was needed as the `kubeconfig` files are now generated on the K8s controller nodes and no longer on the Ansible controller host. Previously it was needed to prepare these files upfront before installing this role. That's no longer needed. Also see **FEATURES** list above.
- Use `kubernetes.core.*` modules instead of `kubectl` binary
- Fix some `ansible-lint` issues

### MOLECULE

- Updated all files to reflect the changes introduces with this version
- Tasks for creating `kubeconfig` for `kube-controller-manager`, `kube-scheduler`, `admin` user and `encryption configuration` are no longer needed as they're now part of `kubernetes_controller` role
- Add `haproxy` to Ubuntu 22 hosts to test new `k8s_ctl_api_endpoint_host` and `k8s_ctl_api_endpoint_port` settings
- Add tasks to install [ansible-role-cni](https://github.com/githubixx/ansible-role-cni) and [ansible-role-runc](https://github.com/githubixx/ansible-role-runc)
- Use `kubernetes.core.k8s_info` module instead of calling `kubectl` binary

## 21.1.3+1.27.5

- rename `githubixx.harden-linux` to `githubixx.harden_linux`

## 21.1.2+1.27.5

- rename `githubixx.kubernetes-ca` to `githubixx.kubernetes_ca`

## 21.1.1+1.27.5

- `molecule/default/molecule.yml`: use Ubuntu 20.04 instead of 22.04 for `test-assets` for now because of certificate problems with Python `urllib` module

## 21.1.0+1.27.5

- add support for Ubuntu 22.04

## 21.0.0+1.27.5

- **BREAKING**: `meta/main.yml`: change role_name from `kubernetes-controller` to `kubernetes_controller`. This is a requirement since quite some time for Ansible Galaxy. But the requirement was introduced after this role already existed for quite some time. So please update the name of the role in your playbook accordingly!
- update `k8s_release` to `1.27.5`
- `meta/main.yml`: remove Ubuntu 18.04 as supported OS (reached EOL)

## 20.0.1+1.26.8

- update `k8s_release` to `1.26.8`
- `kube-apiserver` needs to have network-online.target ready
- `kube-controller-manager` needs to have network-online.target ready
- `kube-scheduler` needs to have network-online.target ready

## 20.0.0+1.26.4

- update `k8s_release` to `1.26.4`
- add Molecule test
- add Github workflow

## 19.2.0+1.25.9

- update `k8s_release` to `1.25.9`
- `kube-apiserver`: remove `--apiserver-count` flag. It has been deprecated and will be removed in a future K8s release.
- `templates/var/lib/kube-scheduler/kube-scheduler.yaml.j2`: `KubeSchedulerConfiguration v1beta2` is deprecated in Kubernetes v1.25, will be removed in v1.26

## 19.1.0+1.25.5

- Introduce `k8s_controller_delegate_to` variable. By default it's set to `127.0.0.1` and reflects the same value as before.

## 19.0.0+1.25.5

- update `k8s_release` to `1.25.5`

## 18.0.1+1.24.9

- update `k8s_release` to `1.24.9`

## 18.0.0+1.24.4

- update `k8s_release` to `1.24.4`

## 17.1.0+1.23.10

- update `k8s_release` to `1.23.10`

## 17.0.0+1.23.3

- update `k8s_release` to `1.23.3`
- add parameter `authentication-kubeconfig`, `authorization-kubeconfig` and `requestheader-client-ca-file` to `k8s_scheduler_settings` (see [K8s Deprecations 1.23](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.23.md#deprecation))
- remove `healthzBindAddress` and `metricsBindAddress` from `kube-scheduler.yaml.j2` (deprecated)
- this role now requires Ansible >= 2.9

## 16.1.0+1.22.6

- update `k8s_release` to `1.22.6`
- use `kubescheduler.config.k8s.io/v1beta2` in `templates/var/lib/kube-scheduler/kube-scheduler.yaml.j2` (`v1beta1` will be removed in Kubernetes v1.23 - see [Remove scheduler policy config and cc v1beta1](https://github.com/kubernetes/enhancements/issues/2901)

## 16.0.0+1.22.5

- update `k8s_release` to `1.22.5`
- add parameter `authentication-kubeconfig`, `authorization-kubeconfig` and `requestheader-client-ca-file` to `k8s_controller_manager_settings` (see [K8s Deprecations 1.22](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.22.md#deprecation))
- removed `kubelet-https: "true"` from `k8s_apiserver_settings` as no longer supported by `kube-apiserver` (see: [Mark --kubelet-https deprecated](https://github.com/kubernetes/kubernetes/pull/91630))

## 15.0.1+1.21.8

- update `k8s_release` to `1.21.8`

## 15.0.0+1.21.4

- update `k8s_release` to `1.21.4`
- remove Ubuntu 16.04 support

## 14.1.0+1.20.10

- update `k8s_release` to `1.20.10`

## 14.0.0+1.20.8

- update `k8s_release` to `1.20.8`

## 13.1.0+1.19.12

- update `k8s_release` to `1.19.12`

## 13.0.0+1.19.4

- update 'k8s_release' to `1.19.4`
- `KubeSchedulerConfiguration` graduates to Beta (see [Scheduler Configuration](https://kubernetes.io/docs/reference/scheduling/config)). Upgrade `kubescheduler.config.k8s.io/v1alpha2` to `kubescheduler.config.k8s.io/v1beta1`

## 12.2.0+1.18.12

- update `k8s_release` to `1.18.12`

## 12.1.0+1.18.6

- update `k8s_release` to `1.18.6`
- added `"allocate-node-cidrs": "true"` to`k8s_controller_manager_settings` otherwise `cluster-cidr` setting won't be used by `kube-controller-manager`
- creating ClusterRole's and ClusterRoleBindings is now delegated to `127.0.0.1` (localhost) instead of picking the first Kubernetes controller node for that task

## 12.0.1+1.18.5

- added Ubuntu 20.04 (Focal Fossa) as supported platform

## 12.0.0+1.18.5

- update `k8s_release` to `1.18.5`
- renamed `cert-etcd.pem/cert-etcd-key.pem` to `cert-k8s-apiserver-etcd.pem/cert-k8s-apiserver-etcd-key.pem`. This was also adjusted in `etcd_certificates` list. The changed name makes it more obvious that this is a client certificate for `kube-apiserver` used to connect to a TLS secured `etcd` cluster. In fact `kube-apiserver` is just a client to `etcd` as all clients. In my [ansible-role-kubernetes-ca](https://github.com/githubixx/ansible-role-kubernetes-ca) this was also changed accordingly (see `etcd_additional_clients` list). [ansible-role-kubernetes-ca](https://github.com/githubixx/ansible-role-kubernetes-ca) is now able to generate client certificates for other services like Traefik or Cilium which are often used in a Kubernetes cluster. So the already existing `etcd` cluster for Kubernetes (esp. for `kube-apiserver`) can be reused for other components.
- replaced `cluster-signing-cert-file": "{{k8s_conf_dir}}/ca-k8s-apiserver.pem` with `cluster-signing-cert-file": "{{k8s_conf_dir}}/cert-k8s-apiserver.pem` in `k8s_controller_manager_settings`
- removed deprecated `port` setting in `k8s_controller_manager_settings` which was replaced by `secure-port` setting (default value `10257`)
- removed `k8s_apiserver_secure_port` as it makes no sense. The value `6443` can be set in `k8s_apiserver_settings` (`"secure-port": "6443"`) as it is not used elsewhere
- `kubescheduler.config.k8s.io/v1alpha1` changed to `kubescheduler.config.k8s.io/v1alpha2` in `kube-scheduler.yaml.j2` (see: [CHANGELOG](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.18.md#kube-scheduler-1))

## 11.0.0+1.17.4

- update `k8s_release` to `1.17.4`
- `rbac.authorization.k8s.io/v1beta1` changed to `rbac.authorization.k8s.io/v1`
- update `runtime-config` (needs boolean expression now)

## 10.1.1+1.16.8

- update `k8s_release` to `1.16.8`

## Remove old tags

- The following tags are removed as they're not compatible with Ansible Galaxy and I guess nobody uses them anymore:

```bash
r2.0.0_v1.9.0
r2.0.1_v1.9.0
r2.0.2_v1.9.1
r3.0.0_v1.9.1
r3.0.0_v1.9.3
r3.0.0_v1.9.8
r4.0.1_v1.10.4
r4.0.2_v1.10.4
r4.0.3_v1.10.4
r4.0.4_v1.10.8
r5.0.1_v1.12.3
v1.0.0_r1.5.1
v1.0.0_v1.8.0
v1.0.0_v1.8.2
v1.1.0_v1.8.4
v1.1.1_v1.8.4
v1.1.2_v1.8.4
v1.2.0_v1.8.4
```

## 10.1.0+1.16.3

- strengthen file permissions for certificate files and other config files

## 10.0.0+1.16.3

- update `k8s_release` to `1.16.3`
- remove deprecated `enable-swagger-ui` option from `kube-apiserver`

## 9.0.0+1.15.6

- update `k8s_release` to `1.15.6`

## 9.0.0+1.15.3

- update `k8s_release` to `1.15.3`

## 8.0.1+1.14.6

- update `k8s_release` to `1.14.6`

## 8.0.0+1.14.2

- update `k8s_release` to `1.14.2`
- add all admissions plugins to `enable-admission-plugins` option that are enabled by default in K8s 1.14
- remove `Initializers` addmission plugin (no longer available in 1.14)

## 7.0.0+1.13.5

- update `k8s_release` to `1.13.5`
- introduce `bind-address` flag and bind on VPN IP by default
- introduce `port` flag for kube-controller-manager and set value to 0 to disable unsecure port

## 6.0.0+1.13.2

- update `k8s_release` to `1.13.2`
- kube-apiserver: `--experimental-encryption-provider-config` flag is deprecated and replaced in favor of `--encryption-provider-config`
- kube-apiserver: the configuration file referenced by `--encryption-provider-config` now uses `kind: EncryptionConfiguration` and `apiVersion: apiserver.config.k8s.io/v1`. Support for `kind: EncryptionConfig` and `apiVersion: v1` is deprecated and will be removed in a future release. See [kubeencryptionconfig](https://github.com/githubixx/ansible-kubernetes-playbooks/tree/master/kubeencryptionconfig) and [Kubernetes the not so hard way with Ansible - Certificate authority](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-certificate-authority/)  (search for `kubeencryptionconfig.yml`). To avoid deprecation warnings it makes sense to create a new `encryption-config.yaml` before running this role to update `kube-apiserver`.
- use correct semantic versioning as described in [semver](https://semver.org). Needed for Ansible Galaxy importer as it now insists on using semantic versioning.
- make Ansible linter happy

## r5.0.1_v1.12.3

- update `k8s_release` to `1.12.3`
- kube-apiserver: added `Priority` admission plugin
- kube-scheduler: deprecated group version changed from `componentconfig/v1alpha1` to `kubescheduler.config.k8s.io/v1alpha1`
- kube-controller-manager: replace deprecated `--address` setting with `--bind-address`

## r5.0.0_v1.11.3

- update `k8s_release` to `1.11.3`

## r4.0.4_v1.10.8

- update `k8s_release` to `1.10.8`

## r4.0.3_v1.10.4

- support Ubuntu 18.04

## r4.0.2_v1.10.4

- wait for kube-apiserver on port 8080 no longer needed (fixes [#11](https://github.com/githubixx/ansible-role-kubernetes-controller/issues/11))

## r4.0.0_v1.10.4

- update `k8s_release` to `1.10.4`
- removed deprecated kube-apiserver parameter `insecure-bind-address` (see: [#59018](https://github.com/kubernetes/kubernetes/pull/59018))
- added variable `k8s_apiserver_secure_port: 6443`
- added parameter `secure-port` to `k8s_apiserver_settings` parameter list
- added `kube-controller-manager-ca` certificate files to `k8s_certificates` list
- added variable `k8s_controller_manager_conf_dir` / added kubeconfig for kube-controller-manager
- added variable `k8s_scheduler_conf_dir` / added kubeconfig for kube-scheduler / settings for kube-scheduler now in `templates/var/lib/kube-scheduler/kube-scheduler.yaml.j2`
- added kubeconfig for `admin` user (located by default in `k8s_conf_dir`). This `admin.kubeconfig` will be needed for `kubectl`
- new `service-account-key-file` value for kube-apiserver
- changes in `k8s_controller_manager_settings`: removed `master` parameter, added `kubeconfig`, new value for `service-account-private-key-file`, new parameter `use-service-account-credentials`

## r3.0.0_v1.9.8

- update `k8s_release` to `1.9.8`

## r3.0.0_v1.9.3

- update `k8s_release` to `1.9.3`

## r3.0.0_v1.9.1

- move advertise-address,bind-address,insecure-bind-address out of kube-apiserver.service.j2 template
- move address,master settings out of kube-controller-manager.service.j2 template / fix variable bug in `k8s_apiserver_settings`
- move address,master settings out of kube-scheduler.service.j2 template
- fix: use `k8s_etcd` hosts group instead of `k8s_controller` group to generate etcd server list
- we need to wait for kube-apiserver port 8080 to become ready before running kubectl tasks

## r2.0.2_v1.9.1

- update to Kubernetes v1.9.1

## r2.0.1_v1.9.0

- removed duplicate key cluster-signing-cert-file from `k8s_controller_manager_settings` dictionary

## r2.0.0_v1.9.0

- introduce flexible parameter settings for API server via `k8s_apiserver_settings/k8s_apiserver_settings_user`
- introduce flexible parameter settings for controller manager via `k8s_controller_manager_settings/k8s_controller_manager_settings_user`
- introduce flexible parameter settings for kube-scheduler via `k8s_scheduler_settings/k8s_scheduler_settings_user`
- change defaults for `k8s_ca_conf_directory` and `k8s_config_directory` variables
- update to Kubernetes v1.9.0

No changelog for releases < r2.0.0_v1.9.0 (see commit history if needed)
