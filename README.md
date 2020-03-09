# Monitoring Cloud Pak for Data using Prometheus, Grafana and AlertManager


OpenShift administrators often face the same challenges as other system administrators: "I need a tool that will monitor the health of my system." Yet, traditional monitoring tools often fall short in their visibility of an OpenShift cluster. Thus, a typical OpenShift monitoring stack includes Prometheus for systems as well as service monitoring, and Grafana for analyzing and visualizing metrics.

Often, administrators are looking to write custom queries and create custom dashboards in Grafana. However, the Grafana instance that is provided with the monitoring stack, along with its dashboards, is read-only. Enter the community-powered Grafana operator provided by OperatorHub.

## 1. Deploying Custom Grafana
The community-powered Grafana cannot be deployed to the existing openshift-monitoring namespace, so we will create a new namespace (i.e. my-monitoring) to deploy into instead.  Navigate to OperatorHub and select the community-powered Grafana Operator.  Press Continue to accept the disclaimer, press Install, and press Subscribe to accept the default configuration values and deploy to the my-monitoring namespace.  Within some time, the Grafana operator will be made available in the my-monitoring namespace.

```
oc new-project monitor
oc project monitor
oc new-app --name=mydashboard docker.io/grafana/grafana:6.6.2
oc adm pod-network join-projects --to=monitoring openshift-monitoring
oc expose svc/mydashboard
```


## Connecting Prometheus to our Custom Grafana
The next step is to connect the existing Prometheus in the openshift-monitoring namespace with the community supported Grafana in the my-monitoring namespace.  To do this, configure the prometheus-k8s StatefulSet to listen on all addresses, not just localhost. Navigate to Workloads -> StatefulSets, edit the YAML for the prometheus-k8s StatefulSet by changing '--web.listen-address=127.0.0.1:9090' to '--web.listen-address=:9090', and save.

```
oc edit statefulset prometheus-k8s -n openshift-monitoring

```


```
spec:
  template:
    spec:
      containers:
        - name: prometheus
          args:
            - '--web.listen-address=:9090'
```

## Customizing Grafana

From the my-monitoring namespace, navigate to Networking -> Routes and click on the Grafana URL to display the custom Grafana user interface.  Click on ‘Sign In’ from the bottom left menu of Grafana, and log in using the default username and password configured earlier.  Now, an editable Grafana interface appears.

Navigate to Configuration -> Data Sources from the menu, press Add Data Source, and select Prometheus.  For the HTTP URL, type in http://prometheus-operated.openshift-monitoring.svc.cluster.local:9090 and you can leave the rest of the configuration values as default.  Press Save and Test to confirm that Grafana is able to connect to the existing Prometheus data source.

Finally, you can create or import a custom Grafana dashboard.  I imported a custom Grafana dashboard from a JSON file in the screenshot below, which displayed the custom metrics I had been looking to view.

Prometheus data source url
```
http://prometheus-operated.openshift-monitoring.svc.cluster.local:9090 
```
