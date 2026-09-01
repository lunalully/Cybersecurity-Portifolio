# Roadmap de Hardening

> Plano de remediação priorizado com base nos 22 achados (0 Crítico, 2 High, 4 Medium,
> 7 Low, 9 Info). Ordenado por redução de risco por esforço.

---

## Imediato — 0 a 7 dias (HIGH/MEDIUM + higiene)

| # | Ação | Achado relacionado | Dono |
|---|------|--------------------|------|
| 1 | Aplicar gate de role no `POST /store` → `403` para não-`store_owner` (ou fluxo de aprovação) | F-01 | Backend |
| 2 | Adicionar constraint `UNIQUE(cnpj)` + `409 cnpj already exists` + teste de regressão de race | F-02 | Backend / DBA |
| 3 | Rotacionar todas as credenciais de pentest e quaisquer segredos expostos em logs | — | Ops/Security |
| 4 | Verificar se `PLATFORM_WEBHOOK_SECRET` está definido em prod; adicionar `503` fail-closed quando vazio | F-08 | Backend |
| 5 | Sanitizar `name` (rejeitar `<`,`>`,`\x00`, remover `../`) e limitar o tamanho (2–100) | F-03/F-04/F-18/F-19 | Backend |
| 6 | Adicionar limite de tamanho de corpo (1 MB) → `413`; retornar `400` para null bytes | F-04 | Gateway/Backend |
| 7 | Rate-limit em `POST /store` (10/min) igual aos endpoints de auth | F-06 | Backend |

## 30 dias (MEDIUM/LOW + hardening)

| # | Ação | Achado relacionado | Dono |
|---|------|--------------------|------|
| 8 | Validar `asaas-access-token` antes do lookup no DB; `401` genérico em falha de webhook | F-07 | Backend |
| 9 | Whitelist CORS explícita (`admin/store/customer` origins), não `*` | F-04 (orig) / config | Backend/Infra |
| 10 | Fallback de rate-limit em memória quando Redis está fora (fail-closed) | F-11 | Backend |
| 11 | Padronizar a nomenclatura de role `store_owner` em login/JWT/RBAC | F-12 | Backend |
| 12 | Adicionar `permissions-policy`, `COOP`, `CORP` + verificação de headers no CI | F-13 | Backend/Infra |
| 13 | Adicionar job de CI com Nuclei/ffuf cobrindo todas as rotas de `RegisterRoutes` | F-22 | Security/CI |
| 14 | Corrigir `500` em `menu/category` e `admin/user/{bad}` → `400` | F-05 | Backend |

## 60 dias (arquitetura & sessão)

| # | Ação | Achado relacionado | Dono |
|---|------|--------------------|------|
| 15 | Mover tokens para cookies `httpOnly` + `Secure` + `SameSite=Strict` com rotação de refresh (se o front usa `localStorage`) | revisão A02 | Backend/Front |
| 16 | Instrumentar `x-request-id` + `jti` + IP em logs de auth; alertar em bursts de 429 | A09 | Observability |
| 17 | Testar `state`/PKCE do OAuth e open-redirect com provedores mockados | — | Backend |
| 18 | Testar uploads destrutivos (logo/banner/imagens de item) em clone de staging | — | Backend |
| 19 | Proxy da API por edge WAF/CDN; rate limit por IP na borda | F-14 | Infra |

## 90 dias (hardening & lógica de negócio)

| # | Ação | Achado relacionado | Dono |
|---|------|--------------------|------|
| 20 | Pentest de lógica financeira: race de cupom (`WELCOME10` × 100), máquina de estados do pedido, fuzz de variant/addon | F-09/F-16 | Backend |
| 21 | Revisão de SSRF em qualquer caminho de fetch de URL externa (`delivery/estimate`) | A10 | Backend |
| 22 | Publicar SBOM + `security.txt`; rodar `govulncheck`/`trivy` no CI | A06 | Security |
| 23 | Bloquear/verificar auto-pedido na própria loja (se desaconselhado pelo negócio) | F-16 | Backend |
| 24 | Re-teste de regressão após todas as correções; pentest contínuo por release de `routes.go` | — | Security |

---

## Racional de priorização

- **Highs primeiro** porque são lacunas de enforcement de baixo esforço e alto impacto
  (gate de role + constraint unique) que bloqueiam a migração da plataforma para dados reais.
- **Webhook fail-closed** é uma mudança de 1 linha com impacto de supply-chain/integridade (A08).
- **Hardening de validação de entrada** colapsa 4 achados (F-03/F-04/F-18/F-19) em uma única
  camada de validação.
- **Itens de arquitetura** (fallback de rate-limit, edge WAF, estratégia de cookies) são
  adiados para a janela de 60 dias porque exigem coordenação cross-team e validação em staging.