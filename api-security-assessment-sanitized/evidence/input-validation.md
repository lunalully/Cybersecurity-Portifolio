# Evidência — Validação de Entrada

## XSS armazenado (candidato) — `PATCH /user/profile`

```http
PATCH /user/profile HTTP/1.1
Authorization: Bearer <customer_jwt>
Content-Type: application/json

{"name":"<script>alert(document.cookie)</script>"}
```
```http
HTTP/1.1 200 OK
```
```http
GET /user/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
Authorization: Bearer <customer_jwt>
```
```json
{ "name": "\u003cscript\u003ealert(document.cookie)\u003c/script\u003e" }
```

> A API JSON escapa o payload, então não executa nas respostas da API. O risco existe apenas se
> algum front-end renderizar este campo como HTML (ver F-03). Payloads de SSTI (`{{7*7}}`, `${7*7}`,
> `<%= 7*7 %>`) retornaram como literais — **sem injeção de template** (PASS).

## Null byte / payload grande → 500

```bash
curl -sk -X PATCH https://api.example.com/user/profile \
  -H "Authorization: Bearer <customer_jwt>" \
  -H "Content-Type: application/json" -d '{"name":"teste\u0000"}'
# → 500 internal error

curl -sk -X PATCH https://api.example.com/user/profile \
  -H "Authorization: Bearer <customer_jwt>" \
  -H "Content-Type: application/json" -d '{"name":"AAAA…(5000)"}'
# → 500 internal error  (sem 413 Payload Too Large)
```

## Tipagem estrita (PASS)

```bash
PATCH /user/profile {"name":12345} → 400 invalid request body
PATCH /user/profile {"name":[]}    → 400 invalid request body
PATCH /user/profile {"name":{}}    → 400 invalid request body
```

## Amostragem de SQLi / NoSQL (PASS)

```bash
GET /user/email/' OR '1'='1            → 404 user not found (sem dump)
GET /stores?search='<script>           → 200 JSON válido, sem reflexão
POST /login {"email":{"$gt":""}}       → 400 invalid request body
Payloads WAITFOR/UNION                 → sem delay, sem vazamento de erro
```

## Prototype pollution (PASS — encoding/json do Go)

```bash
PATCH /user/profile {"__proto__":{"polluted":"yes"}} → 200, sem pollution
```