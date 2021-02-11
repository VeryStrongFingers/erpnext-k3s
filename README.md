### If you'd rather blindly copy+paste to get it running ASAP, see below

Post https://verystrongfingers.github.io/erpnext/2021/01/31/erpnext-k3s.html

```bash
curl -s https://raw.githubusercontent.com/rancher/k3d/main/install.sh | bash
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```


```bash
# create k3s cluster
k3d cluster create strong-erpnext -v /opt/local-path-provisioner:/opt/local-path-provisioner
kubectl config use-context strong-erpnext

# clone 
git clone ssh://...
cd bleh
kubectl apply -f https://raw.githubusercontent.com/VeryStrongFingers/erpnext-k3s/master/pvc.yaml
helm install mariadb -n mariadb bitnami/mariadb -f maria-db-values.yaml
helm install frappe-bench-2 --namespace erpnext frappe/erpnext --version 2.0.11 -f values.yaml
kubectl apply -f site.yaml && kubectl wait --for=condition=complete job/site
kubectl apply -f ingress.yaml
```

