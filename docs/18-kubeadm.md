# Deploying k8s cluster using kubeadm


```
ssh master-1

kubeadm init --control-plane-endpoint=192.168.5.30:6443 --upload-certs --api-advertise-addresses=$(hostname -i | awk '{print $3}')
```