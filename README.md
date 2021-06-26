# k8s home lab

## install k3s

server

```bash
curl -sfL https://get.k3s.io | sh -
```

worker nodes (the value to use for K3S_TOKEN=mynodetoken is stored at /var/lib/rancher/k3s/server/node-token on the server node)

```bash
curl -sfL https://get.k3s.io | K3S_URL=https://serverip:6443 K3S_TOKEN=mynodetoken K3S_NODE_NAME=raspi4 sh -
```

## install dashboard

*installing it with helm didn't go well...*

```bash
GITHUB_URL=https://github.com/kubernetes/dashboard/releases
VERSION_KUBE_DASHBOARD=$(curl -w '%{url_effective}' -I -L -s -S ${GITHUB_URL}/latest -o /dev/null | sed -e 's|.*/||')
sudo kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/${VERSION_KUBE_DASHBOARD}/aio/deploy/recommended.yaml

sudo kubectl apply -f yaml/admin-user.yml
sudo kubectl apply -f yaml/admin-user-role.yml

sudo kubectl -n kubernetes-dashboard \
    get secret $(sudo kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") \
    -o go-template="{{.data.token | base64decode}}"

sudo kubectl proxy
# http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

## install helm

*grr - can't get helm to work without specifying `--kubeconfig` on the command line...*

```bash
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
sudo apt-get install apt-transport-https --yes
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm   

export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
sudo helm ls --all-namespaces --kubeconfig $KUBECONFIG
```

## uninstall k3s

server

```bash
/usr/local/bin/k3s-uninstall.sh
```

worker nodes

```bash
/usr/local/bin/k3s-agent-uninstall.sh
```
