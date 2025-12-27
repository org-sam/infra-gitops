# Bookinfo Application

Aplicação de exemplo do Istio composta por 4 microserviços:

- **productpage**: Frontend que chama os demais serviços
- **details**: Informações sobre o livro
- **reviews**: Avaliações do livro (3 versões)
- **ratings**: Classificação por estrelas

## Acesso

```bash
# Port-forward para acessar localmente
kubectl port-forward -n bookinfo svc/productpage 9080:9080
```

Acesse: http://localhost:9080/productpage
