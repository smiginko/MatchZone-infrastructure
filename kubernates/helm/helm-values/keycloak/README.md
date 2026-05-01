# FSA Keycloak (keycloakx)

- https://github.com/codecentric/helm-charts/tree/master/charts/keycloakx

## Štruktúra súborov

| Súbor | Popis |
|---|---|
| `values-default.yaml` | Kompletné default values z helm chart 7.1.9 (referencia, nemeniť) |
| `override.yaml` | Len FSA-špecifické zmeny — **tu upravuj hodnoty** |
| `keycloak-java-config.yaml` | ConfigMap pre opravu Azure PostgreSQL certifikátu (aplikuj pred inštaláciou) |
| `realm-fsa-configmap.yaml` | ConfigMap s FSA realm exportom — master + FSA realm (aplikuj pred inštaláciou) |

## Prerekvizity

### 1. Secrets (z `02-secrets.yaml`)

```bash
kubectl apply -f ../../02-secrets.yaml
```

Secrets, ktoré Keycloak potrebuje:

- `keycloak-secret` → `kc_password` (admin heslo, default: `admin`)
- `postgres-secret` → `db_password` (PSQL heslo `P@ssword12345!`) + `db_url`

### 2. ConfigMapy pred inštaláciou

```bash
# Fix Azure PostgreSQL deprecated certifikat (SHA1 cert)
kubectl apply -f keycloak-java-config.yaml

# FSA realm — obsahuje master realm s admin userom + FSA realm so studentmi
kubectl apply -f realm-fsa-configmap.yaml
```

> **Dôležité:** `realm-fsa-configmap.yaml` je generovaný z
> `FSA-onion-architecture/.keycloak/realms/realm-fsa.json`.
> Obsahuje **master realm** s admin userom (heslo: `admin`) a **FSA realm**
> so štyrmi usermi (admin@posam.sk, student@posam.sk, teacher@posam.sk, service-account).
> `KC_BOOTSTRAP_ADMIN_*` env vars sa **nepouživajú** — admin je definovaný v realm JSON.

## Inštalácia

```bash
helm repo add codecentric https://codecentric.github.io/helm-charts
helm repo update

kubectl apply -f keycloak-java-config.yaml
kubectl apply -f realm-fsa-configmap.yaml

helm upgrade --install keycloak -n app codecentric/keycloakx \
  --version 7.1.9 \
  -f override.yaml
```

## Odinštalácia

```bash
helm uninstall keycloak -n app
```

> **Po odinštalácii:** KC data ostanú v PSQL databáze. Pre čistý restart treba dropnúť
> a znovu vytvoriť `keycloak` databázu pred ďalšou inštaláciou.
