# Install certmanager
# If you skip this - you'll need to go through the operator code and remove references + it wont support tls
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.1/cert-manager.yaml

# Create namespace
If you want to use another namespace, please refer to the values.yaml of the nifioperator and update it as well as the cluster resources to support it. 

The operator "Watches" resources in a specific namespaces only so you have to configure the role properly.

```
kubectl create namespace nifi
```

# Installing the chart
helm install nifikop ./nifioperator --namespace nifi

# On Openshift
uid=$(kubectl get namespace nifi -o=jsonpath='{.metadata.annotations.openshift\.io/sa\.scc\.supplemental-groups}' | sed 's/\/10000$//' | tr -d '[:space:]')

helm install nifikop \
    ./nifioperator \
    --namespace=nifi \
    --set runAsUser=$uid

# Create a ZK cluster
Make sure to replace global.storageClass with your own. Also, increase these resoure limits don't fit production.

helm repo add bitnami https://charts.bitnami.com/bitnami


helm install zookeeper bitnami/zookeeper \
    --namespace=zookeeper \
    --set resources.requests.memory=256Mi \
    --set resources.requests.cpu=250m \
    --set resources.limits.memory=256Mi \
    --set resources.limits.cpu=250m \
    --set global.storageClass=ebs-sc \
    --set networkPolicy.enabled=true \
    --set replicaCount=3 \
    --create-namespace


# On openshift

Run the following to get  the zk UID
```
zookeper_uid=$(kubectl get namespace zookeeper -o=jsonpath='{.metadata.annotations.openshift\.io/sa\.scc\.supplemental-groups}' | sed 's/\/10000$//' | tr -d '[:space:]')

helm install zookeeper bitnami/zookeeper \
    --set resources.requests.memory=256Mi \
    --set resources.requests.cpu=250m \
    --set resources.limits.memory=256Mi \
    --set resources.limits.cpu=250m \
    --set global.storageClass=ebs-sc \
    --set networkPolicy.enabled=true \
    --set replicaCount=3 \
    --set containerSecurityContext.runAsUser=$zookeper_uid \
    --set podSecurityContext.fsGroup=$zookeper_uid
```

# Edit the nifi CRD
Edit the storage class definition and other settings for the cluster.

# Apply the nifi CRD
kubectl create -n nifi -f simplenifi.yaml


# On openshift:
```
uid=$(kubectl get namespace nifi -o=jsonpath='{.metadata.annotations.openshift\.io/sa\.scc\.supplemental-groups}' | sed 's/\/10000$//' | tr -d '[:space:]')
sed -i "s/1000690000/$uid/g" openshift.yaml
kubectl create -n nifi -f openshift.yaml
```

