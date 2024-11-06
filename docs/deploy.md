# 部署指南

## 前置准备

### IAM
`volcengine-vci-admission` 使用火山引擎的 AccessKeyID 和 SecretAccessKey 初始化。
在部署组件之前，需要生成火山引擎的 AccessKeyID，并为其授权相关权限，[policy](./policy.json) 中列举了 `volcengine-vci-admission` 所需要的最小权限集合。

IAM 鉴权的更多信息见 [IAM](https://www.volcengine.com/docs/6257/0)。

### 关键配置项说明
| 名称                                                   | 说明                                                                                     | 是否必须  | 示例                     |
|------------------------------------------------------|----------------------------------------------------------------------------------------|-------|------------------------| 
| cluster.ak                                           | 火山引擎账号 AccessKeyID                                                                     | **是** |                        |
| cluster.sk                                           | 火山引擎账号 SecretAccessKey                                                                 | **是** |                        |
| cluster.region                                       | 火山引擎资源所属的区域                                                                            | **是** | cn-beijing             |
| cluster.volcHost                                      | 火山引擎网关地址，默认值为 `open.volcengineapi.com`                                                                         | **是** | open.volcengineapi.com | 
| deploy.image.registry| `volcengine-vci-admission` 组件所在的镜像仓库，默认值为 `open-registry-cn-beijing.cr.volces.com/vke` | **否** | |
| deploy.image.name| `volcengine-vci-admission` 组件的镜像名称，默认值为 `volcengine-vci-admission`                     | **否** | |
| deploy.image.tag| `volcengine-vci-admission` 组件的镜像名称，默认值为 `v1.1.0-public`                                       | **否** | |


更多配置详见 [values.yaml](../manifests/values.yaml)

## 部署步骤

本仓库中提供了组件安装的 [Charts 目录](../manifests)。`volcengine-vci-admission` 以 Deployment 形式部署，
并通过 inClusterConfig 或者指定 kubeconfig 的方式访问 Kubernetes 集群。 因此，将本仓库克隆到本地后，可以通过以下两种方式部署组件：

### 1.组件通过 inClusterConfig 的方式访问集群
执行以下命令，会自动创建组件相关的资源，包括组件访问集群的 `ServiceAccount`, `ClusterRole`, `ClusterRoleBinding` 等资源。

```
helm install volcengine-vci-admission {{CHART_FOLDER}}/manifests -n {{NAMESPACE}} \
--set cluster.ak=akxxx \
--set cluster.sk=skxxx \
--set cluster.region=cn-beijing
```
其中 `{{CHART_FOLDER}}` 为 Charts 所在目录，`{{NAMESPACE}}` 期望组件部署的命名空间。


### 2.组件通过指定 kubeconfig 的方式访问集群
如果想要指定 `volcengine-vci-admission` 访问集群的凭证，确保该凭证拥有 [rbac.yaml](../manifests/templates/rbac.yaml) 中定义的权限。
以下步骤展示了如何生成一个凭证，并在部署 `volcengine-vci-admission` 时使用该凭证。

1. 创建 `volcengine-vci-admission` 需要使用的 rbac 资源。资源模板见 [rbac.yaml](../manifests/templates/rbac.yaml)
2. 获取 `volcengine-vci-admission` 的 service account 信息。
- 执行以下命令，将输出的结果作为 TOKEN。其中 `{{NAMESPACE}}`, `{{SecretName}}` 为步骤1中创建的 `Secret` 资源信息。
```
kubectl get secret -n {{NAMESPACE}} {{SecretName}} -o json | jq -r '.data["token"]' | base64 -d 
```
- 执行以下命令，将输出的结果作为 CA。其中 `{{NAMESPACE}}`, `{{SecretName}}` 为步骤1中创建的 `Secret` 资源信息。
```
kubectl get secret -n {{NAMESPACE}} {{SecretName}} -o json | jq -r '.data["ca.crt"]'
```
3. 创建包含了 kubeconfig 的 configmap。
用步骤2得到的 TOKEN, CA 替换 `{{TOKEN}}`, `{{CA}}`；用您的集群的集群名称和 apiserver 的地址替换 `{{CLUSTER_NAME}}` 和 `{{APISERVER_ADDR}}`。

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
4. 部署 volcengine-vci-admission

```
helm install volcengine-vci-admission {{CHART_FOLDER}}/manifests -n {{NAMESPACE}} \
--set cluster.ak=akxxx \
--set cluster.sk=skxxx \
--set cluster.region=cn-beijing \
--set deploy.authenticationMode=useKubeconfig
```

其中 `{{CHART_FOLDER}}` 为 Charts 所在目录，`{{NAMESPACE}}` 期望组件部署的命名空间。

## 指标获取

`volcengine-vci-admission` 指标可以通过访问 /metrics 路径获取，指标包括：
- 相关业务指标：

| 名称                                    | 类型        | 说明                                                 |
|---------------------------------------|-----------|----------------------------------------------------|
| volcengine_request_duration_in_seconds                       | Histogram | 向火山引擎网关发送的请求时间分布，可以从 `name` 和 `status` 维度对请求进行聚合分析 | 

- controller-runtime 的一系列指标，例如：

| 名称                                    | 类型        | 说明                  |
|---------------------------------------|-----------|---------------------|
| controller_runtime_webhook_latency_seconds                       | Histogram | Webhook处理请求时延。      |
| controller_runtime_webhook_requests_in_flight                       | Gauge     | Webhook当前正在处理的请求数量。 |
| controller_runtime_webhook_requests_total                       | Counter   | Webhook处理请求历史总数。    |


