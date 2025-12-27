# Bookinfo Application

Aplicação de exemplo do Istio composta por 4 microserviços:

- **productpage** (Python): Frontend que chama os demais serviços
- **details** (Ruby): Informações sobre o livro
- **reviews** (Java): Avaliações do livro (3 versões)
- **ratings** (Node.js): Classificação por estrelas

## Observabilidade

Serviços instrumentados com OpenTelemetry:
- ✅ **productpage** (Python), **reviews** (Java), **ratings** (Node.js)
- ❌ **details** (Ruby) - auto-instrumentação não suportada
- Traces enviados para `otel-collector.monitoring.svc:4318`
- Sampling rate: 100%

## Acesso

```bash
# Port-forward para acessar localmente
kubectl port-forward -n bookinfo svc/productpage 9080:9080
```

Acesse: http://localhost:9080/productpage
