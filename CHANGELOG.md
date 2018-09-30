Changelog
---------

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
