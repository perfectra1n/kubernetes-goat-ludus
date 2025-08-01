---
title: Kubernetes Goat
---

# Kubernetes Goat Security Lab

This guide will create a [Kubernetes Goat](https://github.com/madhuakula/kubernetes-goat) environment - a deliberately vulnerable Kubernetes cluster designed for security education and hands-on learning. The environment uses KIND (Kubernetes in Docker) to create a local cluster with multiple vulnerable scenarios.

1. Get your current range configuration:

```shell-session
#terminal-command-local
ludus range config get > config.yml
```

2. Add a Debian-based VM with the `kubernetes-goat` role to your configuration:

```yaml title="config.yml"
ludus:
  - vm_name: "{{ range_id }}-k8s-goat"
    hostname: "{{ range_id }}-k8s-goat"
    template: debian-12-x64-server-template
    vlan: 20
    ip_last_octet: 10
    ram_gb: 8
    cpus: 4
    linux: true
    testing:
      snapshot: false
      block_internet: false
    // highlight-next-line
    roles:
    // highlight-next-line
      - kubernetes-goat
```

3. Apply the configuration:

```shell-session
#terminal-command-local
ludus range config set -f config.yml
```

4. Deploy the range:

```shell-session
#terminal-command-local
ludus range deploy
```

## Accessing the Environment

Once deployed, Kubernetes Goat will be accessible via the VM's IP address. The deployment creates multiple vulnerable scenarios, each accessible on different ports:

### Main Interface
- **URL**: `http://VM_IP:1234`
- **Description**: Main Kubernetes Goat dashboard with links to all scenarios

### Individual Scenarios
- **Port 1230**: Sensitive keys in code bases
- **Port 1231**: DIND (docker-in-docker) exploitation scenario
- **Port 1232**: SSRF in Kubernetes scenario  
- **Port 1233**: Container escape scenario
- **Port 1235**: Private registry attack scenario
- **Port 1236**: DoS resources scenario

## Vulnerable Scenarios

The Kubernetes Goat environment includes multiple hands-on scenarios:

- Container escape and privilege escalation
- Kubernetes RBAC and service account abuse  
- Network policy bypass techniques
- Secrets and configuration mismanagement
- Supply chain attacks via vulnerable images
- Resource exhaustion and DoS attacks
- Server-side request forgery (SSRF) exploitation

:::warning

Kubernetes Goat contains intentionally vulnerable applications and misconfigurations. This environment should **NEVER** be deployed in production and must be isolated from production networks.

:::

## Troubleshooting

Check the port forwarding service status:

```bash
sudo systemctl status kubernetes-goat-portforward
```

View service logs:

```bash
sudo journalctl -u kubernetes-goat-portforward -f
```

Check cluster and pod status:

```bash
kind get clusters
kubectl get pods --all-namespaces
```