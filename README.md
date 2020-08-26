# Vault Trust Operator

The Vault Trust Operator manages the trust between a K8S cluster and [Hashicorp Vault](https://www.vaultproject.io/).

In addition to the K8S resources required to run the operator, on deployment the following service accounts will be
created

*   `vault-token-reviewer` with cluster wide permissions to create tokenreviews
*   `vault-trust-admin` with no k8s permissions

When configuring a [Kubernetes Auth Method](https://www.vaultproject.io/docs/auth/kubernetes) the 
`vault-token-reviewer`'s JWT is used as part of the configuration to enable Vault to authenticate K8S
service accounts.

The `vault-trust-admin` will be mapped to all Kubernetes Auth Methods with permissions to manage roles 
and to remove the auth method when no longer required.

After deploying the Operator, create a [VaultKubeAuth](config/samples/vault_v1alpha1_vaultkubeauth.yaml)
custom resource to initialise the trust.

```
apiVersion: vault.controlplane.io/v1alpha1
kind: VaultKubeAuth
metadata:
  name: vaultkubeauth-sample
spec:
  cluster_api_addr: https://kubernetes.default.svc/
  vault_addr: http://vault.vault.svc:8200/
  vault_mount_path: k8s/cluster-1
  vault_mount_admin_policy: vaultkubeauth-cluster1-admin
  vault_token: s.xxxxxxxxxxxxxxxxxxxxxxxx

```

This will create the following K8S resources:

*   A configmap containing the templated scripts for managing the Kubernetes Auth Method
*   A job to configure the Kubernetes Auth Method in Vault

When the job has completed the following will be configured in Vault

*   The Kubernetes Auth Method for this cluster
*   The mapping of the administration policy to the _vault-trust-admin_ service account

When the custom resource is deleted, the Kubernetes Auth Method will be removed for this cluster

With the trust initialised, create [VaultKubeRole](config/samples/vault_v1alpha1_vaultkuberole.yaml) custom
resources to create roles in Vault for additional service accounts created in the cluster.

```
apiVersion: vault.controlplane.io/v1alpha1
kind: VaultKubeRole
metadata:
  name: vaultkuberole-sample
spec:
  vaultkubeauth_name: vaultkubeauth-sample
  service_account_name: default
  service_account_namespace: default
  policies:
    - default
```

When the custom resources are deleted, the roles will be removed
