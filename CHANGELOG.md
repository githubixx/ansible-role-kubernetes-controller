Changelog
---------

**8.0.1+1.14.6**

- update `k8s_release` to `1.14.6`

**8.0.0+1.14.2**

- update `k8s_release` to `1.14.2`
- add all admissions plugins to `enable-admission-plugins` option that are enabled by default in K8s 1.14
- remove `Initializers` addmission plugin (no longer available in 1.14)

**7.0.0+1.13.5**

- update `k8s_release` to `1.13.5`
- introduce `bind-address` flag and bind on VPN IP by default
- introduce `port` flag for kube-controller-manager and set value to 0 to disable unsecure port

**6.0.0+1.13.2**

- update `k8s_release` to `1.13.2`
- kube-apiserver: `--experimental-encryption-provider-config` flag is deprecated and replaced in favor of `--encryption-provider-config`
- kube-apiserver: the configuration file referenced by `--encryption-provider-config` now uses `kind: EncryptionConfiguration` and `apiVersion: apiserver.config.k8s.io/v1`. Support for `kind: EncryptionConfig` and `apiVersion: v1` is deprecated and will be removed in a future release. See https://github.com/githubixx/ansible-kubernetes-playbooks/tree/master/kubeencryptionconfig and https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-certificate-authority/ (search for `kubeencryptionconfig.yml`). To avoid deprecation warnings it makes sense to create a new `encryption-config.yaml` before running this role to update `kube-apiserver`.
- use correct semantic versioning as described in https://semver.org. Needed for Ansible Galaxy importer as it now insists on using semantic versioning.
- make Ansible linter happy

**r5.0.1_v1.12.3**

- update `k8s_release` to `1.12.3`
- kube-apiserver: added `Priority` admission plugin
- kube-scheduler: deprecated group version changed from `componentconfig/v1alpha1` to `kubescheduler.config.k8s.io/v1alpha1`
- kube-controller-manager: replace deprecated `--address` setting with `--bind-address`

**r5.0.0_v1.11.3**

- update `k8s_release` to `1.11.3`

**r4.0.4_v1.10.8**

- update `k8s_release` to `1.10.8`

**r4.0.3_v1.10.4**

- support Ubuntu 18.04

**r4.0.2_v1.10.4**

- wait for kube-apiserver on port 8080 no longer needed (fixes [#11](https://github.com/githubixx/ansible-role-kubernetes-controller/issues/11))

**r4.0.0_v1.10.4**

- update `k8s_release` to `1.10.4`
- removed deprecated kube-apiserver parameter `insecure-bind-address` (see: [#59018](https://github.com/kubernetes/kubernetes/pull/59018))
- added variable `k8s_apiserver_secure_port: 6443`
- added parameter `secure-port` to `k8s_apiserver_settings` parameter list
- added `kube-controller-manager-ca` certificate files to `k8s_certificates` list
- added variable `k8s_controller_manager_conf_dir` / added kubeconfig for kube-controller-manager
- added variable `k8s_scheduler_conf_dir` / added kubeconfig for kube-scheduler / settings for kube-scheduler now in ` templates/var/lib/kube-scheduler/kube-scheduler.yaml.j2`
- added kubeconfig for `admin` user (located by default in `k8s_conf_dir`). This `admin.kubeconfig` will be needed for `kubectl`
- new `service-account-key-file` value for kube-apiserver
- changes in `k8s_controller_manager_settings`: removed `master` parameter, added `kubeconfig`, new value for `service-account-private-key-file`, new parameter `use-service-account-credentials`

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
