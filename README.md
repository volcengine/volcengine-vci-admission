# volcengine-vci-admission
English | [中文](./README_zh.md)

## Introduction
vci-admission allows the Pods configured with the Label `vke.volcengine.com/burst-to-vci: enforce` to be scheduled to virtual nodes 
by appending `spec.NodeSelector` and `spec.Toleration`. This simplifies the use of Volcengine Elastic Container Instance (VCI) in on-premises clusters.

See [Using VCI in Self-built Clusters](https://www.volcengine.com/docs/6460/1159231) for more details.

## Prerequisites
### Network requirement
`volcengine-vci-admission` need to visit `open.volcengineapi.com`, which is the OpenAPI address of Volcano Engine.
- The network from `volcengine-vci-admission` to DNS needs to be reachable if you use a self-built DNS. And because the self-built DNS returns the public IP when visiting the OpenAPI of Volcano Engine, you need to ensure that `volcengine-vci-admission` can access the public network.
- If you do not want `volcengine-vci-admission` to access the public network, you need to use `volcengine-vci-admission` in the Volcano Engine VPC network. And `open.volcengineapi.com` will be resolved to a private IP in the 100 network segment.

## Feature
vci-admission supports appending the following Annotations and Labels to Pod:

### Label
| Key                             |     Value     | Description                                                                                                            |
|---------------------------------|:-------------:|---------------------------------------------------------------------------------------------------------------|
| vke.volcengine.com/burst-to-vci |    enforce    | Whether to deploy the Pod to the VCI. Values: enforce, indicating that the pod is deployed to the VCI. |

### Annotation
| Key         |     Value     | Description                                                                               |
|-------------|:-------------:|----------------------------------------------------------------------------------|
| vke.volcengine.com/preferred-subnet-ids |    subnet-3tispp1nai****    | Set the subnet used by the Pod.Note:<br>Supports specifying multiple subnets, but all subnets must belong to the same availability zone. Use commas (,) to separate multiple subnets. |

## How to deploy
See the [Deployment Guide](docs/deploy.md)
