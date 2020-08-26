# Run Tests

You need a valid K8S context with admin permissions

```shell script
pip install molecule podman openshift
```

Build the Operator Image

```shell script
IMG=docker.io/controlplane/vault-trust-operator:v0.1.0
sed -i 's|REPLACE_IMAGE|'$IMG'|g' deploy/operator.yaml
docker build -f Dockerfile -t $IMG
docker push $IMG
```

Run the tests

`molecule test`
