# Architecture Context

## Overview

ArgoCD GitOps repo using the App of Apps pattern. Manages all Kubernetes workloads and infrastructure components deployed via Helm charts and raw manifests.
Infrastructure counterpart: `github.com/org-sam/terraform-aws-hands-on`

Single cluster today: `dev-demo` (us-east-2, `https://kubernetes.default.svc`).

## Sync Flow (App of Apps)

```
root-app/app.yaml (Application → bootstrap/)
├── bootstrap/projects/hands-on.yaml          AppProject (sync-wave: -1)
├── bootstrap/management/infra-manager.yaml   Application → infra-base/ (sync-wave: 0)
└── bootstrap/management/apps-manager.yaml    Application → apps/*appset.yaml (sync-wave: 1)
```

### infra-base/ (ApplicationSets — infrastructure components)

| File | Component | sync-wave | Notes |
|---|---|---|---|
| karpenter.yaml | Karpenter controller | 0 | **Has hardcoded Terraform outputs** |
| karpenter-config.yaml | EC2NodeClass + NodePool | 1 | **Has hardcoded node role name** |
| aws-load-balancer-controller.yaml | ALB Controller | — | **Has hardcoded vpc_id** |
| external-secrets.yaml | ESO operator | 0 | |
| external-secrets-config.yaml | ClusterSecretStore | 1 | |
| external-dns.yaml | ExternalDNS | 1 | |
| istio.yaml | Istio (base + istiod + gateway) | — | |
| istio-gateway-config.yaml | Istio Gateway config | — | |
| istio-telemetry-config.yaml | Istio telemetry config | — | |
| monitoring-base.yaml | Prometheus + Grafana (kube-prometheus-stack) | — | |
| monitoring-loki.yaml | Loki | — | |
| monitoring-tempo.yaml | Tempo | — | |
| monitoring-promtail.yaml | Promtail | — | |
| monitoring-open-telemetry.yaml | OTel Collector | — | **Duplicate of otel-collector.yaml** |
| otel-collector.yaml | OTel Collector | — | **Duplicate of monitoring-open-telemetry.yaml** |
| monitoring-open-telemetry-operator.yaml | OTel Operator | — | |
| kiali.yaml | Kiali | — | |
| keda.yaml | KEDA | — | |
| mongodb.yaml | MongoDB Community Operator | — | |

### apps/ (ApplicationSets — workloads)

| App | Pattern | Environments |
|---|---|---|
| bookinfo | Raw manifests (directory) | dev |
| ecommerce-catalog-service | Helm generic-chart + values per env | dev |
| ecommerce-frontend | Helm generic-chart + values per env | dev, staging, prod |

Apps use multi-source ApplicationSets: chart from `registry-1.docker.io/saiwmon/generic-chart` + values from this repo via `$values` ref.

## Values That Come From Terraform (hardcoded today)

This is the main pain point. After every `terraform apply` that changes these, they must be manually updated here:

| GitOps File | Field | Terraform Output |
|---|---|---|
| `infra-base/karpenter.yaml` | `api_endpoint` | `eks_cluster_endpoint` |
| `infra-base/karpenter.yaml` | `role_arn` | `karpenter_iam_role_arn` |
| `infra-base/karpenter.yaml` | `queue_name` | `karpenter_queue_name` |
| `infra-base/karpenter-config.yaml` | `node_role_name` | `eks_managed_node_groups_iam_role_name` |
| `infra-base/aws-load-balancer-controller.yaml` | `vpc_id` | `vpc_id` |

## Environment Configuration

```
infra-config/
├── karpenter/dev/templates/
│   ├── default-ec2nodeclass.yaml    # Uses {{ .Values.nodeRole }} and {{ .Values.clusterName }}
│   └── nodepool.yaml                # Instance families, capacity types, limits
├── external-secrets/dev/templates/
│   └── cluster-secret-store.yaml    # Uses {{ .Values.region }}
├── istio-gateway/
│   └── gateway.yaml                 # Shared gateway for *.mkt.posanhanguera.com.br
└── istio-telemetry/
    ├── telemetry.yaml               # Mesh-wide metrics + tracing (100% sampling)
    ├── servicemonitor.yaml          # Istiod metrics → Prometheus
    └── podmonitor.yaml              # Envoy sidecar metrics → Prometheus
```

## Known Issues / Tech Debt

1. **Hardcoded Terraform outputs** — see table above; manual process on every infra change
2. **Duplicate OTel Collector ApplicationSets** — `otel-collector.yaml` and `monitoring-open-telemetry.yaml` are nearly identical
3. **Hardcoded service references** — `monitoring-tempo-dev-demo.monitoring.svc:4317` in OTel config; breaks if environment name changes
4. **Hardcoded Prometheus label** — `release: monitoring-base-dev-demo` in istio-telemetry ServiceMonitor/PodMonitor
5. **AppProject too permissive** — `hands-on` project allows `*` for sourceRepos, clusterResources, and namespaceResources
6. **Missing Chart.yaml** — `infra-config/karpenter/dev/` and `infra-config/external-secrets/dev/` are used as Helm charts but may lack Chart.yaml
7. **Single AZ** — Karpenter nodepool restricted to `us-east-2a` only

## File Structure

```
infra-gitops/
├── root-app/
│   └── app.yaml                     # Entry point: Application → bootstrap/
├── bootstrap/
│   ├── projects/
│   │   └── hands-on.yaml            # AppProject definition
│   └── management/
│       ├── infra-manager.yaml        # Application → infra-base/
│       └── apps-manager.yaml         # Application → apps/*appset.yaml
├── infra-base/                       # ApplicationSets for infrastructure components
│   └── *.yaml                        # One file per component (see table above)
├── infra-config/                     # Per-environment Helm templates for infra
│   ├── karpenter/{env}/templates/
│   ├── external-secrets/{env}/templates/
│   ├── istio-gateway/
│   └── istio-telemetry/
└── apps/                             # ApplicationSets for workloads
    ├── {app}/appset.yaml
    └── {app}/values/{env}.yaml
```
