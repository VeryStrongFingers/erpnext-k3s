### If you'd rather blindly copy+paste to get it running ASAP, see below

Post https://verystrongfingers.github.io/erpnext/2021/01/31/erpnext-k3s.html

```bash
curl -s https://raw.githubusercontent.com/rancher/k3d/main/install.sh | bash
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```


```bash
# k3s cluster creation
k3d cluster create strong-erpnext -v /opt/local-path-provisioner:/opt/local-path-provisioner
kubectl config use-context strong-erpnext

# create nameservers
kubectl create ns mariadb
kubectl create ns erpnext

# pvc & required charts
kubectl apply -f https://raw.githubusercontent.com/VeryStrongFingers/erpnext-k3s/master/kube-resources/pvc.yaml
helm install mariadb -n mariadb bitnami/mariadb -f maria-db-values.yaml
helm install frappe-bench-2 --namespace erpnext frappe/erpnext --version 2.0.11 -f values.yaml

# erpnext site & ingress
kubectl apply -f https://raw.githubusercontent.com/VeryStrongFingers/erpnext-k3s/master/kube-resources/create-site-job.yaml && kubectl wait --for=condition=complete job/site
kubectl apply -f https://raw.githubusercontent.com/VeryStrongFingers/erpnext-k3s/master/kube-resources/site-ingress.yaml
```

