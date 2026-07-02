# GitOps Argo CD Template

Scaffold này bám theo docs `poc-argocd-solo.pdf` và phần Argo CD trong `EKS_ArgoCD.pdf`.

## Design

- App of Apps: bootstrap một root Argo CD Application, root app quản lý addons và workloads.
- Kustomize base + overlays cho từng app/addon.
- Helm charts được khai báo qua Kustomize, tránh `helm install` out-of-band.
- Argo CD auto-sync, prune và self-heal được bật theo docs.

## Structure

```text
bootstrap/
  root-app.yaml
clusters/
  eks-argocd/
    kustomization.yaml
    addons/
    apps/
addons/
  argocd/
  ingress-nginx/
  cert-manager/
apps/
  backend/
  _templates/new-app/
```

## Bootstrap

Apply root app một lần sau khi Argo CD đã chạy:

```bash
kubectl apply -f bootstrap/root-app.yaml
```

Because this template uses `helmCharts` inside Kustomize, the running Argo CD instance must allow Kustomize Helm rendering. The self-managed Argo CD addon sets:

```yaml
configs:
  cm:
    kustomize.buildOptions: --enable-helm
```

If the first Argo CD install was done manually, set the same option before syncing this repo.

Root app points to:

```text
https://github.com/croyce97/eks-argocd-apps
path: clusters/eks-argocd
```

Nếu repo thực tế khác, đổi `repoURL` trong các `Application` manifests.

## Add a new app

Theo docs:

```bash
cp -r apps/_templates/new-app apps/my-app
find apps/my-app -type f | xargs sed -i '' 's/new-app/my-app/g'

cp clusters/eks-argocd/apps/backend-app.yaml \
  clusters/eks-argocd/apps/my-app-app.yaml
sed -i '' 's/backend/my-app/g' clusters/eks-argocd/apps/my-app-app.yaml

# Add my-app-app.yaml to clusters/eks-argocd/apps/kustomization.yaml
git add . && git commit -m 'feat: add my-app' && git push
```

## Known placeholders from docs

- ECR image account is set to `392423995152`; replace repository names/tags if your images use different ECR repos.
- Replace `carlos@example.com` in cert-manager issuers before production.
- Replace `api.yourdomain.com` in prod Ingress overlays with the real domain.
