# Evidência — Autenticação & Sessão

## Fluxo de login (sanitizado)

```http
POST /login/customer HTTP/1.1
Host: api.example.com
Content-Type: application/json

{"email":"customer@example.com","password":"<REDACTED>"}
```

```http
HTTP/1.1 200 OK
content-security-policy: default-src 'none'
x-request-id: 7114d96d-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

```json
{
  "access_token": "<customer_jwt>",
  "refresh_token": "b354e1…",
  "user": { "role": "customer" }
}
```

## Payload do JWT decodificado (claims sanitizadas)

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
{
  "iss": "saas-core",
  "iat": 1756…,
  "exp": 1756…,
  "uid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "role": "store_owner",
  "jti": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```

> `exp - iat = 900s` (TTL de 15 minutos). `jti` único por login. Nota: a resposta de login
> reporta `role: store` enquanto o JWT codifica `role: store_owner` (ver achado F-12).

## Tentativas de adulteração (todas rejeitadas)

```bash
# alg:none
curl -sk https://api.example.com/user/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
  -H "Authorization: Bearer eyJhbGciOiJub25lIn0…"  # → 401 invalid token

# role adulterada + uid reatribuído
# → 401 invalid token

# expirado (exp:1000)
# → 401 invalid token
```

## Rate limit (auth)

```bash
for i in $(seq 1 12); do
  curl -sk -o /dev/null -w '%{http_code}\n' -X POST https://api.example.com/login/customer \
    -H "Content-Type: application/json" \
    -d '{"email":"customer@example.com","password":"wrong"}'
done
# 401 401 401 401 401 401 429 429 429 429 429 429
```

## Fluxo de refresh

```http
POST /auth/refresh HTTP/1.1
Content-Type: application/json

{}   # sem device_id
```

```http
HTTP/1.1 400 Bad Request
{"error":"refresh_token is required"}
```

```http
POST /auth/refresh HTTP/1.1
Content-Type: application/json

{"refresh_token":"b354e1…"}   # falta device_id
```

```http
HTTP/1.1 400 Bad Request
{"error":"device_id is required"}
```

> O refresh é vinculado ao `device_id` — uma mitigação contra roubo isolado de refresh token.
> Testar rotação/revogação completa exige um `device_id` real (não disponível no ambiente de
> pentest); sinalizado como follow-up em staging.