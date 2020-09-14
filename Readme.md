## Demo for installing ECK/

```bash
# Install OLM (Operator Lifecycle Management)
kubectl apply -f https://github.com/operator-framework/operator-lifecycle-manager/releases/download/0.16.1/crds.yaml
kubectl apply -f https://github.com/operator-framework/operator-lifecycle-manager/releases/download/0.16.1/olm.yaml
kubectl get olm -A

# Install ECK/Elasticsearch Operator
kubectl apply -f https://download.elastic.co/downloads/eck/1.2.1/all-in-one.yaml

# Create a namespace for logging
kubectl create ns logging

# Install Elasticsearch cluster (Powershell)
@"
apiVersion: elasticsearch.k8s.elastic.co/v1beta1
kind: Elasticsearch
metadata:
  name: quickstart
spec:
  version: 7.5.0
  nodeSets:
  - name: default
    count: 1
    config:
      node.master: true
      node.data: true
      node.ingest: true
      node.store.allow_mmap: false
"@ | kubectl apply -n logging -f -

# Install Elasticsearch cluster (Bash)
cat <<EOF | kubectl apply -n logging -f -
apiVersion: elasticsearch.k8s.elastic.co/v1beta1
kind: Elasticsearch
metadata:
  name: quickstart
spec:
  version: 7.5.0
  nodeSets:
  - name: default
    count: 1
    config:
      node.master: true
      node.data: true
      node.ingest: true
      node.store.allow_mmap: false
EOF

# Install Kibana (Powershell)
@"
apiVersion: kibana.k8s.elastic.co/v1beta1
kind: Kibana
metadata:
  name: quickstart
spec:
  version: 7.5.0
  count: 1
  elasticsearchRef:
    name: quickstart
"@ | kubectl apply -n logging -f -

# Install Kibana (Shell)
cat <<EOF | kubectl apply -n logging -f -
apiVersion: kibana.k8s.elastic.co/v1beta1
kind: Kibana
metadata:
  name: quickstart
spec:
  version: 7.5.0
  count: 1
  elasticsearchRef:
    name: quickstart
EOF

# Install Logging/Fluentd & Fluent Bit Operator inside logging namespace
helm repo add banzaicloud-stable https://kubernetes-charts.banzaicloud.com
helm install --namespace logging logging banzaicloud-stable/logging-operator --set createCustomResource=false
kubectl -n logging get pods

# Install demo app
helm install --namespace logging logging-demo banzaicloud-stable/logging-demo --set "elasticsearch.enabled=True"

# Get the password for Kibana (default user is 'elastic')'
kubectl get secret quickstart-es-elastic-user -n logging -o=jsonpath='{.data.elastic}' | base64 --decode; echo

# Access Kibana dashboard
kubectl port-forward -n logging svc/quickstart-kb-http 5601
```
