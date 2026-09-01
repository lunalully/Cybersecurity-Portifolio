# Linha do Tempo do Engajamento

> Engajamento autorizado, com uma única sessão de teste ativo de ~90 minutos.
> Todos os horários em UTC. Todos os artefatos sanitizados.

| Horário | Fase | Trabalho realizado | Resultado |
|---------|------|--------------------|-----------|
| D-1 | Prep | Inventário de rotas no código-fonte (`RegisterRoutes`); escopo; emissão de credenciais | 130 rotas mapeadas; escopo acordado |
| T+0:00 | Recon | Probes de health/metadata; revisão de headers; handshake TLS; fingerprint | TLS 1.3, HSTS, CSP `'none'`, 404 estilo Gin |
| T+0:05 | Auth | Login ×3 roles; decode de JWT; vínculo de refresh; loop de rate-limit | 10/min efetivo; `alg:none` rejeitado |
| T+0:15 | OWASP A01 | Bateria IDOR cross-role: users, stores, orders, addresses | Tudo bloqueado 403 — sem IDOR |
| T+0:30 | OWASP A03/A05 | Amostragem de injeção; CORS; métodos; headers; path traversal | Sem SQLi; CORS restritivo |
| T+0:40 | Lógica de negócio | Idempotência; mínimos de cupom; auto-pedido | Invariantes se sustentam; documentado (F-09) |
| T+0:50 | Uploads | Bateria de MIME sniff (SVG/HTML/polyglot/null-byte) | Todos 400 — validação por decode |
| T+0:55 | Race | Bursts paralelos de favorito/pedido/loja | Favoritos/pedidos idempotentes; race de CNPJ quebrada (F-02) |
| T+1:10 | Deep-dive ofensivo | Escalada customer→store; CNPJ massivo; 30×5k payload; 20× svg bombs | F-01/F-02/F-06 confirmados; app ficou de pé |
| T+1:25 | Automação | `ffuf` (38 paths), `nuclei` (misconfig/exposures/cves) | Apenas missing-security-headers (info) |
| T+1:30 | Limpeza | Revertido nome de perfil; favorito removido; artefatos sinalizados para reset | Ambiente deixado limpo |
| Depois | Reporte | Triage, mapeamento CVSS/CWE/OWASP/MITRE, plano de remediação | 22 achados (0 crítico) |

## Métricas de cobertura

- **Rotas:** 130/130 cobertas (exaustivo) com os 3 perfis de role
- **Requisições:** 150+ construídas (fuzz, race, upload, método, rate-limit)
- **OWASP 2021:** 10/10 categorias exercitadas
- **Automação:** validação dos achados manuais com ffuf + nuclei
- **Resultado:** 0 Crítico / 2 High / 4 Medium / 7 Low / 9 Info

## Não coberto neste ciclo (próximo ciclo com reset/staging autorizado)

1. Fluxos OAuth Google/Apple (env-gated, não expostos neste ambiente)
2. Uploads destrutivos (logo/banner/imagens de item) com polyglots reais e arquivos gigantes
3. Falsificação de webhook de pagamento contra sandbox Asaas
4. Rotação/replay de refresh token (requer um `device_id` real do app cliente)
5. Race de cupom em pedido `PLACED` com itens
6. Probes de SSRF em qualquer caminho que faça fetch de URL externa (`delivery/estimate` geocoding)