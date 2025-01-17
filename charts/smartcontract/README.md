# smartcontract

![Version: 0.4.2](https://img.shields.io/badge/Version-0.4.2-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: latest](https://img.shields.io/badge/AppVersion-latest-informational?style=flat-square)

A Helm chart for deploying the Smartcontract on a goquorum node for a given ETH account.
The anchoring info (address and Abi) of the Smartcontract will be stored in a Kubernetes ConfigMap to make them usable by other components running on Kubernetes.

## Requirements

- [helm 3](https://helm.sh/docs/intro/install/)
- These values:

  | Configuration<br/>value | Description | Sandbox | Non-Sandbox<br/>connected blockchain |
  |-------------------------|:-----------:|:-------:|:------------------------------------:|
  | `config.quorumNodeAddress`<br/>and `config.quorumNodePort` | Address of<br/>Quorum node | **Not required**<br/>Defaults to first node of<br/>helm chart *standalone-quorum* | **Required** |
  | `config.account` | ETH account address<br/>of preexisting and unlocked account. | **Not required**<br/>Defaults to an account provided<br/>by helm chart *standalone-quorum* | **Required** |

## Changelog

- From 0.3.x to 0.4.x
  - Creates *info* in ConfigMap as JSON-input for deploying and auto-configuring *ethadapter* in Sandbox installation (for use with ethadapter helm chart >= `0.7.4`)
  - ConfigMap for storing abi, address and info has been renamed from *smartcontract-anchoring-info* to *smartcontractinfo*.

- From 0.2.x to 0.3.x
  - Uses `pharmaledger/anchor_smart:latest` image for SmartContract anchoring (`image.repository` and `image.tag`) which is compatible to epi application v1.1.x or higher. Not compatible with epi v1.0.x !
  - An preexisting ETH Account that you own needs must be provided. A new ETH account will *NOT* be created anymore.
  - Therefore no secret with OrgAccount data will be created anymore.
  - `config.quorumNodeUrl` has been replaced by `config.quorumNodeAddress` and `config.quorumNodePort`.
  - `config.account` defaults to an ETH account created by helm chart *standalone-quorum*.
  - `config.anchoringSC` has been removed. The SmartContract definition is now part of the container image.

- From 0.1.x to 0.2.x
  - Value `config.quorumNodeUrl` has changed from `http://quorum-member1.quorum:8545` to `http://quorum-validator1.quorum:8545`.
  This reflects the change of chart [standalone-quorum](https://github.com/pharmaledgerassoc/helmchart-ethadapter/tree/standalone-quorum-0.2.0/charts/standalone-quorum#changelog) from version 0.1.x to 0.2.x where member nodes are not enabled by default.

## Usage

- [Here](./README.md#values) is a full list of all configuration values.
- The [values.yaml file](./values.yaml) shows the raw view of all configuration values.

## How it works

1. A Kubernetes Job will be deployed which triggers scheduling of a Pod
2. The Pod compiles the SmartContract and anchors it with the configure ETH account on the Quorum Blockchain.
3. Then the SmartContract address, its Abi and a JSON object called *info* (which contains both address and abi) will be stored in a Kubernetes ConfigMap.

![How it works](./docs/smartcontract.drawio.png)

Note: Persisting these values in Kubernetes ConfigMap enables passing values and easier configuration of *ethadapter* on a Sandbox environment.

## Installing the Chart

### Sandbox installation

**IMPORTANT** On a sandbox environment, install into the same namespace as *ethadapter* (usually namespace `ethadapter`). Otherwise *ethadapter* cannot auto-configure itself by reading values from Secret and ConfigMap.

```bash
helm upgrade --install smartcontract ph-ethadapter/smartcontract \
  --version=0.4.2 \
  --namespace=ethadapter --create-namespace \
  --wait --wait-for-jobs \
  --timeout 10m

```

### Non-Sandbox installation

**Note:** An installation on non-Sandbox is usually not required as the SmartContract is already deployed on connected Blockchain network.

```bash
helm upgrade --install smartcontract ph-ethadapter/smartcontract \
  --version=0.4.2 \
  --namespace=ethadapter --create-namespace \
  --wait --wait-for-jobs \
  --timeout 10m \
  --set config.quorumNodeAddress="mynode.company.com" \
  --set config.quorumNodePort=5432 \
  --set config.account="0x1234567890abcdef"
```

## Uninstalling the Chart

To uninstall/delete the `smartcontract` deployment:

```bash
helm delete smartcontract \
  --namespace=ethadapter

```

## Security

Unfortunately the container image as of 2022 May-18 (`pharmaledger/anchor_smart:latest@sha256:6c146032888e99090200763e9479fd832aba36c5cc57859df521131fe913d731`) does not allow running as not root user.

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| tgip-work |  | <https://github.com/tgip-work> |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| affinity | object | `{}` | Affinity for scheduling a pod. See [https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/) |
| config.account | string | `"0xb5ced4530d6ccbb31b2b542fd9b4558b52296784"` | Existing account on Blockchain network Defaults to the predefined account from node 'quorum-validator1' deployed by helm chart 'standalone-quorum' |
| config.configMapAnchoringInfoName | string | `"smartcontractinfo"` | Name of the ConfigMap with the anchoring info. If empty uses a generic name |
| config.quorumNodeAddress | string | `"quorum-validator1.quorum"` | DNS Name or IP Address of Quorum node. Defaults to first Quorum node provided by helm chart 'standalone-quorum' on a Sandbox environment. |
| config.quorumNodePort | string | `"8545"` | Port of Quorum Node endpoint |
| fullnameOverride | string | `""` | fullnameOverride completely replaces the generated name. From [https://stackoverflow.com/questions/63838705/what-is-the-difference-between-fullnameoverride-and-nameoverride-in-helm](https://stackoverflow.com/questions/63838705/what-is-the-difference-between-fullnameoverride-and-nameoverride-in-helm) |
| image.pullPolicy | string | `"Always"` | Image Pull Policy of the node container |
| image.repository | string | `"pharmaledger/anchor_smart"` | The repository of the container image which deploys the Smart Contract |
| image.sha | string | `"6c146032888e99090200763e9479fd832aba36c5cc57859df521131fe913d731"` | sha256 digest of the image. Do not add the prefix "@sha256:" Default to image digest for version latest - see [dockerhub](https://hub.docker.com/layers/pharmaledger/anchor_smart/latest/images/sha256-6c146032888e99090200763e9479fd832aba36c5cc57859df521131fe913d731?context=explore) <!-- # pragma: allowlist secret --> |
| image.tag | string | `"latest"` | The tag of the container image which deploys the Smart Contract |
| imagePullSecrets | list | `[]` | Secret(s) for pulling an container image from a private registry. See [https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/) |
| kubectlImage.pullPolicy | string | `"Always"` | Image Pull Policy |
| kubectlImage.repository | string | `"bitnami/kubectl"` | The repository of the container image which creates configmap and secret |
| kubectlImage.sha | string | `"bba32da4e7d08ce099e40c573a2a5e4bdd8b34377a1453a69bbb6977a04e8825"` | sha256 digest of the image. Do not add the prefix "@sha256:" <br/> Defaults to image digest for "bitnami/kubectl:1.21.14", see [https://hub.docker.com/layers/kubectl/bitnami/kubectl/1.21.14/images/sha256-f9814e1d2f1be7f7f09addd1d877090fe457d5b66ca2dcf9a311cf1e67168590?context=explore](https://hub.docker.com/layers/kubectl/bitnami/kubectl/1.21.14/images/sha256-f9814e1d2f1be7f7f09addd1d877090fe457d5b66ca2dcf9a311cf1e67168590?context=explore) <!-- # pragma: allowlist secret --> |
| kubectlImage.tag | string | `"1.21.14"` | The Tag of the image containing kubectl. Minor Version should match to your Kubernetes Cluster Version. |
| nameOverride | string | `""` | nameOverride replaces the name of the chart in the Chart.yaml file, when this is used to construct Kubernetes object names. From [https://stackoverflow.com/questions/63838705/what-is-the-difference-between-fullnameoverride-and-nameoverride-in-helm](https://stackoverflow.com/questions/63838705/what-is-the-difference-between-fullnameoverride-and-nameoverride-in-helm) |
| namespaceOverride | string | `""` | Override the deployment namespace. Very useful for multi-namespace deployments in combined charts |
| nodeSelector | object | `{}` | Node Selectors in order to assign pods to certain nodes. See [https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/) |
| podSecurityContext | object | `{"fsGroup":0,"runAsGroup":0,"runAsUser":0}` | Security Context for the pod. See [https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#set-the-security-context-for-a-pod](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#set-the-security-context-for-a-pod) |
| resources | object | `{"limits":{"cpu":"500m","memory":"1Gi"},"requests":{"cpu":"50m","memory":"1Gi"}}` | Resource constraints for each container |
| securityContext | object | `{"allowPrivilegeEscalation":false,"capabilities":{"drop":["ALL"]},"readOnlyRootFilesystem":false,"runAsGroup":0,"runAsNonRoot":false,"runAsUser":0}` | Security Context for the container. See [https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#set-the-security-context-for-a-container](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#set-the-security-context-for-a-container) |
| serviceAccount.annotations | object | `{}` |  |
| serviceAccount.create | bool | `true` |  |
| serviceAccount.name | string | `""` |  |
| tolerations | list | `[]` | Tolerations for scheduling a pod. See [https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/) |

----------------------------------------------
Autogenerated from chart metadata using [helm-docs v1.8.1](https://github.com/norwoodj/helm-docs/releases/v1.8.1)
