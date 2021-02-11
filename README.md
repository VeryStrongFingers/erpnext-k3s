### If you'd rather blindly copy+paste to get it running ASAP, see below

Post https://verystrongfingers.github.io/erpnext/2021/01/31/erpnext-k3s.html

## Tooling
```bash
curl -s https://raw.githubusercontent.com/rancher/k3d/main/install.sh | bash
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash

helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add frappe https://helm.erpnext.com
helm repo update
```

```bash
# k3s cluster creation
docker volume create erpnext-persistence

k3d cluster create strong-erpnext --volume erpnext-persistence:/opt/local-path-provisioner@server[0]
kubectl config use-context strong-erpnext

# create nameservers
kubectl create ns mariadb
kubectl create ns erpnext

# pvc & required charts
kubectl apply -f https://raw.githubusercontent.com/VeryStrongFingers/erpnext-k3s/master/kube-resources/pvc.yaml
kubectl apply -f https://raw.githubusercontent.com/VeryStrongFingers/erpnext-k3s/master/kube-resources/erpnext-db-secret.yaml
helm install mariadb --namespace mariadb bitnami/mariadb --version 9.3.1 -f https://raw.githubusercontent.com/VeryStrongFingers/erpnext-k3s/master/helm-charts/maria-db-values.yaml --wait
helm install erpnext --namespace erpnext frappe/erpnext --version 2.0.11 -f https://raw.githubusercontent.com/VeryStrongFingers/erpnext-k3s/master/helm-charts/erpnext-values.yaml --wait

# erpnext site
kubectl apply --namespace erpnext  -f https://raw.githubusercontent.com/VeryStrongFingers/erpnext-k3s/master/kube-resources/create-site-job.yaml \
  && kubectl wait --namespace erpnext --for=condition=complete jobs/create-erp-site --timeout=120s

# ingress routing
kubectl apply -n erpnext -f https://raw.githubusercontent.com/VeryStrongFingers/erpnext-k3s/master/kube-resources/site-ingress.yaml
```

