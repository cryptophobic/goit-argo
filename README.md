# goit-argo — GitOps маніфести для ML-OPS-CI-CD-classes (lesson-8-9)

Цей репозиторій — джерело правди для ArgoCD у кластері EKS, який піднімається з
Terraform у [ML-OPS-CI-CD-classes](https://github.com/Cryptophobic/ML-OPS-CI-CD-classes).

## Структура

```
.
├── root-app.yaml                       # App-of-Apps — apply один раз
├── apps/
│   ├── postgres.yaml                   # Application → manifests/postgres/
│   ├── minio.yaml                      # Application → charts.min.io/minio
│   ├── mlflow.yaml                     # Application → community-charts/mlflow
│   ├── pushgateway.yaml                # Application → prom-comm/pushgateway
│   └── kube-prometheus-stack.yaml      # Application → prom-comm/kps
└── manifests/
    └── postgres/                       # Plain k8s маніфести для Postgres
        ├── secret.yaml
        ├── service.yaml
        └── deployment.yaml
```

## Bootstrap

1. У основному репо: `terraform apply` (root) → `terraform -chdir=terraform/argocd apply`.
2. Зареєструвати root-Application:
   ```bash
   kubectl apply -n infra-tools -f root-app.yaml
   ```
3. ArgoCD створить 5 дочірніх Application: postgres, minio, mlflow, pushgateway,
   kube-prometheus-stack. Auto-sync + self-heal — увімкнено.

## Namespace layout

| Namespace      | Що живе                                                |
|----------------|--------------------------------------------------------|
| `infra-tools`  | ArgoCD (Helm release від Terraform); Application CRs   |
| `mlflow`       | Postgres, MinIO, MLflow Tracking Server                |
| `monitoring`   | Prometheus, Grafana, PushGateway, kube-state-metrics   |

## Service DNS

| Сервіс      | DNS                                                                 |
|-------------|---------------------------------------------------------------------|
| Postgres    | `postgres.mlflow.svc.cluster.local:5432`                            |
| MinIO       | `minio.mlflow.svc.cluster.local:9000` (API), `:9001` (console)      |
| MLflow      | `mlflow.mlflow.svc.cluster.local:5000`                              |
| PushGateway | `prometheus-pushgateway.monitoring.svc.cluster.local:9091`          |
| Grafana     | `kube-prometheus-stack-grafana.monitoring.svc.cluster.local:80`     |
| Prometheus  | `kube-prometheus-stack-prometheus.monitoring.svc.cluster.local:9090`|