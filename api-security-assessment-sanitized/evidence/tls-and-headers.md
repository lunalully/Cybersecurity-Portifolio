# Evidência — TLS & Headers

## Handshake TLS 1.3 (OpenSSL)

```text
openssl s_client -connect api.example.com:443
Protocol : TLSv1.3
Cipher  : TLS_AES_128_GCM_SHA256
KEM     : X25519MLKEM768 / RSASSA-PSS
```

> Sem TLS 1.0/1.1 ofertado. Certificado válido (Let's Encrypt, RSA 4096). HSTS presente em todas
> as respostas. Nota: a falha de CA local (error 14) no container é limitação do store de CA do
> cliente, não defeito do servidor.

## Headers de resposta (todas as rotas)

```http
HTTP/1.1 200 OK
content-security-policy: default-src 'none'
referrer-policy: strict-origin-when-cross-origin
strict-transport-security: max-age=31536000; includeSubDomains
x-content-type-options: nosniff
x-frame-options: DENY
x-request-id: 7114d96d-xxxx-xxxx-xxxx-xxxxxxxxxxxx
alt-svc: h3=":443"; ma=2592000
```

> Ausentes: `permissions-policy`, `cross-origin-opener-policy`, `cross-origin-resource-policy`
> (achado F-13).

## CORS — sem reflexão (PASS)

```http
OPTIONS /cuisines HTTP/1.1
Origin: https://evil.example.com
Access-Control-Request-Method: GET
```
```http
HTTP/1.1 403 Forbidden
```
> Nenhum `Access-Control-Allow-Origin` refletido para origin não permitida; `GET /cuisines` com
> `Origin: evil.example.com` não reflete a origin.

## Tratamento de métodos (PASS)

```bash
TRACE /health          → 404 (sem echo)
PUT  /health           → 404 (default do Gin; sem vazamento de header Allow)
OPTIONS /health (evil) → 403 (sem ACAO)
```

## Path traversal em uploads (PASS)

```bash
GET /uploads/../../../../etc/passwd → 404
GET /uploads/.env                   → 404
GET /.git/HEAD                      → 404
GET /metrics (sem token)            → 401
```

## Higiene de fingerprint (PASS)

```text
Sem header Server:. Sem X-Powered-By. 404 é "page not found" estilo Gin.
fronts (admin/store/customer) atrás de challenge da CDN; API exposta diretamente (F-14).
```