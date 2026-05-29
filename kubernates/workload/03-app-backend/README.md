# MatchZone — Backend Deployment

Nasadenie Spring Boot backendu do Kubernetes namespace `app`.

## Súbory

| Súbor | Popis |
|---|---|
| `deployment.yaml` | Deployment s env premennými (DB, Keycloak, Liquibase) |
| `service.yaml` | ClusterIP service na porte 8080 |

## Nasadenie

```sh
kubectl apply -f workload/03-app-backend/
```

## Kľúčové env premenné v deployment.yaml

| Premenná | Zdroj |
|---|---|
| `DB_URL`, `DB_USERNAME`, `DB_PASSWORD` | Secret `postgres-secret` |
| `KEYCLOAK_ADMIN_SECRET` | Secret `keycloak-secret` |
| `MATCHZONE_LIQUIBASE_ENABLED` | `"true"` — spúšťa migrácie pri štarte |
| `ISSUER_URI` | URL Keycloak realmu MatchZone |

## Reštart po zmene

```sh
kubectl rollout restart deployment/matchzone-be -n app
kubectl rollout status deployment/matchzone-be -n app
```
