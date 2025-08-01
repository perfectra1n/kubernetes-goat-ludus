# Kubernetes Goat Ludus Role

This Ludus Ansible role deploys [Kubernetes Goat](https://github.com/madhuakula/kubernetes-goat), a deliberately vulnerable Kubernetes cluster designed for security education and training. The role installs Docker, KIND (Kubernetes in Docker), creates a local Kubernetes cluster, deploys the vulnerable scenarios, and exposes them to the network.

This role is specifically designed for use with the [Ludus](https://ludus.cloud) cybersecurity lab platform and follows Ludus role conventions.

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

## Using with Ludus

### Step 1: Install the Role

Add this role to your Ludus instance using the official Ludus command:

```bash
# Clone this repository locally first
git clone https://github.com/perfectra1n/kubernetes-goat-ludus
cd kubernetes-goat-ludus

# Add the role to Ludus (the --force flag updates if already exists)
ludus ansible roles add -d ./kubernetes-goat --force
```

### Step 2: Configure Your Range

Get your current range configuration and add the Kubernetes Goat VM:

```bash
# Get current config
ludus range config get > config.yml
```

Edit your `config.yml` to include a VM with the kubernetes-goat role:

```yaml
ludus:
  - vm_name: "{{ range_id }}-k8s-goat"
    hostname: "{{ range_id }}-k8s-goat"
    template: debian-12-x64-server-template
    vlan: 20
    ip_last_octet: 10
    ram_gb: 8              # Minimum 4GB recommended
    cpus: 4                # Minimum 2 CPUs recommended  
    linux: true
    testing:
      snapshot: false
      block_internet: false  # Needs internet to download components
    roles:
      - kubernetes-goat
```

### Step 3: Deploy Your Range

Apply the configuration and deploy:

```bash
# Set the new configuration
ludus range config set -f config.yml

# Deploy the entire range
ludus range deploy

# OR deploy only user-defined roles (faster for testing)
ludus range deploy -t user-defined-roles
```

### Step 4: Access Kubernetes Goat

Once deployment completes, find your VM's IP address:

```bash
ludus range list
```

Then access Kubernetes Goat at: `http://VM_IP:1234`

### Testing and Development

When developing or testing changes to this role:

```bash
# 1. Update your local role code, then re-add to Ludus
ludus ansible roles add -d ./kubernetes-goat --force

# 2. Deploy only this role to the specific VM
ludus range deploy -t user-defined-roles --limit k8s-goat --only-roles kubernetes-goat

# 3. Monitor deployment with logs
ludus range logs -f

# 4. Check for any errors
ludus range errors

# 5. SSH to the VM for troubleshooting if needed
ludus range ssh k8s-goat
```

### Available Ludus Variables

This role can utilize these Ludus-provided variables:
- `ludus_dns_server`: DNS server IP for the VM's VLAN
- `ludus_domain_fqdn`: Full domain name
- `ludus_domain_netbios_name`: NetBIOS domain name
- `ludus_domain_fqdn_tail`: Non-NetBIOS domain portion
- `ludus_dc_vm_name`: Primary Domain Controller VM name
- `ludus_dc_ip`: Primary Domain Controller IP
- `ludus_dc_hostname`: Primary Domain Controller hostname
- `range_id`: Unique identifier for your range

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

## Contributing

Found this role useful? Have improvements? Please consider sharing:
- Email: info@badsectorlabs.com
- X (Twitter): @badsectorlabs
- Join the [Ludus Discord](https://discord.gg/ludus) community

## Author Information

This role was created by the Ludus community for integrating Kubernetes Goat into the Ludus cybersecurity lab platform. Based on the original [Kubernetes Goat](https://github.com/madhuakula/kubernetes-goat) project by Madhu Akula.
