## Introduction
vci-admission 将对配置了 Label `vke.volcengine.com/burst-to-vci: enforce` 的 Pod 
自动追加 NodeSelector、Toleration 等，使得 Pod 可以调度到虚拟节点，简化了线下集群使用火山弹性容器实例（VCI）。

可以在[这里](https://www.volcengine.com/docs/6460/1159231)获取自建集群使用 VCI 方案。


## 约束限制

#### 网络约束
`volcengine-vci-admission` 需要访问火山引擎的 OpenAPI 的通用服务接入地址 open.volcengineapi.com。

- 如果您使用自建的 DNS，需要确保 `volcengine-vci-admission` 到 DNS 的网络打通。并且由于自建的 DNS 访问火山引擎的服务接入地址时返回公网 IP， 需要确保 `volcengine-vci-admission` 能访问公网。
- 如果您不希望 `volcengine-vci-admission` 访问公网，需要在火山引擎 VPC 网络内使用 `volcengine-vci-admission`。在 VPC 网络内， open.volcengineapi.com 会解析到 100 网段内的私网 IP。


## Feature
`volcengine-vci-admission` 支持 Pod 添加以下 Annotation, Label:
#### Label
| Key                             |     Value     | 说明                                                        |
|---------------------------------|:-------------:|-----------------------------------------------------------|
| vke.volcengine.com/burst-to-vci |    enforce    | 是否将 Pod 强制部署到 VCI 上。取值如下：enforce，表明创建 Pod 时将其强制部署到 VCI 上。 |

#### Annotation
| Key         |     Value     | 说明                                                            |
|-------------|:-------------:|---------------------------------------------------------------|
| vke.volcengine.com/preferred-subnet-ids |    subnet-3tispp1nai****    | 设置 VCI 实例子网。说明：<br>支持指定多个子网，但多个子网必须属于同一个可用区。多子网之间使用半角逗号（,）隔开。 |


## Deploy
参考[部署指南](docs/deploy.md)