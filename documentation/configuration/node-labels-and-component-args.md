# Configure node labels and component arguments

The following command line arguments can be specified multiple times with
`key=value` pairs:

  - `--kube-kubelet-arg`
  - `--kube-proxy-arg`
  - `--kube-apiserver-arg`
  - `--kube-scheduler-arg`
  - `--kube-controller-manager-arg`
  - `--kube-cloud-controller-manager-arg`
  - `--node-label`
  - `--node-taint`

In the config file, this is done by defining a list of values for each
command like argument, for example:

```yaml
---

rke2_server:
  # Set the plugins registry directory
  kubelet-arg:
    - "volume-plugin-dir=/var/lib/rancher/rke2/agent/kubelet/plugins_registry"
  # Set the pod eviction timeout and node monitor grace period
  kube-controller-manager-arg:
    - "pod-eviction-timeout=2m"
    - "node-monitor-grace-period=30s"
  # Set API server feature gate
  kube-apiserver-arg:
    - "feature-gates=RemoveSelfLink=false"
  # Laels to apply to a node
  node-label:
    - "NodeTier=development"
    - "NodeLocation=eu-west-2a"
  # Stop rke2 control plane having workloads scheduled on them
  node-taint:
    - "rke2-controlplane=true:NoExecute"
```
