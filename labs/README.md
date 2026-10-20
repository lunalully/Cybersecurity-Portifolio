# CTFs & Labs — manutenção dos dados

A seção **CTFs & Labs** do portfólio (`#labs` no `index.html`) é renderizada no navegador a partir de
`labs/labs.json`. O site é estático (GitHub Pages, sem backend), então o cadastro é feito **editando este
arquivo JSON versionado**. Não há formulário de cadastro nem banco de dados no navegador.

Todos os registros são **laboratórios controlados de estudo**, não vulnerabilidades em sistemas reais
nem pentest profissional.

## Arquivos

```
labs/
├── labs.json        # dados (array de objetos, um schema único)
├── README.md        # este documento
└── relatorios/      # relatórios em PDF referenciados por link_relatorio
```

## Schema

`labs.json` é um array. Cada item usa os campos abaixo; todos são opcionais, exceto `id` e `nome`.
Campo sem informação deve ser **omitido** (ou vazio), nunca inventado.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `id` | string | Slug único (letras minúsculas, números e hífen). Usado no deep-link `#lab-<id>`. |
| `nome` | string | Nome do lab. |
| `plataforma` | string | Ex.: PortSwigger Web Security Academy. |
| `data` | string | Data ISO `AAAA-MM-DD`. |
| `categoria` | string | Ex.: Access control. |
| `dificuldade` | string | Somente se houver fonte. |
| `status` | string | `resolvido`, `em andamento` ou `revisitando`. |
| `resumo` | string | Texto curto exibido (uma linha) na lista da categoria. |
| `vulnerabilidade` | string | Vulnerabilidade ou conceito. |
| `tecnicas` | string[] | Técnicas usadas. |
| `ferramentas` | string[] | Ferramentas usadas. |
| `aprendizados` | string | Lições do lab. |
| `link_lab` | string | URL pública do lab (somente `http:`/`https:`). |
| `link_relatorio` | string | URL ou caminho relativo do relatório (ex.: `labs/relatorios/arquivo.pdf`). |
| `evidencias` | objeto[] | `{ "descricao": "...", "arquivo": "caminho/ou/url" }`. |
| `objetivo`, `cenario`, `metodologia`, `observacoes`, `impacto`, `causa_raiz` | string | Seções do detalhe. |
| `recomendacoes` | string[] ou string | Recomendações defensivas. |
| `referencias` | objeto[] | `{ "titulo": "...", "url": "https://..." }`. |

### Exemplo

```json
{
  "id": "exemplo-de-lab",
  "nome": "Exemplo de lab",
  "plataforma": "PortSwigger Web Security Academy",
  "data": "2026-10-21",
  "categoria": "Access control",
  "status": "resolvido",
  "resumo": "Descrição curta do que o lab ensina.",
  "tecnicas": ["IDOR"],
  "ferramentas": ["Burp Suite Repeater"],
  "metodologia": "Passos executados, sem credenciais.",
  "aprendizados": "O que ficou de lição.",
  "referencias": [{ "titulo": "Página pública do lab", "url": "https://portswigger.net/web-security" }]
}
```

## Como adicionar, editar e remover

- **Adicionar:** acrescente um objeto ao array (atenção às vírgulas) com `id` novo e único.
- **Editar:** altere os campos do objeto; mantenha o `id` para não quebrar links já compartilhados.
- **Remover:** apague o objeto correspondente.
- Valide sempre: `python3 -m json.tool labs/labs.json > /dev/null`.

## Regras de sanitização

Nunca inclua no JSON, nos relatórios ou nas evidências: senhas, tokens, cookies de sessão, chaves de API,
dados pessoais, nem endereços de instância de lab (`*.web-security-academy.net`). Descreva o achado sem
reproduzir o segredo (ex.: "uma credencial no transcript"). Revise PDFs e imagens antes de publicá-los.

## Teste local

O `fetch` não funciona em `file://`. Na raiz do repositório:

```
python3 -m http.server 8000
```

e abra `http://127.0.0.1:8000/#labs`.

## Publicação

Faça commit e push nos remotes `pages` e `origin`. Os caminhos são relativos, então funcionam tanto na
raiz quanto no subcaminho `/Cybersecurity-Portifolio/` do GitHub Pages.
