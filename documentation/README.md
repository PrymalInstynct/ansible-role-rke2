# ansible-role-rke2

This document describes a number of ways of consuming this Ansible role for use
in your own rke2 deployments. It will not be able to cover every use case
scenario but will provide some common example configurations.

## Requirements

Before you start you will need an Ansible controller. This can either be your
workstation, or a dedicated system that you have access to. The instructions
in this documentation assume you are using `ansible` CLI, there are no
instructions available for Ansible Tower at this time.

Follow the below guide to get Ansible installed.

https://docs.ansible.com/ansible/latest/installation_guide/index.html

## Quickstart

Below are quickstart examples for a single node rke2 server, a rke2 cluster
with a single control node and HA rke2 cluster. These represent the bare
minimum configuration.

  - [Single node rke2](quickstart-single-node.md)
  - [Simple rke2 cluster](quickstart-cluster.md)
  - [HA rke2 cluster using embedded etcd](quickstart-ha-cluster.md)

## Example configurations and operations

### Configuration

  - [Setting up 2-node HA control plane with external datastore](configuration/2-node-ha-ext-datastore.md)
  - [Provision multiple standalone rke2 nodes](configuration/multiple-standalone-rke2-nodes.md)
  - [Set node labels and component arguments](configuration/node-labels-and-component-args.md)
  - [Use an alternate CNI](configuration/use-an-alternate-cni.md)
  - [IPv4/IPv6 Dual-Stack config](configuration/ipv4-ipv6-dual-stack.md)
  - [Start K3S after another service](configuration/systemd-config.md)

### Operations

  - [Stop/Start a cluster](operations/stop-start-cluster.md)
  - [Updating rke2](operations/updating-rke2.md)
  - [Extending a cluster](operations/extending-a-cluster.md)
  - [Shrinking a cluster](operations/shrinking-a-cluster.md)
