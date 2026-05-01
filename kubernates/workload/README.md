# FSA Kubernetes

- Postup inštalácie Kubernetes komponentov pre FSA Workshop.
- Školiteľ inštaluje rovnaké nástroje ako študent + navyše `GitLab`. Secrets aplikuje cez `1Password` namiesto YAML súboru.

---

## Študent

### 1. Prihlásenie a pripojenie na AKS

```sh
az login
az aks get-credentials --resource-group rg-fsa-<prefix> --name aks-fsa-<prefix> --admin
kubectl get nodes
```

---

### 2. Namespaces

```sh
kubectl apply -f workload/01-namespace.yaml
```

---

### 3. Secrets

Pred aplikovaním skontroluj hodnoty v `workload/02-secrets.yaml` — `db_url` musí odkazovať na tvoj PSQL server (`psql-fsa-<prefix>.postgres.database.azure.com`).

```sh
# Ako zakódovať hodnotu do base64
echo -n "hodnota" | base64

kubectl apply -f workload/02-secrets.yaml
```

Token pre GitLab Runner (`gitlab-runner-secret`) vyplníš neskôr v kroku **7. GitLab Runner**.

---

### 4. ingress-nginx

Uprav `helm/helm-values/ingress-nginx/override.yaml`:

- `azure-load-balancer-resource-group` → Node Resource Group tvojho AKS (`MC_fsa-<prefix>_<region>`)
- `azure-pip-name` → `pip-fsa-<prefix>`

```sh
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \
  -n ingress-nginx \
  --version 4.15.1 \
  -f helm/helm-values/ingress-nginx/override.yaml
```

---

### 5. cert-manager

```sh
helm repo add jetstack https://charts.jetstack.io
helm repo update

helm upgrade --install cert-manager jetstack/cert-manager \
  -n cert-manager \
  --version 1.20.1 \
  -f helm/helm-values/cert-manager/override.yaml

kubectl apply -f helm/helm-values/cert-manager/letsencrypt-cluster-issuer.yaml
```

---

### 6. Keycloak

Uprav `helm/helm-values/keycloak/override.yaml`:

- `database.hostname` → `psql-fsa-<prefix>.postgres.database.azure.com`

```sh
helm repo add codecentric https://codecentric.github.io/helm-charts
helm repo update

kubectl apply -f helm/helm-values/keycloak/keycloak-java-config.yaml
kubectl apply -f helm/helm-values/keycloak/realm-fsa-configmap.yaml

helm upgrade --install keycloak -n app codecentric/keycloakx \
  --version 7.1.9 \
  -f helm/helm-values/keycloak/override.yaml
```

---

### 7. GitLab Runner

Každý študent si vytvorí vlastného **Group Runner** — runner beží v jeho AKS a spracúva joby len jeho group.

Token získaš v GitLab: **tvoja Group → Build → Runners → New group runner** (tag: `fsa`).

Aktualizuj secret s tokenom:

```sh
kubectl create secret generic gitlab-runner-secret \
  --from-literal=runner-token=<TOKEN> \
  -n infra \
  --dry-run=client -o yaml | kubectl apply -f -
```

```sh
helm repo add gitlab https://charts.gitlab.io
helm repo update

helm upgrade --install gitlab-runner -n infra gitlab/gitlab-runner \
  --version 0.87.0 \
  -f helm/helm-values/gitlab-runner/override.yaml
```

---

### 8. Workload

Pred deploymentom skontroluj `workload/03-app-backend/deployment.yaml` a `workload/04-app-frontend/deployment.yaml` — ACR adresa musí odkazovať na tvoj register (`acrfsa<prefix>.azurecr.io`).

```sh
kubectl apply -f workload/03-app-backend/
kubectl apply -f workload/04-app-frontend/
kubectl apply -f workload/05-ingress/app-ingress.yaml
kubectl apply -f workload/05-ingress/keycloak-ingress.yaml
```

---

## Školiteľ

Postup je rovnaký ako u študenta s nasledujúcimi rozdielmi:

- **Prihlásenie:** `--resource-group rg-fsa --name aks-fsa`
- **Secrets:** cez `1Password` namiesto YAML — vyžaduje nainštalovaný a prihlásený `op` CLI
- **GitLab:** navyše krok **5** pred Keycloakom
- **ingress-nginx override:** `azure-load-balancer-resource-group: mc_rg-fsa`, `azure-pip-name: pip-fsa`

---

### 2. Secrets (školiteľ)

```sh
op inject -i 02-secrets.tpl.yaml | kubectl apply -f -
```

---

### 5. GitLab (školiteľ)

Vyžaduje `gitlab-secret` a `gitlab-azure-secret` z kroku 2.

```sh
helm upgrade --install fsa-gitlab -n infra \
  -f helm/helm-values/gitlab/values.yaml \
  helm/helm-charts/gitlab/1.0.0/

kubectl apply -f workload/05-ingress/gitlab-ingress.yaml
```

- GitLab štartuje prvotne ~10 minút. Stav: `kubectl logs -f -n infra deploy/fsa-gitlab`
- Po štarte: runner token získaš v GitLab → **Settings → CI/CD → Runners → New project runner** (tag: `fsa`)
