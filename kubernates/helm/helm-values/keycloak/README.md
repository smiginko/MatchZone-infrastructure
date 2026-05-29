# MatchZone — Keycloak (keycloakx)

Helm values pre nasadenie Keycloak 26.x na AKS pre projekt MatchZone.

Chart: [codecentric/keycloakx 7.1.9](https://github.com/codecentric/helm-charts/tree/master/charts/keycloakx)

## Štruktúra súborov

| Súbor | Popis |
|---|---|
| `override.yaml` | MatchZone-špecifické Helm values — **tu upravuj** |
| `keycloak-java-config.yaml` | ConfigMap pre opravu Azure PostgreSQL certifikátu |
| `realm-matchzone-configmap.yaml` | ConfigMap s exportom MatchZone realmu (import pri prvom štarte) |

## Prerekvizity

### Secrets (z `02-secrets.yaml`)

```sh
kubectl apply -f ../../workload/02-secrets.yaml
```

Keycloak potrebuje:

- `keycloak-secret` → `kc_password` (admin heslo)
- `postgres-secret` → `db_password` + `db_url`

### ConfigMapy

```sh
kubectl apply -f keycloak-java-config.yaml
kubectl apply -f realm-matchzone-configmap.yaml
```

> **Poznámka:** `realm-matchzone-configmap.yaml` obsahuje export MatchZone realmu.
> Keycloak ho importuje automaticky pri prvom štarte (`--import-realm`), **len ak realm v DB ešte neexistuje**.
> Pri opakovanom reštarte sa import preskočí — realm zostáva tak ako je v DB.

## Inštalácia

```sh
helm repo add codecentric https://codecentric.github.io/helm-charts
helm repo update

kubectl apply -f helm/helm-values/keycloak/keycloak-java-config.yaml
kubectl apply -f helm/helm-values/keycloak/realm-matchzone-configmap.yaml

helm upgrade --install keycloak -n app codecentric/keycloakx \
  --version 7.1.9 \
  -f helm/helm-values/keycloak/override.yaml
```

## Odinštalácia

```sh
helm uninstall keycloak -n app
```

> Po odinštalácii ostanú dáta realmu v PostgreSQL databáze `keycloak`.
> Pre čistý restart treba pred novou inštaláciou dropnúť a znovu vytvoriť databázu `keycloak`.
