# Evidência — IDOR / BOLA / RBAC

> Todas as tentativas cross-account e cross-role retornaram `403`. O isolamento do dono funciona.

## IDOR horizontal em `GET /user/{id}`

```bash
# customer tenta ler store_owner
curl -sk https://api.example.com/user/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
  -H "Authorization: Bearer <customer_jwt>"   # → 403 access denied

# store_owner tenta ler customer
curl -sk https://api.example.com/user/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
  -H "Authorization: Bearer <store_jwt>"      # → 403 access denied

# admin pode ler qualquer um
# → 200

# sem token
# → 401 unauthorized
```

## Lookup de email / CPF (sem enumeração em massa)

```http
GET /user/email/customer@example.com
Authorization: Bearer <customer_jwt>
```
```http
HTTP/1.1 200 OK
```

```http
GET /user/email/other@example.com
Authorization: Bearer <customer_jwt>
```
```http
HTTP/1.1 403 Forbidden
{"error":"access denied"}
```

## Checks de propriedade de loja

```bash
# customer numa loja de outro usuário
curl -sk https://api.example.com/store/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/metrics \
  -H "Authorization: Bearer <customer_jwt>"   # → 403 you do not own this store

# store_owner na própria loja
# → 200 (métricas retornadas)
```

## Propriedade de pedido (estrita)

```bash
# dono do pedido (customer)
curl -sk https://api.example.com/order/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
  -H "Authorization: Bearer <customer_jwt>"   # → 200

# store_owner cuja loja produziu o pedido
curl -sk https://api.example.com/order/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
  -H "Authorization: Bearer <store_jwt>"      # → 403 order does not belong to user

# admin via caminho da loja
curl -sk https://api.example.com/store/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/orders/xxxxxxxx-… \
  -H "Authorization: Bearer <admin_jwt>"      # → 403 not store owner
```

## BOLA de endereço

```bash
DELETE /user/address/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx (endereço do customer)
  Authorization: Bearer <store_jwt>           # → 403 address does not belong to user
  Authorization: Bearer <admin_jwt>           # → 403 (sem bypass admin)
```

## Rotas admin

```bash
GET /admin/dashboard   customer → 403 | store → 403 | admin → 200
GET /admin/users       customer → 403 | admin → 200
GET /admin/stores      customer → 403 | admin → 200
```

## Tentativa de mass assignment

```http
PATCH /user/profile HTTP/1.1
Authorization: Bearer <customer_jwt>
Content-Type: application/json

{"role":"admin"}
```
```http
HTTP/1.1 200 OK
```
```json
{ "role": "customer" }   // inalterado — protegido
```