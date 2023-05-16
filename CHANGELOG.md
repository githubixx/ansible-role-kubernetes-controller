# Changelog

## 19.2.0+1.25.9

- update `k8s_release` to `1.25.9`

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
