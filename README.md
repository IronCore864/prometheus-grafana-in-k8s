# 0 Prerequisites

You should have a k8s cluster working already, and we assume you have RBAC enabled.

# 1 Create Service Account and Cluster Role Binding

Create a yaml file and use kubectl to create. Example:

See file `rbac-config.yaml`.

Run:

```
kubectl create -f rbac-config.yaml
```

This will create an service account named "tiller" and bind it with cluster-admin role which is a default in kubernetes.

# 2 Install Helm

Helm is a package manager for Kubernetes.

To install Helm, follow official readme.

To make it short, if you are using mac, run:

```
brew install kubernetes-helm
```

If on Ubuntu, download the binary release here for Linux, unzip, and put it as `/usr/local/bin/helm`.

# 3 Install Tiller

Tiller is the cluster-side service for helm.

Helm will figure out where to install Tiller by reading your Kubernetes configuration file (usually $HOME/.kube/config). This is the same file that kubectl uses.

Once you have Helm ready, you can initialize the local CLI and also install Tiller into your Kubernetes cluster in one step:

```
helm init --service-account <NAME>
```

Here you want to replace the <NAME> by the service account name you created in step 1, in our example, it's "tiller".

# 4 Deploy

```
$ helm repo add coreos https://s3-eu-west-1.amazonaws.com/coreos-charts/stable/
$ helm install coreos/prometheus-operator --name prometheus-operator --namespace monitoring
$ helm install coreos/kube-prometheus --name kube-prometheus --set global.rbacEnable=true --namespace monitoring
```

This adds coreos repo, installs prometheus-operator and prometheus with helms.

More details on prometheus-operator see [HERE](https://github.com/coreos/prometheus-operator).

Assuming all is well, you can run the command bellow to list the apps:

```
kubectl get pods -n monitoring
```

In the result, you should be able to see:

- prometheus-kube-prometheus-0
- prometheus-operator
- kube-prometheus-grafana

up and running.

# 5 Test

Forward the Prometheus server to your machine so you can take a better look at the dashboard by opening http://localhost:9090

```
kubectl port-forward -n monitoring prometheus-kube-prometheus-0 9090
```

To have a good-looking dashboard, use Grafana, it has a datasource ready to query on Prometheus.

```
kubectl port-forward $(kubectl get pods --selector=app=kube-prometheus-grafana -n monitoring --output=jsonpath="{.items..metadata.name}") -n monitoring 3000
```

Wait few seconds until grafana load the dashboards, open your browser at http://localhost:3000 to check out your dashboard.

# 6 Todo

- Expose Grafana as a service so that proxy/port-forward is not needed
- Automation for the whole process using ansible/shell
- Persistent storage

