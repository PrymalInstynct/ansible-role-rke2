# IPv4 and IPv6 Dual-stack config

If you need to run your K3S cluster with both IPv4 and IPv6 address ranges
you will need to configure the `rke2_server.cluster-cidr` and
`rke2_server.service-cidr` values specifying both ranges.

:hand: if you are using `rke2<1.23` you will need to use a different CNI as
dual-stack support is not available in Flannel.

Below is a noddy example:

```yaml
---

rke2_server:
  # Using Calico on rke2<1.23 so Flannel needs to be disabled.
  flannel-backend: 'none'
  # Format: ipv4/cidr,ipv6/cidr
  cluster-cidr: 10.42.0.0/16,fc00:a0::/64
  service-cidr: 10.43.0.0/16,fc00:a1::/64
```
