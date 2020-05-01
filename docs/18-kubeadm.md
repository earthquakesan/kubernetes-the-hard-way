# Deploying k8s cluster using kubeadm

```
ssh-add ~/.vagrant.d/insecure_private_key
ssh master-1

sudo su
kubeadm init --control-plane-endpoint=192.168.5.30:6443 --upload-certs --apiserver-advertise-address=$(hostname -i | awk '{print $3}')

# Follow instructions to join master-1 and worker-1, worker-2 into the cluster, add --apiserver-advertise-address flag to the commands
# For example

# On master-2

kubeadm join 192.168.5.30:6443 --token 8z1nae.up1t4yza6frd018t \
    --apiserver-advertise-address=$(hostname -i | awk '{print $3}') \
    --discovery-token-ca-cert-hash sha256:d9916580f425bea255f61c2237a012d9e40bf742ea4b992d60150b0cfb698b06 \
    --control-plane --certificate-key a3ec4fd28d8888608397b2ba871bac79adfc68510158dc0b48d1c8f476d66efb

# On worker-1, worker-2

kubeadm join 192.168.5.30:6443 --token 8z1nae.up1t4yza6frd018t \
    --apiserver-advertise-address=$(hostname -i | awk '{print $3}') \
    --discovery-token-ca-cert-hash sha256:d9916580f425bea255f61c2237a012d9e40bf742ea4b992d60150b0cfb698b06


# On master-1 check that the nodes are there

vagrant@master-1:~$ kubectl get nodes
NAME       STATUS     ROLES    AGE     VERSION
master-1   NotReady   master   2m57s   v1.18.2
master-2   NotReady   master   51s     v1.18.2
worker-1   NotReady   <none>   13s     v1.18.2
worker-2   NotReady   <none>   12s     v1.18.2

# Deploy WeaveNet (on master-1)

kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

# Check that everything is running

kubectl -n kube-system get po -w

# On your local machine run tests on the cluster

go get -u k8s.io/test-infra/kubetest
go get github.com/onsi/ginkgo/ginkgo
go get github.com/onsi/gomega/...

-- Install google SDK from here: https://cloud.google.com/sdk/docs/downloads-versioned-archives
gcloud auth

kubetest --extract=v1.18.2
cd kubernetes

scp master-1:/home/vagrant/.kube/config ./
# has to be absolute path to the config file
export KUBECONFIG=$(pwd)/config
export KUBE_MASTER_IP=192.168.5.30:6443
export KUBE_MASTER=master-1
export KUBERNETES_PROVIDER=skeleton
export KUBE_MASTER_URL=https://192.168.5.30:6443
kubetest --test --test_args="--ginkgo.focus=\[Conformance\]" > test.out
```

## Notes

The cluster setup does not tolerate restart (sic!)