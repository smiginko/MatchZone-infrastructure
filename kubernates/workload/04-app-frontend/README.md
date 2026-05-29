# MatchZone — Frontend Deployment

Nasadenie Angular SPA (nginx) do Kubernetes namespace `app`.

## Súbory

| Súbor | Popis |
|---|---|
| `deployment.yaml` | Deployment s nginx kontajnerom |
| `service.yaml` | ClusterIP service na porte 80 |

## Nasadenie

```sh
kubectl apply -f workload/04-app-frontend/
```

## Reštart po zmene

```sh
kubectl rollout restart deployment/matchzone-fe -n app
kubectl rollout status deployment/matchzone-fe -n app
```
