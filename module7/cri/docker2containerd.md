https://kubernetes.io/zh/docs/setup/production-environment/container-runtimes/#containerd

### Stop service

```sh
systemctl stop kubelet
systemctl stop docker
systemctl stop containerd
```

### Create containerd config folder

```sh
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```

### Update default config

```sh
vi /etc/containerd/config.toml
sed -i s#k8s.gcr.io/pause:3.5#registry.aliyuncs.com/google_containers/pause:3.5#g /etc/containerd/config.toml
sed -i s#'SystemdCgroup = false'#'SystemdCgroup = true'#g /etc/containerd/config.toml

# 最新的pause容器镜像tag为3.9, registry.aliyuncs.com/google_containers/pause:3.9
```

### Edit kubelet config and add extra args. Based on v1.24 onwards version, it is not needed!!!

```sh
vi /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
Environment="KUBELET_EXTRA_ARGS=--container-runtime=remote --container-runtime-endpoint=unix:///run/containerd/containerd.sock --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.5"
```

### Restart

```sh
systemctl daemon-reload
systemctl restart containerd
systemctl restart kubelet
```

### Config crictl to set correct endpoint

```sh
cat <<EOF | sudo tee /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
EOF
```
