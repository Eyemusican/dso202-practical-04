# DSO202 Practical 4: GitOps with Argo CD

In this practical I set up Argo CD on my dso202 kind cluster and used it to deploy a small web app the GitOps way. Instead of running `kubectl apply`, I pushed Kubernetes manifests to GitHub, and Argo CD pulled them into the cluster and kept the cluster matching Git. Then I tried the three experiments from the lab guide: scaling through Git, self heal after a manual change, and pruning a deleted resource.

The full write up with screenshots is in [report/practical-04-report.md](report/practical-04-report.md).

## Two repositories

The lab guide says the GitOps repo should hold only Kubernetes manifests, so I used two repos:

- [dso202-gitops-demo](https://github.com/Eyemusican/dso202-gitops-demo): the desired state that Argo CD watches. It only has `manifests/deployment.yaml`, `manifests/service.yaml` and a README, and its commit history is the deployment history
- this repo: the Argo CD `Application` file, my screenshots and the report

## Layout

```text
dso202-practical-04/
├── argocd-app.yaml   the Argo CD Application: watches dso202-gitops-demo/manifests, deploys to demo-app
├── evidence/         screenshots
└── report/
```

## Running it

```bash
# Argo CD (server side apply avoids the ApplicationSet CRD "Too long" error)
kubectl create namespace argocd
kubectl apply -n argocd --server-side --force-conflicts \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Dashboard at https://localhost:8080 (user admin)
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Register the app
kubectl apply -f argocd-app.yaml
```
