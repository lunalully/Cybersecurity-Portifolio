# Evidências — Sanitizadas

> Cada trecho abaixo foi limpo de dados identificadores do cliente: hostnames → `example.com`,
> emails → `@example.com`, UUIDs → `xxxxxxxx-xxxx-…`, CNPJs → mascarados, tokens → `<…_jwt>`,
> segredos → `<REDACTED>`. Esta é a **edição pública de portfólio** — a evidência original é
> confidencial e fica fora deste repositório.

## Índice

| Arquivo | Tópico |
|---------|--------|
| [auth-and-session.md](auth-and-session.md) | Fluxo de login, estrutura JWT, refresh, rate limit 429 |
| [idor-and-rbac.md](idor-and-rbac.md) | Tentativas de IDOR e respostas 403 |
| [webhooks.md](webhooks.md) | Tratamento de assinatura de webhook (diferencial 401 / 404) |
| [input-validation.md](input-validation.md) | XSS / null-byte / payloads grandes |
| [tls-and-headers.md](tls-and-headers.md) | Handshake TLS 1.3 e headers de resposta |
| [upload-tests.md](upload-tests.md) | Resultados de MIME sniffing em upload |
| [race-and-business-logic.md](race-and-business-logic.md) | Bursts paralelos, duplicação de CNPJ, idempotência de pedido |

---

## O que foi preservado vs removido

**Preservado** (o valor técnico):
- Formas de requisição/resposta HTTP (status codes, headers, estrutura JSON)
- Mensagens de erro e diferenciais comportamentais (401 vs 403 vs 404)
- Categorias de payload e resultados observados
- Padrões de timing, ordem e repetição

**Removido / mascarado** (confidencialidade do cliente):
- Nomes de empresa e produto
- Hostnames reais, endereços IP, números de CNPJ, IDs fiscais
- Endereços de email, telefones, nomes completos
- Valores de payload JWT, IDs de requisição além de um prefixo mascarado, segredos
- Qualquer coisa que permitisse reproduzir ataques contra o ambiente live