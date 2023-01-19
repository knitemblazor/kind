#### <ins>install istio in kind</ins>

$ kind create cluster

*creating kind cluster with "ingress-ready=true" causes istio installation to fail* \

using istio version = 1.15.0

the demo profile has ingress enabled - need to explore more \

$ istioctl install --set profile=demo -y

$ kubectl get services

$ kubectl get pods

$ kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml

check if the application is up and running
$ kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"

#### Open the application to outside traffic

$ kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml 

$ export INGRESS_NAME=istio-ingressgateway

$ export INGRESS_NS=istio-system

$ kubectl get svc "$INGRESS_NAME" -n "$INGRESS_NS"

$ export INGRESS_HOST=$(kubectl -n "$INGRESS_NS" get service "$INGRESS_NAME" -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

$ export INGRESS_PORT=$(kubectl -n "$INGRESS_NS" get service "$INGRESS_NAME" -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')

$ export SECURE_INGRESS_PORT=$(kubectl -n "$INGRESS_NS" get service "$INGRESS_NAME" -o jsonpath='{.spec.ports[?(@.name=="https")].port}')

$ export TCP_INGRESS_PORT=$(kubectl -n "$INGRESS_NS" get service "$INGRESS_NAME" -o jsonpath='{.spec.ports[?(@.name=="tcp")].port}')

$ export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT

$ curl -s "http://${GATEWAY_URL}/productpage" | grep -o "<title>.*</title>"


#### set up nginx ingress

creating kind cluster without ingress-ready leads to failing nginx pod initialization

```console
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
EOF
```

$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
