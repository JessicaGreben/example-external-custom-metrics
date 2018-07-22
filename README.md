# example-external-custom-metrics
An example app with kubernetes configuration to autoscale on external custom metrics.

### Description

This is an example app that shows how to set up pod autoscaling with custom and external metrics.

See [this blog post]() for more details.

### Steps

1. Using Google Cloud [GKE service create a Kubernetes cluster](https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-cluster) version 1.10 or greater so that there is support for external metrics.

2. Install Tiller in this cluster. This step assumes you have [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) and [helm](https://docs.helm.sh/using_helm/) installed locally and that your kubeconfig is configured to talk to the cluster created in step 1.

    # confirm kubectl is configured for the correct cluster
    kubectl config current-context

    # create serviceaccount for tiller to use
    kubectl -n kube-system create sa tiller
    
    # create a clusterrolebinding for tiller
    kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller

    # create the tiller deployment specifying the tiller serviceaccount
    helm init --upgrade --service-account tiller

3. Deploy the External Metrics server. Here I use the [Stackdriver Adapter](https://github.com/GoogleCloudPlatform/k8s-stackdriver/tree/master/custom-metrics-stackdriver-adapter) that implements the external metrics API.

4. Deploy [Nginx Ingress Controller helm chart](https://github.com/helm/charts/tree/master/stable/nginx-ingress). Make sure to add the [`prometheus-to-stackdriver`](https://github.com/GoogleCloudPlatform/k8s-stackdriver/tree/master/prometheus-to-sd) sidecar to the Nginx deployment. This can be added to the default values.yaml file with `controller.extraConatiners` configuration.  [Here is a good example](https://github.com/GoogleCloudPlatform/k8s-stackdriver/blob/master/prometheus-to-sd/kubernetes/prometheus-to-sd-kube-state-metrics.yaml#L26) to follow, however the  `--source` value needs to be change to where nginx exposes it's metrics.

    helm upgrade --install nginx stable/nginx-ingress --values charts/nginx-ingress-ctlr-values/values.yaml
    
5. Deploy the example application as a Kubernetes deployment with a service, ingress, and the horizontal pod autoscaler.

    helm install charts/example-nodejs-app/ --name example-nodejs-app
