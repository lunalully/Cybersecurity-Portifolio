# Resumo dos Achados

> **Assessment:** API REST (Go/Gin) — plataforma SaaS de delivery de comida
> **Total de achados:** 22 | **Crítico:** 0 | **High:** 2 | **Medium:** 4 | **Low:** 7 | **Info:** 9
> **Todos os dados sanitizados — sem informação identificadora do cliente.**

---

## Distribuição por Severidade

```
HIGH ████████████████████████████████  2
MED  ████████████████████████         4
LOW  ████████████████████             7
INFO ██████████████████████████████   9
```

## Matriz Completa

| ID | Severidade | Título | OWASP 2021 | MITRE ATT&CK | Status |
|----|-----------|--------|-----------|--------------|--------|
| F-01 | **HIGH** | Escalada vertical de privilégio via auto-registro de loja | A01 / A04 | T1078 | Aberto |
| F-02 | **HIGH** | Duplicação massiva de CNPJ — constraint unique ausente & sem lock de race | A04 | T1078 / T1498 | Aberto |
| F-03 | **MEDIUM** | XSS armazenado (candidato) via campo `name` do perfil | A03 | T1059.007 | Aberto |
| F-04 | **MEDIUM** | 500 descontrolado em null-byte / payload grande | A03 | T1499 | Aberto |
| F-05 | **MEDIUM** | 500 não tratado em `menu/category` e lookup de user admin | A05 | T1499 | Aberto |
| F-06 | **MEDIUM** | Rate limit ausente na criação de loja | A07 | T1110 | Aberto |
| F-07 | LOW | Webhook de loja vaza existência de loja (404 vs 401) | A01 | T1046 | Aberto |
| F-08 | LOW | Validação de assinatura de webhook é fail-open quando secret ausente | A08 | T1195 | Aberto |
| F-09 | LOW | Criação de pedido idempotente silenciosa (sem Idempotency-Key) | A04 | T1498 | Aberto |
| F-10 | LOW | Catálogo ignora JWT inválido (mascara bugs de auth no cliente) | A07 | T1078 | Aberto |
| F-11 | LOW | Rate limit depende de Redis — sem fallback | A07 | T1498 | Aberto |
| F-12 | LOW | Inconsistência de nomenclatura de role (`store` vs `store_owner`) | A07 | — | Aberto |
| F-13 | LOW | Headers de segurança ausentes (Permissions-Policy, COOP, CORP) | A05 | T1190 | Aberto |
| F-14 | INFO | API exposta sem edge WAF / proteção CDN | A05 | T1190 | Aberto |
| F-15 | INFO | `POST /store` como customer retorna 400 em vez de 403 | A01 | — | Aberto |
| F-16 | INFO | Customer pode auto-pedir na própria loja | A04 | T1078 | Aberto |
| F-17 | INFO | Grupos internos do catálogo visíveis a customers | A01 | T1046 | Aberto |
| F-18 | INFO | Strings tipo path traversal armazenadas literalmente em `name` | A03 | — | Aberto |
| F-19 | INFO | Validação fraca de tamanho de nome (1 char) | A04 | — | Aberto |
| F-20 | INFO | Delete de avatar idempotente retorna 200 | A05 | — | Info |
| F-21 | INFO | XFF / Host header não burlam rate-limit ou vhost (controle positivo) | A07 | — | Validado |
| F-22 | INFO | ffuf / nuclei não acharam exposição adicional (controle positivo) | A05 | — | Validado |

## Controles Verificados (evidência negativa — PASS)

| Controle | Resultado |
|----------|-----------|
| JWT rejeita `alg: none`, uid/role adulterados, expirados, RS256 falso | PASS (401) |
| IDOR/BOLA em `user/:id`, `store/:id/*`, `order/:id`, endereços | PASS (403) |
| Escalada de privilégio `customer → store → admin` | BLOQUEADA (403) |
| Mass assignment (`role: admin`) | PROTEGIDO |
| TLS 1.3 + HSTS + CSP `default-src 'none'` + `nosniff` + `DENY` | PASS |
| Upload: SVG/HTML/polyglot/path-traversal/null-byte | BLOQUEADO (400) |
| CORS: sem reflexão de origin, preflight de origin `evil` rejeitado | PASS |
| Rate limit de auth: 429 após logins falhos (Redis ativo) | PASS |
| Path traversal em `/uploads/*` e `/.env`, `/.git` | BLOQUEADO (404) |
| Webhook de billing sem assinatura | REJEITADO (401) |

---

*Esta é uma edição pública de portfólio sanitizada. Os cards de achados estão disponíveis como arquivos individuais (`F-01.md` … `F-22.md`).*