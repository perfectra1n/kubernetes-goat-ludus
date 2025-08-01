# Kubernetes Goat Role

This Ansible role deploys [Kubernetes Goat](https://github.com/madhuakula/kubernetes-goat), a deliberately vulnerable Kubernetes cluster designed for security education and training. The role installs Docker, KIND (Kubernetes in Docker), creates a local Kubernetes cluster, deploys the vulnerable scenarios, and exposes them to the network.

## Requirements

- Debian-based Linux distribution (Debian, Ubuntu)
- At least 4GB RAM and 2 CPU cores recommended
- Internet access for downloading components

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

```yaml
# Kubernetes Goat configuration
kubernetes_goat_version: "master"
kubernetes_goat_repo: "https://github.com/madhuakula/kubernetes-goat.git"
kubernetes_goat_path: "/opt/kubernetes-goat"

# KIND configuration  
kind_version: "0.20.0"
kind_cluster_name: "kubernetes-goat"

# Network configuration
kubernetes_goat_bind_address: "0.0.0.0"
kubernetes_goat_main_port: 1234
kubernetes_goat_scenario_ports:
  - 1230
  - 1231
  - 1232
  - 1233
  - 1234
  - 1235
  - 1236
```

## Dependencies

None.

## Example Playbook

```yaml
- hosts: kubernetes-goat-servers
  become: true
  roles:
    - kubernetes-goat
```

## Usage in Ludus

Add the role to a Debian-based VM in your range configuration:

```yaml
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
    roles:
      - kubernetes-goat
```

## Accessing Kubernetes Goat

After deployment, Kubernetes Goat will be accessible via:

- **Main Interface**: `http://VM_IP:1234`
- **Individual Scenarios**: Ports 1230-1236

### Available Scenarios

- Port 1230: Sensitive keys in code bases
- Port 1231: DIND (docker-in-docker) exploitation scenario  
- Port 1232: SSRF in Kubernetes scenario
- Port 1233: Container escape scenario
- Port 1234: Kubernetes Goat home page (main interface)
- Port 1235: Private registry attack scenario
- Port 1236: DoS resources scenario

## Service Management

The role creates a systemd service for persistent port forwarding:

```bash
# Check service status
sudo systemctl status kubernetes-goat-portforward

# Restart service
sudo systemctl restart kubernetes-goat-portforward

# View service logs
sudo journalctl -u kubernetes-goat-portforward -f
```

## Security Warning

⚠️ **WARNING**: Kubernetes Goat contains intentionally vulnerable applications and configurations. It should NEVER be deployed in production environments or networks connected to production systems. Use only in isolated lab environments.

## Troubleshooting

### Common Issues

1. **Pods not starting**: Check if Docker service is running and user is in docker group
2. **Port forwarding fails**: Ensure all pods are in Running state before starting the service
3. **Network access issues**: Verify firewall rules allow traffic on ports 1230-1236

### Manual Commands

```bash
# Check KIND cluster status
kind get clusters

# Check Kubernetes pods
kubectl get pods --all-namespaces

# Manually start port forwarding
cd /opt/kubernetes-goat
./access-kubernetes-goat-network.sh

# Clean up cluster
kind delete cluster --name kubernetes-goat
```

## License

MIT

## Author Information

This role was created by the Ludus community for integrating Kubernetes Goat into the Ludus cybersecurity lab platform.