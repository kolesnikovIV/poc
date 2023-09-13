# https://trello.com/c/8URqdF7r
## prerequests
- kubectl, helm, istioctl binaries installed
- kubeconfig has been configured (aws eks cluster)
## install istio
```
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update
kubectl create namespace istio-system
helm install istio-base istio/base -n istio-system --set defaultRevision=default
helm install istiod istio/istiod -n istio-system --wait
```
## enable mTLS
since we preparing for SoC2 good fit is strictly mode for cluster:
```
kubectl apply -n istio-system -f - <<EOF
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
spec:
  mtls:
    mode: STRICT
EOF
```
## setup workloads
```
kubectl create ns foo
curl -O https://raw.githubusercontent.com/istio/istio/release-1.19/samples/httpbin/httpbin.yaml
curl -O https://raw.githubusercontent.com/istio/istio/release-1.19/samples/sleep/sleep.yaml
kubectl apply -f <(istioctl kube-inject -f httpbin.yaml) -n foo
kubectl apply -f <(istioctl kube-inject -f sleep.yaml) -n foo
kubectl create ns bar
kubectl apply -f <(istioctl kube-inject -f httpbin.yaml) -n bar
kubectl apply -f <(istioctl kube-inject -f sleep.yaml) -n bar
kubectl create ns legacy
kubectl apply -f sleep.yaml -n legacy
```
### Demo and explanation
Execute and get output:
```
for from in "foo" "bar" "legacy"; do for to in "foo" "bar"; do kubectl exec "$(kubectl get pod -l app=sleep -n ${from} -o jsonpath={.items..metadata.name})" -c sleep -n ${from} -- curl http://httpbin.${to}:8000/ip -s -o /dev/null -w "sleep.${from} to httpbin.${to}: %{http_code}\n"; done; done
```
Output:
```
sleep.foo to httpbin.foo: 200
sleep.foo to httpbin.bar: 200
sleep.bar to httpbin.foo: 200
sleep.bar to httpbin.bar: 200
sleep.legacy to httpbin.foo: 000
command terminated with exit code 56
sleep.legacy to httpbin.bar: 000
command terminated with exit code 56
```
since `legacy` namespace has no istio proxy container, for that service mTLS is disabled and we cannot connect to the services, then we got issue: `command terminated with exit code 56`

## Summary
That pretty quick exmplanation how mTLS String mode works, mTLS has permissive mode that will allow to connect w/o mTLS if it's disabled. ALso there is lots of ways to setup for exact service via envoyconfig/VirtualService/DestinationRules also.
