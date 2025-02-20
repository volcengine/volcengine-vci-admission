# Deployment Guide

## Prerequisites

### IAM
`volcengine-vci-admission` uses AccessKeyID and SecretAccessKey for initialization.
Before deploying the component, you need to generate the Volcengine AccessKeyID and grant it the necessary permissions. The minimum set of permissions required by `volcengine-vci-admission` is listed in the [policy](./policy.json) .

For more information on IAM authentication, see [IAM](https://www.volcengine.com/docs/6257/0).

### Configurations
| Name                                                   | Description                                                                           | Required  | Example                     |
|------------------------------------------------------|-----------------------------------------------------------------------------------|-------|------------------------| 
| cluster.ak                                           | Volcengine account ak                                                             | **YES** |                        |
| cluster.sk                                           | Volcengine account sk                                                             | **YES** |                        |
| cluster.region                                       | The region of service                                                             | **YES** | cn-beijing             |
| cluster.volcHost                                      | Volcengint Open API host, the default value is `open.volcengineapi.com`           | **YES** | open.volcengineapi.com | 
| deploy.image.registry| image registry, the default value is `open-registry-cn-beijing.cr.volces.com/vke` | **NO** | |
| deploy.image.name| image name, the default value is `volcengine-vci-admission`                       | **NO** | |
| deploy.image.tag| image tag, the default value is `v1.1.0-public`                                   | **NO** | |
See [values.yaml](../manifests/values.yaml) for more details.

## Installation Steps
This repository provides the [Charts directory](../manifests) for installation.
`volcengine-vci-admission` is deployed as Deployment and accesses the Kubernetes cluster through inClusterConfig or specified kubeconfig. 
Therefore, you can deploy the component in the following two ways:

### 1. Access the cluster with inClusterConfig
The following command will create the resources for the component, including the `ServiceAccount`, `ClusterRole` and `ClusterRoleBinding`.
```
helm install volcengine-vci-admission {{CHART_FOLDER}}/manifests -n {{NAMESPACE}} \
--set cluster.ak=akxxx \
--set cluster.sk=skxxx \
--set cluster.region=cn-beijing
```
Here, `{{CHART_FOLDER}}` is the directory where the Charts are located, and `{{NAMESPACE}}` is the namespace where you want to deploy the component.

### 2. Access the cluster with specified kubeconfig
To specify the credentials for `volcengine-vci-admission` to access the cluster, ensure that the credentials have the permissions defined in [rbac.yaml](../manifests/templates/rbac.yaml). 
The following steps demonstrate how to generate a credential and use it.
1. Create rbac. See [rbac.yaml](../manifests/templates/rbac.yaml) for details.
2. Create kubeconfig for the service account.
- Execute the following command to get the TOKEN. Replace `{{NAMESPACE}}` and `{{SecretName}}` with the `Secret` created in step 1.
```
kubectl get secret -n {{NAMESPACE}} {{SecretName}} -o json | jq -r '.data["token"]' | base64 -d
```
- Execute the following command to get the CA. Replace `{{NAMESPACE}}` and `{{SecretName}}` with the `Secret` created in step 1.
```
kubectl get secret -n {{NAMESPACE}} {{SecretName}} -o json | jq -r '.data["ca.crt"]'
```
3. Create configmap containing the kubeconfig.
Use the TOKEN and CA obtained in step 2 to replace `{{TOKEN}}` and `{{CA}}`. And Replace `{{CLUSTER_NAME}}` and `{{APISERVER_ADDR}}` with the cluster name and apiserver address of your cluster.
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: volcengine-vci-admission-cluster-config
  namespace: kube-system
data:
  volcengine-vci-admission.conf: |
    apiVersion: v1
    kind: Config
    contexts:
      - context:
          cluster: {{CLUSTER_NAME}}
          user: system:volcengine-vci-admission
        name: system:volcengine-vci-admission
    current-context: system:volcengine-vci-admission
    users:
      - name: system:volcengine-vci-admission
        user:
          token: {{TOKEN}}
    clusters:
      - cluster:
          certificate-authority-data: {{CA}}
          server: {{APISERVER_ADDR}}
        name: {{CLUSTER_NAME}}
```
4. Deploy `volcengine-vci-admission`
```
helm install volcengine-vci-admission {{CHART_FOLDER}}/manifests -n {{NAMESPACE}} \
--set cluster.ak=akxxx \
--set cluster.sk=skxxx \
--set cluster.region=cn-beijing \
--set deploy.authenticationMode=useKubeconfig
```
Here, `{{CHART_FOLDER}}` is the directory where the Charts are located, and `{{NAMESPACE}}` is the namespace where you want to deploy the component.

## Metrics
`volcengine-vci-admission` exposes metrics at /metrics. The following metrics are available:
- metrics:

| Name                                    | Type        | Description                                   |
|---------------------------------------|-----------|-----------------------------------------------|
| volcengine_request_duration_in_seconds                       | Histogram | The request to volcengine host, which can be aggregated and analyzed from the `name` and `status` dimensions | 

- controller-runtime metrics, such as:

| Name                                    | Type        | Description                                   |
|---------------------------------------|-----------|-----------------------------------------------|
| controller_runtime_webhook_latency_seconds                       | Histogram | The latency of processing admission requests. |
| controller_runtime_webhook_requests_in_flight                       | Gauge     | Current number of admission requests being served.                          |
| controller_runtime_webhook_requests_total                       | Counter   | Total number of admission requests by HTTP status code.                              |
