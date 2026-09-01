# Evidência — Tratamento de Assinatura de Webhook

> Resposta diferencial entre dois endpoints de webhook revela existência de loja.

## Webhook de billing — assinatura validada primeiro

```http
POST /webhooks/billing HTTP/1.1
Host: api.example.com
Content-Type: application/json

{}
```
```http
HTTP/1.1 401 Unauthorized
{"error":"invalid webhook signature"}
```

## Webhook de loja — lookup de config acontece ANTES da checagem de assinatura

```http
POST /webhooks/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx HTTP/1.1
Host: api.example.com
Content-Type: application/json

{}   # sem header asaas-access-token
```
```http
HTTP/1.1 404 Not Found
{"error":"store payment config not found"}
```

> O `404` (vs `401`) revela que a loja existe mas não tem config de pagamento. Um agente não
> autenticado pode enumerar existência de loja desta forma (achado F-07).

## Risco de fail-open quando secret ausente (observação no código-fonte)

```go
// do inventário, parafraseado:
// if os.Getenv("PLATFORM_WEBHOOK_SECRET") == "" {
//     // handler processa o evento SEM validar a assinatura
// }
```

> Em qualquer ambiente em que a env var do secret esteja vazia, o webhook de billing aceita
> eventos sem autenticação (achado F-08). Produção deve ser verificada para ter o secret
> definido, e o código deveria falhar fechado.