# MatchZone — Infrastructure

Kubernetes infraštruktúra pre projekt MatchZone. Zabezpečuje nasadenie backendu, frontendu a Keycloaku na Azure Kubernetes Service (AKS) s verejným HTTPS prístupom.

---

## Čo je nasadené

| Komponent | Technológia | URL |
|-----------|-------------|-----|
| Frontend | Angular 21 + nginx | https://app.20.234.18.69.nip.io |
| Backend | Spring Boot 4 + Java 25 | https://app.20.234.18.69.nip.io/api/v1 |
| Keycloak | KC 26.5.5 (realm: MatchZone) | https://keycloak.20.234.18.69.nip.io/auth |
| Databáza | Azure PostgreSQL (db: matchzone) | psql-fsa-ziaciks.postgres.database.azure.com |

---

## Architektúra

```
Internet
   │
   ▼
ingress-nginx (LoadBalancer, pip-fsa-ziaciks)
   │
   ├── app.20.234.18.69.nip.io
   │     ├── /        → matchzone-fe (nginx, port 80)
   │     ├── /api     → matchzone-be (Spring Boot, port 8080)
   │     └── /auth    → fsa-keycloak-http (Keycloak, port 80)
   │
   └── keycloak.20.234.18.69.nip.io
         └── /        → fsa-keycloak-http (Keycloak, port 80)

cert-manager → Let's Encrypt (nip.io domény)
```

---

## Štruktúra repozitára

```
kubernates/
├── helm/
│   └── helm-values/
│       ├── cert-manager/       # Let's Encrypt ClusterIssuer
│       ├── ingress-nginx/      # Azure Load Balancer konfigurácia
│       ├── keycloak/           # Keycloak Helm values + realm import
│       └── gitlab-runner/      # GitLab CI/CD runner
└── workload/
    ├── 01-namespace.yaml       # Namespacey: app, infra, ingress-nginx, cert-manager
    ├── 02-secrets.yaml         # DB a Keycloak credentials
    ├── 03-app-backend/         # Backend Deployment + Service
    ├── 04-app-frontend/        # Frontend Deployment + Service
    └── 05-ingress/             # Ingress pravidlá pre všetky služby
```

---

## Postup inštalácie

### Prerekvizity
- `kubectl` pripojený na AKS cluster
- `helm` nainštalovaný
- `az login` hotový

### 1. Namespaces a Secrets
```sh
kubectl apply -f workload/01-namespace.yaml
kubectl apply -f workload/02-secrets.yaml
```

### 2. ingress-nginx
```sh
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \
  -n ingress-nginx --version 4.15.1 \
  -f helm/helm-values/ingress-nginx/override.yaml
```

### 3. cert-manager
```sh
helm repo add jetstack https://charts.jetstack.io
helm upgrade --install cert-manager jetstack/cert-manager \
  -n cert-manager --version 1.20.1 \
  -f helm/helm-values/cert-manager/override.yaml
kubectl apply -f helm/helm-values/cert-manager/letsencrypt-cluster-issuer.yaml
```

### 4. Keycloak
```sh
helm repo add codecentric https://codecentric.github.io/helm-charts
kubectl apply -f helm/helm-values/keycloak/keycloak-java-config.yaml
kubectl apply -f helm/helm-values/keycloak/realm-matchzone-configmap.yaml
helm upgrade --install keycloak -n app codecentric/keycloakx \
  --version 7.1.9 -f helm/helm-values/keycloak/override.yaml
```

### 5. Workload
```sh
kubectl apply -f workload/03-app-backend/
kubectl apply -f workload/04-app-frontend/
kubectl apply -f workload/05-ingress/app-ingress.yaml
kubectl apply -f workload/05-ingress/keycloak-ingress.yaml
```

### 6. GitLab Runner
```sh
helm repo add gitlab https://charts.gitlab.io
helm upgrade --install gitlab-runner -n infra gitlab/gitlab-runner \
  --version 0.87.0 -f helm/helm-values/gitlab-runner/override.yaml
```
