# Registro de Achados

> Os 22 achados do assessment, cada um em um card individual (F-01…F-22). Todos com
> severidade, CVSS v3.1, mapeamentos CWE / OWASP 2021 / MITRE ATT&amp;CK, impacto, evidência
> sanitizada e remediação.
>
> **[→ Visão geral consolidada: `severity-summary.md`](severity-summary.md)**

## Sumário rápido

| Severidade | Qtd | IDs |
|------------|-----|-----|
| **HIGH** | 2 | F-01, F-02 |
| **MEDIUM** | 4 | F-03, F-04, F-05, F-06 |
| **LOW** | 7 | F-07, F-08, F-09, F-10, F-11, F-12, F-13 |
| **INFO** | 9 | F-14, F-15, F-16, F-17, F-18, F-19, F-20, F-21, F-22 |

> F-21 e F-22 são **controles positivos** (PASS): ataques que foram **bloqueados** e
> documentados como prova de que a defesa funciona.

## Leitura recomendada

1. **F-01** e **F-02** — os dois HIGHs (regras de negócio ausentes) — leitura obrigatória.
2. **F-21** e **F-22** — controles positivos (a metade "o que funciona" do assessment).
3. **F-03 / F-04** — hardening de validação de entrada.

## Índice

| ID | Severidade | Título |
|----|-----------|--------|
| [F-01](F-01.md) | HIGH | Escalada vertical de privilégio via auto-registro de loja |
| [F-02](F-02.md) | HIGH | Duplicação massiva de CNPJ — constraint unique ausente & sem lock de race |
| [F-03](F-03.md) | MEDIUM | XSS armazenado (candidato) via campo `name` do perfil |
| [F-04](F-04.md) | MEDIUM | 500 descontrolado em null-byte / payload grande |
| [F-05](F-05.md) | MEDIUM | 500 não tratado em `menu/category` e lookup de user admin |
| [F-06](F-06.md) | MEDIUM | Rate limit ausente na criação de loja |
| [F-07](F-07.md) | LOW | Webhook de loja vaza existência (404 vs 401) |
| [F-08](F-08.md) | LOW | Validação de assinatura de webhook é fail-open quando secret não definido |
| [F-09](F-09.md) | LOW | Criação de pedido idempotente silenciosa (sem Idempotency-Key) |
| [F-10](F-10.md) | LOW | Catálogo ignora JWT inválido (mascara bugs de auth no cliente) |
| [F-11](F-11.md) | LOW | Rate limit depende de Redis — sem fallback |
| [F-12](F-12.md) | LOW | Inconsistência de nomenclatura de role (`store` vs `store_owner`) |
| [F-13](F-13.md) | LOW | Headers de segurança ausentes (Permissions-Policy, COOP, CORP) |
| [F-14](F-14.md) | INFO | API exposta sem edge WAF / CDN |
| [F-15](F-15.md) | INFO | `POST /store` como customer retorna 400 em vez de 403 |
| [F-16](F-16.md) | INFO | Customer pode pedir na própria loja |
| [F-17](F-17.md) | INFO | Grupos internos do catálogo visíveis a customers |
| [F-18](F-18.md) | INFO | Strings tipo path traversal armazenadas literalmente em `name` |
| [F-19](F-19.md) | INFO | Validação fraca de tamanho de nome (1 char) |
| [F-20](F-20.md) | INFO | Delete de avatar idempotente retorna 200 |
| [F-21](F-21.md) | INFO | XFF / Host header sem bypass (**controle positivo**) |
| [F-22](F-22.md) | INFO | Escaneamento automatizado não achou exposição adicional (**controle positivo**) |