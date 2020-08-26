# OpenShift Development

Create the Project

```shell script
oc new-project vault-trust-operator
```
Create the BuildConfig and the ImageStream

```shell script
oc new-build --name=vault-trust-operator \
    --to='vault-trust-operator' \
    --binary=true
```

Start a build

```shell script
oc start-build vault-trust-operator \
  --from-dir=.
```

Set the image in [operator.yaml](../deploy/operator.yaml)

```shell script
sed -i 's|REPLACE_IMAGE|'$(oc get is vault-trust-operator -ojsonpath={.status.dockerImageRepository})':latest|g' deploy/operator.yaml
```

Deploy

```shell script
oc create -f deploy/crds/vault.controlplane.io_vaultkubeauths_crd.yaml
oc create -f deploy/crds/vault.controlplane.io_vaultkuberoles_crd.yaml
oc create -f deploy/service_account.yaml
oc create -f deploy/vault_token_reviewer.yaml
oc create -f deploy/vault_token_reviewer_clusterrole.yaml
oc create -f deploy/vault_token_reviewer_clusterrole_binding.yaml
oc create -f deploy/vault_trust_admin.yaml
oc create -f deploy/role.yaml
oc create -f deploy/role_binding.yaml
oc create -f deploy/operator.yaml
```

Undeploy

```shell script
oc delete -f deploy/crds
oc delete -f deploy
oc delete project vault-trust-operator
```
