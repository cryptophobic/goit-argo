# goit-argo

GitOps-репозиторій із маніфестами, які підхоплює **ArgoCD**, розгорнутий у EKS
(див. основний репозиторій із Terraform-кодом ArgoCD).

## Структура

```text
goit-argo
├── application.yaml              # ArgoCD Application: Helm-деплой MLflow
├── namespaces
│   ├── application
│   │   ├── ns.yaml               # Namespace application
│   │   └── nginx.yaml            # додатковий легкий smoke-test застосунок
│   └── infra-tools
│       └── ns.yaml               # Namespace infra-tools (де живе ArgoCD)
├── values
│   └── mlflow-values.yaml        # довідкова копія overrides для MLflow
└── README.md
```

## Що деплоїться

`application.yaml` описує ArgoCD `Application`, який тягне Helm-чарт **MLflow**
(`community-charts/mlflow`, версія `1.8.1`) з ArtifactHub і розгортає його в
namespace `application`:

- `repoURL: https://community-charts.github.io/helm-charts`
- `chart: mlflow`, `targetRevision: 1.8.1`
- overrides — **inline** (`spec.source.helm.values`)
- `syncPolicy.automated`: `prune: true`, `selfHeal: true`
- `syncOptions: [CreateNamespace=true]`

## Як застосувати

ArgoCD має бути вже встановлений у namespace `infra-tools` (через Terraform).
Зареєструйте Application у кластері:

```bash
kubectl apply -f application.yaml
```

Далі auto-sync + self-heal самі підтримують MLflow у синхронному стані.
Перевірка:

```bash
kubectl get applications -n infra-tools
kubectl get pods -n application
```

## Доступ до MLflow

```bash
kubectl port-forward svc/mlflow -n application 5001:80
# відкрити http://localhost:5001
```