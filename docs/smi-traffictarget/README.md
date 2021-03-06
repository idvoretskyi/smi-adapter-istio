# Traffic Target

This demo will show you how to use the [traffic spec](https://github.com/servicemeshinterface/smi-spec/blob/master/traffic-specs.md) and [traffic access control](https://github.com/servicemeshinterface/smi-spec/blob/master/traffic-access-control.md) specification

## Prerequisites
Follow the [getting started section](https://github.com/servicemeshinterface/smi-adapter-istio#getting-started) to ensure you have the following:
- A running Kubernetes cluster
- Istio installed on Kubernetes cluster
- Deployed smi-istio-operator and related configs
```bash
kubectl apply -f https://raw.githubusercontent.com/servicemeshinterface/smi-adapter-istio/master/deploy/crds/crds.yaml
kubectl apply -f https://raw.githubusercontent.com/servicemeshinterface/smi-adapter-istio/master/deploy/operator-and-rbac.yaml
```

## Setup sample bookinfo application
* Now deploy the sample bookinfo application

```bash
kubectl label namespace default istio-injection=enabled
kubectl apply -f manifests/configs.yml
```

Try to visit the application homepage and verify that the application is running fine.

If you are running istio and this whole setup on minikube then the easiest way to access the application is to open the URL generated by following command in your browser.

```bash
echo "http://$(minikube ip):31380/productpage"
```

But if you are running this setup in a public cloud that allows load balancer services then run following command to get the URL.

```bash
echo "http://$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')/productpage"
```

Above command extracts the public IP of the service `istio-ingressgateway` of type Load Balancer. And constructs the URL to reach the productpage.

* Deploy configs which are needed to provide access to frontend application *productpage*.

```bash
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.1/samples/bookinfo/platform/kube/rbac/rbac-config-ON.yaml
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.1/samples/bookinfo/platform/kube/rbac/productpage-policy.yaml
```

If you try to revisit the application, you will see that you don't get any output from `details` and `reviews` page. And will see the output as `Error fetching product details!` and `Error fetching product reviews!`.


## Deploy Traffic Access Control and Traffic Spec

* Finally deploy the configs

File [traffictarget.yaml](manifests/traffictarget.yaml) has configs needed to allow traffic among all the micro-services.

```bash
kubectl apply -f manifests/traffictarget.yaml
```

Once the configs are deployed after a while you can see that the application functions without any problems. The only difference from the original application is that right now you will see reviews from v1 and v3 only i.e. you will see only orange colored stars or no stars.

* Deploy the config to allow productpage to talk to reviews-v2

```
kubectl apply -f manifests/traffictarget-reviews-v2.yaml
```

Now you can see that black colored stars also start appearing on the page. If you look at the config file [traffictarget-reviews-v2.yaml](manifests/traffictarget-reviews-v2.yaml) there productpage is allowed to talk to reviews-v2 service.

## Cleanup
Delete sample bookinfo application and all SMI TrafficTarget related configuration:

```console
$ kubectl delete -f manifests/
```
