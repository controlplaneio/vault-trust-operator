# Vault Policies

Create a bootstrap policy that allows the Kubernetes Auth Method to be mounted, configured and a role created for the
`vault-trust-admin` ServiceAccount in K8S.

This will be used when a VaultKubeAuth is created and should have a restricted use limit of 3 when the token is created. 

This needs to be performed once, at Vault deployment time, assuming a fixed path for the mount.

The following example assumes the kubernetes auth methods will be mounted at k8s/<mount-id>.

`vaultkubeauth-bootstrap`

```hcl
path "auth/k8s/*"
{
  capabilities = ["create", "update"]
}
path "sys/auth/k8s/*"
{
  capabilities = ["update", "sudo"]
}
```

For each mount, create an admin policy, restricting access to configuring roles on the mount and deleting the mount when
no longer required. The name of this policy will be passed in via the VaultKubeAuth CustomResource.

`vaultkubeauth-cluster1-mount1-admin`

```hcl
path "sys/auth/k8s/cluster1-mount1"
{
    capabilities = ["delete", "sudo"]
}
path "auth/k8s/cluster1-mount1/role/*"
{
    capabilities = ["create", "read", "update", "delete", "list"]
}
```

```shell script
vault token create -policy=vaultkubeauth-bootstrap -use-limit=3 -field=token
```
