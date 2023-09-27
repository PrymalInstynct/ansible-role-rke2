# systemd config

Below are examples to tweak how and when RKE2 starts up.

## Wanted service units

In this example, we're going to start RKE2 after Wireguard. Our example server
has a Wireguard connection `wg0`. We are using "wants" rather than "requires"
as it's a weaker requirement that Wireguard must be running. We then want
RKE2 to start after Wireguard has started.

```yaml
---

rke2_service_wants:
  - wg-quick@wg0.service
rke2_service_after:
  - wg-quick@wg0.service
```
