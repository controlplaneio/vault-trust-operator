# Packaging and Deployment using OLM

Build the Operator Image

```shell script
IMG=docker.io/controlplane/vault-trust-operator:v0.1.0
sed -i 's|REPLACE_IMAGE|'$IMG'|g' deploy/operator.yaml
docker build -f Dockerfile -t $IMG
docker push $IMG
```

Build and Validate the Bundle Image

```shell script
BUNDLE_IMG=docker.io/controlplane/vault-trust-operator-bundle:v0.1.0
docker build -f bundle.Dockerfile -t $BUNDLE_IMG
docker push $BUNDLE_IMG
operator-sdk bundle validate $BUNDLE_IMG
```

Create an Index Image

```shell script
INDEX_IMG=docker.io/controlplane/operator-index:v0.1.0
opm index add --bundles $BUNDLE_IMG --tag $INDEX_IMG
docker push $INDEX_IMG
```

Create a CatalogSource

```shell script
cat <<EOF | kubectl apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  name: controlplane-operators
  namespace: openshift-marketplace
spec:
  displayName: Control Plane Operators
  description: Open source operators from Control Plane
  sourceType: grpc
  image: docker.io/controlplane/operator-index:v0.1.0
EOF
```

Create a Subscription

```shell script
cat <<EOF | kubectl apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: vault-trust-operator
  namespace: vault-trust-operator
spec:
  channel: alpha
  installPlanApproval: Automatic
  name: vault-trust-operator
  source: controlplane-operators
  sourceNamespace: openshift-marketplace
  startingCSV: vault-trust-operator.v0.1.0
EOF
```
