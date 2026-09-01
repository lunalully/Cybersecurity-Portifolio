# Metodologia

> Assessment de segurança de API REST — engajamento **autorizado** (contrato + escopo formal +
> credenciais de teste). Este documento descreve **como** o assessment foi realizado. Nenhuma
> informação identificadora do cliente está incluída.

---

## 1. Modelo do Engajamento

| Atributo | Detalhe |
|----------|---------|
| Tipo | Híbrido: white-box (inventário no código-fonte) + black-box (teste em runtime) |
| Autorização | Documento formal de autorização de pentest, assinado |
| Credenciais | Três roles de teste (customer / store_owner / admin) — rotacionadas ao final |
| Escopo | Autenticação e autorização, sessões, IDOR/elevação, OWASP Top 10, CORS/headers/TLS/rate-limit, upload, coleta de evidência |
| Fora de escopo | Engenharia social, ataques a terceiros, DoS volumétrico contra provedores externos, persistência |
| Regras operacionais | Sem dados reais; indisponibilidade temporária e `reset/migrations/seed` autorizadas; limpeza dos artefatos de teste ao final |

## 2. Fases

### 2.1 Inventário & Reconhecimento (white-box)
- Inventário das rotas no código-fonte (`RegisterRoutes`) — o mapa de rotas autoritativo.
- **130 rotas fixas + 5 por provedor OAuth** (máx. 140).
- Classificação dos endpoints por grupo de acesso: Infra, Auth, Catálogo, JWT Protegido, Admin, Webhook.

### 2.2 Verificação em Runtime (black-box)
- Somente testes não destrutivos (sem DoS, sem exfiltração além do necessário, sem backdoors).
- Cobertura completa de 130/130 endpoints com os três perfis de role.
- Enumeração de métodos HTTP, fuzzing de caminhos ocultos, fuzzing de parâmetros.

### 2.3 Bateria OWASP Top 10 (2021)
Cada categoria mapeada para um plano de teste, executada e classificada:

- **A01 Broken Access Control** — testes cross-role e cross-objeto (`GET /user/{id}`,
  métricas/pedidos de loja, pedidos, endereços, cupons, rotas admin), tentativa de mass assignment.
- **A02 Falhas Criptográficas** — verificação de versão/cipher TLS (OpenSSL), downgrade de
  algoritmo JWT e adulteração de assinatura, vínculo do refresh token.
- **A03 Injeção** — amostragem de SQLi (boolean/union/time), candidatos a XSS armazenado, CRLF/HPP,
  prototype pollution, probes de SSTI.
- **A04 Insecure Design** — idempotência, máquina de estados do pedido, lógica de cupom,
  auto-registro de loja, unicidade de CNPJ, race conditions (bursts paralelos).
- **A05 Security Misconfiguration** — headers, CORS, métodos HTTP, traversal em `/uploads`,
  fingerprint, painéis ocultos.
- **A06 Componentes Vulneráveis e Desatualizados** — verificação de SBOM, exposição de versão
  (sem vazamento por header observado).
- **A07 Falhas de Identificação e Autenticação** — rate limiting, abuso de credenciais,
  refresh/sessão, controles de rotação de senha.
- **A08 Falhas de Integridade de Software e Dados** — validação de assinatura de webhook
  (positiva + negativa), verificação de fail-open.
- **A09 Falhas de Logging e Monitoramento** — correlação via `x-request-id`, qualidade do tratamento de erros.
- **A10 SSRF** — nenhuma superfície de fetch de URL encontrada; verificação de confusão de
  parâmetro em `delivery/estimate`.

### 2.4 Lógica de Negócio & Casos de Abuso
- Criação de loja como customer (teste de escalada).
- Registro duplicado de CNPJ (sequencial + paralelo).
- Idempotência de criação de pedido e auto-pedido.
- Aplicação de cupom em carrinhos vazios.
- Abuso de upload de avatar/logo/banner.

### 2.5 Race Conditions
- 10× criação/remoção paralela de favorito.
- 10× criação paralela de pedido.
- 10×/20× criação paralela de loja com CNPJ idêntico.
- 30× writes paralelos de perfil com payload grande (controlado, não volumétrico).

### 2.6 Teste de Upload
- Validação por MIME sniff (SVG/HTML/PNG falso/polyglot/extensão dupla/null-byte/path traversal).

### 2.7 Automação
- `ffuf` para caminhos ocultos e fuzzing de parâmetros.
- `nuclei` para templates de misconfiguração/exposição/CVE.
- `curl` manual + instrumentação OpenSSL.

## 3. Classificação

Os achados foram classificados com:
- Severidade (Crítico / High / Medium / Low / Info)
- Score CVSS v3.1
- Mapeamentos CWE, OWASP 2021 e MITRE ATT&CK
- Evidência sanitizada reproduzível
- Remediação priorizada

## 4. Limpeza & Conformidade

- Todos os artefatos de teste revertidos (`PATCH /user/profile`, favorito removido, dados de teste
  excluídos quando possível).
- Credenciais de teste sinalizadas para rotação.
- Evidência mantida no mínimo e tratada como confidencial.
- Esta edição pública é **totalmente sanitizada** — sem dados do cliente, sem detalhes de exploit live.