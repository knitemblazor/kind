## setting up a gateway 

#### install istio 

istio version = 1.15.0

the demo profile has ingress enabled - need to explore more
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


#### set up using a default webapp