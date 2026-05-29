# MatchZone — Kubernetes Workload

Nasadenie aplikačných komponentov MatchZone do AKS namespace `app`.

---

## Prerekvizity

```sh
az login
az aks get-credentials --resource-group rg-fsa-ziaciks --name aks-fsa-ziaciks --admin
kubectl get nodes
```

---

## Postup nasadenia

### 1. Namespaces

```sh
kubectl apply -f 01-namespace.yaml
```

### 2. Secrets

Pred aplikovaním skontroluj a vyplň hodnoty v `02-secrets.yaml` (base64 zakódované):

```sh
# Zakódovanie hodnoty do base64
echo -n "hodnota" | base64

kubectl apply -f 02-secrets.yaml
```

Potrebné secret keys:

| Secret | Key | Popis |
|---|---|---|
| `postgres-secret` | `db_url` | hostname PostgreSQL servera |
| `postgres-secret` | `db_username` | DB username |
| `postgres-secret` | `db_password` | DB heslo |
| `keycloak-secret` | `kc_password` | Keycloak admin heslo |
| `keycloak-secret` | `keycloak_admin_secret` | Secret pre `matchzone-backend-sa` client |

### 3. Backend

```sh
kubectl apply -f 03-app-backend/
```

### 4. Frontend

```sh
kubectl apply -f 04-app-frontend/
```

### 5. Ingress

```sh
kubectl apply -f 05-ingress/app-ingress.yaml
kubectl apply -f 05-ingress/keycloak-ingress.yaml
```

---

## Užitočné príkazy

```sh
# Stav podov
kubectl get pods -n app

# Logy backendu
kubectl logs -f deployment/matchzone-be -n app

# Reštart backendu (napr. po zmene secrets)
kubectl rollout restart deployment/matchzone-be -n app

# Reštart frontendu
kubectl rollout restart deployment/matchzone-fe -n app
```
