# Evidência — Teste de Upload

> O caminho de upload valida o conteúdo por **MIME sniffing e decode de imagem** — não apenas
> pela extensão ou pelo header do arquivo. Isso bloqueou todas as tentativas.

## `POST /user/avatar` resultados multipart

| Arquivo | Resposta |
|---------|----------|
| sem arquivo | `400 image field is required` |
| `payload.svg` (`<svg onload>`) | `400 unsupported image format: text/plain; charset=utf-8` |
| `payload.html` | `400 unsupported image format: text/html` |
| `fake.png` (header PNG de 8 bytes + lixo) | `400 failed to decode image` |
| `evil.svg.php` (extensão dupla) | `400 text/plain` |
| `../../evil.svg` (path traversal) | `400 text/plain` |
| `evil.svg%00.jpg` (null byte) | `400 text/plain` |

> O caso `fake.png` é o sinal mais forte: o servidor **decodifica** a imagem, não apenas os
> magic bytes. Nenhuma cadeia upload→execução foi encontrada.

## Stress (20 bombas SVG paralelas)

```bash
# 20 × <svg><g><rect>…1000 rects → todas as 20 retornaram 400 unsupported image format
```

> O servidor permaneceu saudável o tempo todo (`/health` 200).

## Caminhos relacionados (não totalmente exercitados neste ciclo)

```text
POST /store/{id}/logo|banner        → 400 (validação; upload destrutivo não testado)
POST /store/{id}/item/{itemId}/image/{index} → 400
DELETE /user/avatar                 → 200 idempotente
```

> Follow-up: uploads polyglot, arquivos gigantes (10MB+), varredura antivírus e servir de
> `.env` via `/uploads/*` em clone de staging (ambiente de reset autorizado).