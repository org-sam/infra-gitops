# Bookinfo - Aplicação de Teste Istio

Aplicação de demonstração do Istio composta por 4 microsserviços.

## 🏗️ Arquitetura

```
productpage (frontend) → details
                      ↘
                        reviews (v1/v2/v3) → ratings
```

### Microsserviços

- **productpage** (Frontend): Chama details e reviews para popular a página
- **details**: Contém informações do livro
- **reviews**: Contém avaliações do livro e chama ratings
  - v1: Não chama ratings
  - v2: Chama ratings e exibe estrelas pretas (1-5)
  - v3: Chama ratings e exibe estrelas vermelhas (1-5)
- **ratings**: Contém ranking que acompanha as avaliações

## 📁 Estrutura Organizada

```
manifests/
├── productpage/          # Frontend (ponto de entrada)
│   ├── deployment.yaml
│   ├── gateway.yaml      # Expõe aplicação externamente
│   └── virtualservice.yaml
├── details/
│   └── deployment.yaml
├── reviews/              # Serviço com múltiplas versões
│   ├── deployment.yaml   # v1, v2, v3
│   ├── destinationrule.yaml  # Define subsets por versão
│   └── virtualservice.yaml   # Roteamento entre versões
├── ratings/
│   └── deployment.yaml
└── instrumentation.yaml  # OpenTelemetry (compartilhado)
```

## 🔀 Roteamento Istio

### Gateway (productpage)
- Expõe `bookinfo.mkt.posanhanguera.com.br` na porta 80
- Ponto de entrada externo da aplicação

### VirtualService Reviews
- **URI `/v1/*`** → reviews v1
- **Header `x-version: v2`** → reviews v2
- **URI `/v3/*`** → reviews v3
- **Default**: Distribuição 34/33/33 entre v1/v2/v3

### DestinationRule Reviews
Define subsets baseados em labels `version: v1/v2/v3`

## 🔍 Observabilidade

- **Prometheus**: Métricas no productpage (porta 9080)
- **OpenTelemetry**:
  - productpage: Python instrumentation
  - reviews: Java instrumentation
  - ratings: Node.js instrumentation
