### If you'd rather blindly copy+paste to get it running ASAP, see below

Post https://verystrongfingers.github.io/erpnext/2021/02/11/erpnext-k3s.html

## Tooling
```bash
curl -s https://raw.githubusercontent.com/rancher/k3d/main/install.sh | bash
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash

helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add frappe https://helm.erpnext.com
helm repo update
```

## Cluster
```bash
# k3s cluster creation
docker volume create erpnext-persistence

k3d cluster create strong-erpnext --volume erpnext-persistence:/opt/local-path-provisioner
kubectl config use-context k3d-strong-erpnext

# create nameservers
kubectl create ns mariadb
kubectl create ns erpnext

# pvc & required charts
kubectl apply --namespace erpnext -f https://raw.githubusercontent.com/VeryStrongFingers/erpnext-k3s/master/kube-resources/pvc.yaml
kubectl apply --namespace erpnext -f https://raw.githubusercontent.com/VeryStrongFingers/erpnext-k3s/master/kube-resources/erpnext-db-secret.yaml
helm install mariadb --namespace mariadb bitnami/mariadb --version 9.3.1 -f https://raw.githubusercontent.com/VeryStrongFingers/erpnext-k3s/master/helm-charts/maria-db-values.yaml --wait
helm install erpnext --namespace erpnext frappe/erpnext --version 2.0.11 -f https://raw.githubusercontent.com/VeryStrongFingers/erpnext-k3s/master/helm-charts/erpnext-values.yaml --wait
```

## ERPNext site
```bash
# erpnext site
kubectl apply --namespace erpnext  -f https://raw.githubusercontent.com/VeryStrongFingers/erpnext-k3s/master/kube-resources/create-site-job.yaml \
  && kubectl logs --namespace erpnext -f job/create-erp-site

# ingress routing
kubectl apply --namespace erpnext -f https://raw.githubusercontent.com/VeryStrongFingers/erpnext-k3s/master/kube-resources/site-ingress.yaml
```

# Usage
```
kubectl port-forward --namespace kube-system svc/traefik 8080:80
```

Now visit http://localhost:8080 and you should be prompted with the ERPNext login page.

u: `administrator`<br />
p: `bigchungus`

---

# Configure different hostname

Being forced to `kubectl port-forward` and use `localhost` is really not ideal.

ERPNext treats the `SITE_NAME` provided on create-site job, as the fully qualified domain name of the site (which in turn resolves using HTTP `Host` header, but that's mostly irrelevant)

## Prepare your create-site job
Save the two kubernetes resources present in [ERPNext site](#erpnext-site) to your local drive.

- https://raw.githubusercontent.com/VeryStrongFingers/erpnext-k3s/master/kube-resources/create-site-job.yaml
- https://raw.githubusercontent.com/VeryStrongFingers/erpnext-k3s/master/kube-resources/site-ingress.yaml

1. Change `SITE_NAME` in `create-site-job.yaml`, to your desired hostname
2. Change `host` (`spec.rules[0].host`), to the same matching desired hostname

## Find your Clusters ingress IP
You're able to find your cluster's traefik external IP address by running `kubectl get svc --namespace kube-system` (look for External IP) - or simply
```
kubectl get svc --namespace kube-system traefik -o jsonpath='{..ip}'
```

Typically the IP address returned will be `172.27.0.2` (but may differ for several unconcerning reasons)

**This is the IP address you map the desired `SITE_NAME` to**

You can do this by Googling one of the following:
- How to setup /etc/hosts
- Setting up a DNS resolver

Once your desired arbitrary hostname resolves to the traefik External IP address, run `kubectl -n erpnext -f ...` on your local, modified save kubernetes resources from the first step.

