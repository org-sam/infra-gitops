# Infra GitOps

Este repositório gerencia a infraestrutura Kubernetes e aplicações através do ArgoCD, seguindo o padrão **App of Apps**.

## 📁 Estrutura do Projeto

```text
infra-gitops/
├── apps/           # Configurações e valores das aplicações de negócio
├── bootstrap/      # Projetos e Applications de gerenciamento (App of Apps)
├── infra-base/     # Manifestos (ApplicationSets) dos componentes de infraestrutura
├── infra-config/   # Configurações customizadas (Karpenter, Istio, External Secrets)
└── root-app/       # Ponto de entrada que inicializa todo o ecossistema
```

## 🚀 Fluxo de Deployment

```mermaid
graph TD
    A[root-app] --> B[bootstrap]
    B --> C[infra-manager]
    B --> D[apps-manager]
    C --> E[Infraestrutura Core]
    C --> F[Observabilidade]
    C --> G[Databases]
    D --> H[Aplicações]

    subgraph "Infraestrutura Core"
    E1[Karpenter] & E2[KEDA] & E3[ALB Controller] & E4[External Secrets] & E5[Istio Service Mesh]
    end

    subgraph "Observabilidade"
    F1[Grafana Stack] & F2[OpenTelemetry] & F3[Kiali]
    end
```

## 📦 Componentes de Infraestrutura

Os componentes estão organizados por categorias funcionais no diretório `infra-base/`:

- **Compute & Scaling**: [Karpenter](https://karpenter.sh/) para auto-scaling de nós e [KEDA](https://keda.sh/) para auto-scaling baseado em eventos.
- **Networking**: [AWS Load Balancer Controller](https://kubernetes-sigs.github.io/aws-load-balancer-controller/) para gerenciamento de ALBs e NLBs.
- **Service Mesh**: [Istio](https://istio.io/) (base, istiod, ingressgateway) para gerenciamento de tráfego, segurança e observabilidade de malha.
- **Security**: [External Secrets Operator](https://external-secrets.io/) para integração com AWS Secrets Manager.
- **Database**: Gerenciamento de instâncias [MongoDB](https://www.mongodb.com/) via Helm.
- **Observability**: Stack completa composta por **Grafana, Prometheus, Loki, Promtail e Tempo** (LGTM), além do **OpenTelemetry Operator/Collector** para instrumentação e coleta de telemetria. [Kiali](https://kiali.io/) para visualização da malha Istio.

## 🔧 Manutenção e Expansão

### Adicionar Novos Ambientes ou Clusters
Para adicionar um novo cluster ou ambiente, localize o `ApplicationSet` do componente desejado em `infra-base/` e atualize a lista de `generators`. O projeto ArgoCD utilizado é o `hands-on`, definido em `bootstrap/projects/`.

### Boas Práticas Implementadas
- **Sync Waves**: Ordenação automática (ex: controllers antes de configurações).
- **Auto-healing**: `prune` e `selfHeal` habilitados para evitar derivação de configuração.
- **Server-side Apply**: Utilizado para gerenciar CRDs e recursos complexos.
