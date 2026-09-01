# Ferramentas Utilizadas

> Ferramentas para um assessment autorizado de segurança de API. Tudo rodou de um ambiente
> Linux (Gentoo); nenhuma alteração permanente foi feita no alvo.

## Matriz de Ferramentas

| Ferramenta | Categoria | Propósito | Uso principal |
|------------|-----------|-----------|---------------|
| **curl** | Cliente HTTP | Construção manual de requisições, fluxos de auth, enumeração de método/verbo, inspeção de headers | Fluxos de login, bateria IDOR, loop de rate-limit, diferenciais de webhook |
| **OpenSSL** | Cripto/TLS | Validação de versão & cipher TLS | `s_client -connect api:443` → TLS 1.3 `TLS_AES_128_GCM_SHA256`, X25519MLKEM768 |
| **ffuf** | Fuzzing | Descoberta de caminhos ocultos e parâmetros | Dicionário de 38 paths (`/v1`, `/swagger`, `/.env`, `/.git`, `/backup.zip`, …) → apenas `/health` |
| **Nuclei** | Escaneamento | Templates de misconfiguração, exposição, CVE | `http/misconfiguration`, `exposures`, `cves` → apenas missing-security-headers (info) |
| **Decodificador JWT** | Cripto | Inspeção de estrutura & claims do token | Verificado `alg: HS256`, `iss`, `exp−iat=900s`, `jti`, claims `role`; confirmado `alg:none`/tokens adulterados rejeitados |
| **Shell Linux** | Ambiente | Script de bursts paralelos, loops, extração de status HTTP | Testes de race paralelos (10×/20×), `grep` em status codes, timing (`time`) |

## Fluxo de trabalho

```
1. Recon       → curl + OpenSSL (fingerprint, TLS, headers)
2. Inventário  → mapa de rotas do código-fonte (RegisterRoutes) — white-box
3. Enumeração  → matriz de métodos curl + ffuf paths ocultos
4. Auth        → fluxos de login curl → decoder JWT → testes de adulteração curl
5. OWASP A01   → bateria IDOR cross-role com curl
6. Negócio     → testes curl de idempotência/CNPJ/cupom (paralelo via shell)
7. Upload      → bateria MIME multipart com curl
8. Automação   → cross-check ffuf + nuclei
9. Limpeza     → curl revert/cleanup, aviso de rotação de credencial
```

## O que foi deliberadamente NÃO usado (e por quê)

| Ferramenta | Motivo |
|------------|--------|
| `sqlmap` | Não disponível no ambiente; a amostragem manual de SQLi cobriu os vetores de maior valor |
| DoS volumétrico pesado | Fora de escopo por contrato (§5); apenas stress leve controlado permitido |
| Persistência / implantes | Fora de escopo; regras do ambiente proíbem mudanças duradouras |

## Recomendação de CI

Versione a coleção `rest/*.http` e a coleção Postman referenciadas no inventário, e conecte
Nuclei/ffuf ao CI para que toda mudança em `routes.go` seja re-validada automaticamente.