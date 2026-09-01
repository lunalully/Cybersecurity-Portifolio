# Notas do Portfólio

> Revisão técnica do relatório original do assessment, melhorias aplicadas nesta edição do
> portfólio e lições aprendidas. Escrito da perspectiva de um lead sênior de AppSec/pentest.

---

## 1. Lições aprendidas (técnicas & profissionais)

### 1.1 Controles positivos valem tanto quanto achados
As seções mais fortes do relatório são os **resultados negativos**: cada IDOR/BOLA, adulteração
de JWT, polyglot de upload e reflexão de CORS que retornou `403`/`401`/`400` constitui evidência
de um controle funcionando. Recrutadores e leads de segurança avaliam como você **prova** que um
controle funciona, não apenas como você encontra bugs. Esta edição promove esses testes negativos
a "controles validados" de primeira classe (veja `findings/severity-summary.md`).

### 1.2 Teste o modelo de negócio, não só o framework
Os achados de maior severidade (F-01, F-02) não eram vulnerabilidades de framework — eram
**regras de negócio ausentes** (gate de role na criação de loja, unicidade de CNPJ). Saída
clássica de scanner nunca os encontraria. Essa é a diferença entre operar scanner e conduzir um
assessment.

### 1.3 Race conditions precisam de teste paralelo, não sequencial
A duplicação de CNPJ só era comprovável com bursts paralelos de 10×/20×. Um re-teste sequencial
mostraria "um por chamada" e perderia completamente o lock ausente.

### 1.4 Severidade é sobre contexto de negócio
O mesmo rate limit ausente é `LOW` numa leitura de catálogo e `MEDIUM` num caminho de escrita que
habilita fraude em massa. Severidade contextual — não tabela de consulta — é o que clientes
empresariais pagam.

### 1.5 Comunicação importa tanto quanto exploração
O relatório separou claramente "escopo autorizado" de "fora de escopo", incluiu um roadmap de
remediação e sinalizou rotação de credenciais. Entregar com um plano de 7/30/60/90 dias é o que
torna um pentest **acionável**, não um objeto de museu.

## 2. Melhorias aplicadas nesta edição do portfólio

| # | Melhoria | Por que eleva o nível profissional |
|---|----------|------------------------------------|
| 1 | **CVSS v3.1** em cada achado | Padroniza a severidade; recrutadores/leads esperam isso |
| 2 | **CWE + OWASP 2021 + MITRE ATT&CK** em cada achado | Mostra vocabulário maduro de threat modeling |
| 3 | **Divisão executivo vs técnico** | Exec summary + evidência sanitizada deixa qualquer audiência ler o mesmo doc |
| 4 | **Seção de controles verificados** (testes positivos) | Prova habilidade de validação defensiva, não só ofensiva |
| 5 | **Risk matrix + distribuição de severidade** | Triage visual, leitura instantânea |
| 6 | **Numeração limpa e consistente de achados (F-01…F-22)** | Merge/re-numeração dos relatórios delta → um entregável coerente |
| 7 | **Roadmap de hardening com donos + janelas** | Transforma achados em programa, não em lista |
| 8 | **Política de sanitização documentada** | Demonstra disciplina de confidencialidade — sinal para empregador |
| 9 | **Mapeamento completo ferramenta→propósito→fluxo** | Mostra raciocínio de ferramentas, não citação de nomes |
| 10 | **Seção honesta de "não testado"** | Reconhecer lacunas de cobertura é mais crível do que reivindicar 100% de tudo |

## 3. O que o relatório original não tinha (como revisor sênior)

1. **Sem score de severidade** — substituído por CVSS v3.1 + contexto de negócio.
2. **Achados delta anexados** — consolidados num registro numerado único.
3. **Sem mapeamento CWE/ATT&CK** — adicionado a cada card.
4. **Postura de sanitização fraca** — o original nomeava cliente/produto por todo lado.
5. **Sem exec summary** — adicionado (veja report.pdf).
6. **Evidência verbosa** — destilada em trechos canônicos de requisição/resposta.
7. **Sem plano de re-teste** — adicionado um roadmap de 90 dias com donos.
8. **Sem enquadramento de "controles verificados"** — evidência negativa agora enquadrada como PASS.

## 4. Pontos de conversa para entrevista (ver `../linkedin/` para o kit completo)

- "Os 2 Highs eram regras de negócio ausentes, não bugs de framework — validei com testes de
  race condition e de gate de role."
- "A API estava endurecida contra ataques web clássicos, então mudei para teste de casos de
  abuso: quem pode virar lojista? O identificador fiscal pode ser duplicado?"
- "Provei os controles que FUNCIONAVAM (JWT, RBAC, upload com MIME sniff, CORS) — essa é a
  metade de engenharia do assessment."

---

*Este documento faz parte de um portfólio público e sanitizado. Não contém dados do cliente e nem
instruções que possam reproduzir ataques no ambiente live.*