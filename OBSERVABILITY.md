# Arquitetura de Observabilidade Unificada (Datadog-like)

## 🎯 Visão Geral

Arquitetura opensource que replica o modelo do Datadog: **um agente único por nó** que coleta logs, métricas e traces.

## 📊 Componentes

### **Coleta (Agent Layer)**
- **OTel Agent (DaemonSet)**: Agente único em cada nó
  - Coleta logs de containers (substitui Promtail)
  - Coleta métricas de host (substitui Node Exporter)
  - Recebe traces das aplicações
  - Coleta métricas do cluster Kubernetes

### **Processamento (Gateway Layer)**
- **OTel Gateway (Deployment)**: Processamento centralizado
  - Enriquecimento com atributos K8s
  - Sampling inteligente de traces
  - Roteamento para backends

### **Armazenamento (Backend Layer)**
- **Loki**: Logs
- **Tempo**: Traces
- **Prometheus**: Métricas

### **Visualização**
- **Grafana**: Dashboards unificados com correlação automática

### **Auto-instrumentação**
- **OTel Operator**: Injeta instrumentação automática nas aplicações

## 🔄 Fluxo de Dados

```
┌─────────────────────────────────────┐
│  Aplicações (auto-instrumentadas)   │
└──────────────┬──────────────────────┘
               │ OTLP
               ▼
┌─────────────────────────────────────┐
│  OTel Agent (DaemonSet)             │
│  • Logs (filelog receiver)          │
│  • Métricas (hostmetrics receiver)  │
│  • Traces (otlp receiver)           │
│  • K8s Metrics (k8s_cluster)        │
└──────────────┬──────────────────────┘
               │ OTLP
               ▼
┌─────────────────────────────────────┐
│  OTel Gateway (Deployment x2)       │
│  • k8sattributes processor          │
│  • tail_sampling processor          │
│  • batch processor                  │
└──────────────┬──────────────────────┘
               │
       ┌───────┼───────┐
       ▼       ▼       ▼
   [Loki]  [Tempo]  [Prometheus]
       └───────┴───────┘
               ▼
          [Grafana]
```

## 📦 Arquivos da Stack

### Ativos (Arquitetura Final)
- `monitoring-otel-agent.yaml` - DaemonSet (agente único)
- `monitoring-otel-gateway.yaml` - Gateway centralizado
- `monitoring-otel-operator.yaml` - Auto-instrumentação
- `monitoring-base.yaml` - Prometheus + Grafana (Node Exporter desabilitado)
- `monitoring-loki.yaml` - Backend de logs
- `monitoring-tempo.yaml` - Backend de traces

### Removidos (Substituídos pelo OTel Agent)
- ~~`monitoring-promtail.yaml`~~ → OTel Agent (filelog receiver)
- ~~`monitoring-open-telemetry.yaml`~~ → OTel Gateway
- ~~Node Exporter~~ → OTel Agent (hostmetrics receiver)

## 🚀 Deploy

Os componentes são gerenciados pelo ArgoCD via ApplicationSets. Para aplicar:

```bash
# Aplicar novos componentes
kubectl apply -f infra-base/monitoring-otel-agent.yaml
kubectl apply -f infra-base/monitoring-otel-gateway.yaml

# Remover componentes antigos
kubectl delete -f infra-base/monitoring-promtail.yaml
kubectl delete -f infra-base/monitoring-open-telemetry.yaml
```

## ✅ Vantagens

1. **Simplicidade**: 1 DaemonSet ao invés de 3 (Node Exporter + Promtail + OTel)
2. **Correlação**: Trace ID automático em logs e métricas
3. **Padrão**: OpenTelemetry é o padrão CNCF
4. **Recursos**: Menor footprint de CPU/RAM nos nós
5. **Vendor-neutral**: Troca backends sem mudar coleta

## 🔍 Validação

Após deploy, verificar:

```bash
# Agents rodando em todos os nós
kubectl get ds -n monitoring otel-agent

# Gateway com 2 réplicas
kubectl get deploy -n monitoring otel-gateway

# Logs chegando no Loki
kubectl logs -n monitoring -l app.kubernetes.io/name=loki

# Traces chegando no Tempo
kubectl logs -n monitoring -l app.kubernetes.io/name=tempo

# Métricas no Prometheus
kubectl port-forward -n monitoring svc/monitoring-base-dev-demo-kube-prom-prometheus 9090:9090
# Acessar: http://localhost:9090/targets
```

## 📊 Dashboards Grafana

Importar dashboards recomendados:
- **15983**: OpenTelemetry Collector
- **13639**: Kubernetes Cluster Monitoring
- **12019**: Loki & Promtail
- **16594**: Tempo Traces

## 🎯 Resultado

Arquitetura idêntica ao Datadog, 100% opensource e sob seu controle.
