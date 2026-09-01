# Cybersecurity Portfolio

Portfólio de segurança ofensiva — API security, pentest e AppSec.

## Acesso

- **Página do portfólio (GitHub Pages):** https://lunalully.github.io/Cybersecurity-Portifolio/
- **Repositório GitHub:** https://github.com/lunalully/Cybersecurity-Portifolio

## Estrutura

```
.
├── index.html                  # Página pessoal do portfólio (GitHub Pages)
├── .nojekyll                   # Desativa processamento Jekyll no GitHub Pages
├── README.md
└── api-security-assessment-sanitized/   # Projeto: assessment de API (PT-BR, sanitizado)
    ├── report.pdf              # Relatório premium de 25 páginas
    ├── findings/               # 22 cards de achados (CVSS/CWE/OWASP/ATT&CK)
    ├── evidence/               # Evidências sanitizadas
    ├── assets/                 # Diagramas SVG
    └── linkedin/               # Carrossel, currículo ATS, kit de entrevistas
```

## Publicação no GitHub Pages

O Pages está configurado para servir a **raiz** da branch `main`:

1. Push do repositório para `main`.
2. No GitHub: **Settings → Pages → Source: Deploy from a branch → main / (root)**.
3. O site fica disponível em `https://lunalully.github.io/Cybersecurity-Portifolio/`.

O `index.html` usa apenas caminhos relativos (`api-security-assessment-sanitized/report.pdf`),
então funciona tanto localmente quanto no Pages sem ajustes.