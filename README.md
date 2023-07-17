# K3s Cluster Installation Guide on Ubuntu Servers

This guide provides step-by-step instructions for installing a K3s cluster on Ubuntu servers. Please ensure that you have the following prerequisites before proceeding:

- 3 or more Ubuntu servers on the same network
- Private IP address for each server
- Open ports: 2379-2380, 6443, 10250
- K3S_TOKEN

## Step 1: Disable swap

```shell
sudo swapoff -a
sudo nano /etc/fstab
# Comment out the line starting with `/swap.img`
```


## Step 2: Install K3s on Master Node

1. Open a terminal on the master server.

2. Run the following command to install K3s on the master node and initialize the cluster:
```shell
curl -sfL https://get.k3s.io | K3S_TOKEN=<secret> sh -s - server --disable=traefik --tls-san <node ip>
```
Replace `<secret>` with your actual K3S_TOKEN value. This command will download and install K3s on the master node with the specified token value. The `--cluster-init` flag initializes the cluster.

## Step 3: Install K3s on Agent Nodes

1. Open a terminal on each agent server.

2. Connect other nodes [Workers]
Run the following command to install K3s on other control nodes and connect it to the master node:
```shell
curl -sfL https://get.k3s.io | K3S_TOKEN=<secret> sh -s - agent --server https://<master-ip>:6443
```
Replace `<secret>` with your actual K3S_TOKEN value and `<master-ip>` with the private IP address of your master node. This command will download and install K3s on the agent node and configure it to connect to the master node using the specified server address.

## Step 4: Cluster Access

To access and manage the Kubernetes cluster, follow one of the two methods below:

### Method 1: Export KUBECONFIG Environment Variable

1. Open a terminal on your local machine.

2. Run the following command to set the `KUBECONFIG` environment variable:
```shell
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
```
This command configures `kubectl` and other Kubernetes command-line tools to use the provided kubeconfig file for cluster access.

3. You can now use `kubectl` and other tools to interact with your K3s cluster. For example:
```shell
kubectl get pods --all-namespaces
helm ls --all-namespaces
```

### Method 2: Copy kubeconfig File

1. Copy the kubeconfig file located at `/etc/rancher/k3s/k3s.yaml` on the master node to your local machine.

2. Rename the copied file to `config` and place it in the `~/.kube/` directory on your local machine.

3. Open the `config` file using a text editor.

4. Replace the value of the `server` field with the IP address or hostname of your K3s server.

5. Save the `config` file.

6. You can now use `kubectl` and other tools to manage your K3s cluster on your local machine.

## Backup and Restore

### Creating Snapshots

Snapshots are enabled by default, at 00:00 and 12:00 system time, with 5 snapshots retained. To configure the snapshot interval or the number of retained snapshots, refer to the options.

The snapshot directory defaults to `${data-dir}/server/db/snapshots`. The `data-dir` value defaults to `/var/lib/rancher/k3s` and can be changed by setting the `--data-dir` flag.

### Restore Snapshot



On S1, start K3s with the `--cluster-reset` option, with the `--cluster-reset-restore-path` also given:
```shell
k3s server \
  --cluster-reset \
  --cluster-reset-restore-path=<PATH-TO-SNAPSHOT>
```

On S2 and S3, stop K3s. Then delete the data directory, `/var/lib/rancher/k3s/server/db/`.
```shell
systemctl stop k3s
rm -rf /var/lib/rancher/k3s/server/db/
```

On S1, start K3s again:
```shell
systemctl start k3s
```

On S2 and S3, start K3s again to join the restored cluster:
```shell
systemctl start k3s
```

### Using S3 Snapshot

Snapshot:
```shell
k3s etcd-snapshot \
  --s3 \
  --s3-bucket=<S3-BUCKET-NAME> \
  --s3-access-key=<S3-ACCESS-KEY> \
  --s3-secret-key=<S3-SECRET-KEY>
```

Restore:
```shell
k3s server \
  --cluster-init \
  --cluster-reset \
  --etcd-s3 \
  --cluster-reset-restore-path=<SNAPSHOT-NAME> \
  --etcd-s3-bucket=<S3-BUCKET-NAME> \
  --etcd-s3-access-key=<S3-ACCESS-KEY> \
  --etcd-s3-secret-key=<S3-SECRET-KEY>
```

