+++
date = '2025-06-02'
draft = false
title = 'Embrace Container Orchestration with Podman'
+++

# Setting Up a k3s Cluster on Three Raspberry Pi 5 Devices

## Prerequisites

1. **Three Raspberry Pi 5** (or compatible ARM64 hardware)
2. **Raspberry Pi OS Buster or later** installed on each device
3. **Static IP addresses** for all nodes (check router/DHCP settings or use `dhcpcd` configuration files)
4. **Network connectivity**: All devices must be able to communicate with each other

## 1. Basic System Preparation

```bash
# Update system packages and reboot
sudo apt update && sudo apt upgrade -y
sudo reboot
```

After booting up, ensure:
- Swap memory is disabled (k3s works better without swap on these systems)
- Kernel parameters are set to allow high numbers of file handles
- Basic security hardening (firewall rules)

## 2. Hostname Configuration

Set unique hostnames for each Raspberry Pi:

```bash
sudo hostnamectl set-hostname controller-node
# Repeat similar command on worker nodes with appropriate names
```

Update `/etc/hosts` files to point node names to their actual IPs:

```bash
echo -e "<controller-ip>   controller-node\n<worker1-ip>   worker-node-1\n<worker2-ip>   worker-node-2" | sudo tee /etc/hosts > /dev/null
```

## 3. Disable Unnecessary Services

On each node:
```bash
sudo systemctl disable ssh --now  # Temporarily disable SSH if you'll use k3s CLI instead
sudo systemctl stop bluetooth     # Prevent conflicts with container networking
sudo systemctl stop hciuart         # Disable Bluetooth serial ports
```

## 4. Install k3s on the Controller Node

```bash
curl -sfL https://get.k3s.io | K3S_KUBECONFIG_MODE="file" sh -
# Accept default configuration options unless you need specific tuning
```

You'll be prompted to:
- Choose installation mode (`server` or `worker`)
- Set container runtime (default uses Docker)

## 5. Create Token for Worker Nodes

```bash
k3s token create --output json | jq '.data.token'
# Save this token securely on all worker nodes
```

## 6. Install k3s on Worker Nodes

On each worker node, use the controller's IP and saved token:

```bash
curl -sfL https://get.k3s.io | K3S_URL="https://controller-node-ip:6443" K3S_TOKEN="<your-token>" sh -
```

## 7. Verify Cluster Health

On any node (preferably the controller):
```bash
k3s kubectl get nodes
# Expect all three nodes to show as "Ready"
```

Check cluster components:
```bash
k3s kubectl get pods -n kube-system
k3s kubectl describe nodes
```

## 8. Persistent Storage Considerations

For reliable storage, consider adding persistent storage solutions like:
- **USB drives** formatted with ext4 (check for proper labeling)
- **Network-attached storage** using NFS or SMB
- **Raspberry Pi's built-in SD card storage**

Example USB drive configuration:

```bash
# Create directory and format first partition to ext4
sudo mkfs.ext4 /dev/sda1

# Set up auto-mounting in /etc/fstab
echo "<usb-drive>   /mnt/storage  auto  rw,relatime 0 2" | sudo tee -a /etc/fstab > /dev/null

# Create PersistentVolumeClaim template
cat <<EOF | k3s kubectl create -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pi-storage
spec:
  selector:
    matchLabels:
      pi.storage: "true"
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
EOF
```

## 9. Networking Configuration

For proper cluster communication, ensure:
- Port `6443` (Kubernetes API) is open on controller node firewall
- Port `10250` (kubelet) is accessible from workers
- Consider using a network plugin like Calico or Weave Net for better routing

## 10. Basic Cluster Management

Check cluster status:
```bash
k3s kubectl get componentstatuses
```

Access the API server directly:
```bash
curl -k https://controller-node-ip:6443/api/v1/pods/
```

Scale workloads:
```bash
k3s kubectl create deployment test --image=nginx
k3s kubectl scale deployment/test --replicas=2
```

## 11. Backup and Maintenance

Regularly back up the cluster configuration:

```bash
sudo cp -a /etc/rancher/k3s/ /root/k3s-backup-$(date +%Y-%m-%d) > /dev/null
```

On each node, check hardware health periodically using:
```bash
# Monitor system resources and temperatures
echo "vmstat every 5 minutes" | sudo tee -a /etc/crontab > /dev/null
sudo apt install sysstat -y
```

## Conclusion

We now have a Raspberry Pi cluster is now ready to run containerized workloads! While ARM-based Kubernetes clusters are suitable for edge deployments, note these limitations:

- Maximum pod density: Expect 1-2 moderately sized pods per worker node depending on hardware capabilities
- Network throughput: Hardware limitations mean prioritizing low-latency applications over high-throughput ones
- Power requirements: Ensure proper power supply to all nodes (3A USB ports recommended)