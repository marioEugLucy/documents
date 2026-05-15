# 0008 — Cache Busting via Query String no CDN

**Data:** 2026-05-12
**Status:** Aceito

## Contexto

Os parcels são servidos por um CDN (Azure CDN via Terraform) com políticas de cache agressivas para reduzir latência. Após um novo deploy, o CDN pode continuar servindo versões antigas do bundle `gvdasa-index.js` até a expiração do TTL, causando inconsistência entre o shell (atualizado) e o parcel (versão antiga em cache). URLs com hash no nome do arquivo (estratégia padrão do Webpack) não são viáveis porque o endereço do parcel é registrado estaticamente no `Menus.json` e no shell GVDASA.

## Decisão

Adotar **cache busting via query string** no formato `gvdasa-index.js?v=<BuildId>`. O `index.html` gerado pelo Webpack contém referência ao bundle sem query string; após o build, o script `scripts/sspa-inject-html-cache-query.js` reescreve o atributo `src` do script injetando `?v=$(Build.BuildId)` (valor da variável de pipeline `SSPA_APP_BUILD_VERSION`). O CDN deve ser configurado para incluir query strings na cache key.

## Alternativas Consideradas

| Alternativa | Motivo do descarte |
|---|---|
| Hash no nome do arquivo (`gvdasa-index.[contenthash].js`) | O endereço do parcel é fixo no `Menus.json` e no shell; mudança de nome exigiria atualização do registro a cada deploy |
| Invalidação manual de cache no CDN após deploy | Processo manual propenso a falhas humanas; aumenta complexidade operacional do pipeline |
| `Cache-Control: no-store` | Elimina todos os benefícios do CDN, aumentando latência e carga de origem |
| ETag / `Last-Modified` | Ainda faz round-trip ao servidor para validação; não elimina latência do CDN para o primeiro request |

## Consequências

**Positivos:**
- Cada build produz uma URL única (`?v=BuildId`), garantindo que o CDN sirva a versão correta após um deploy.
- O nome do arquivo permanece estático (`gvdasa-index.js`), preservando o endereço registrado no `Menus.json`.
- O pipeline de build injeta automaticamente a query string, sem intervenção manual.

**Negativos:**
- A configuração do CDN deve explicitamente incluir query strings na cache key (`Forward all` / `Include Query Strings = All`) — uma configuração incorreta invalida toda a estratégia.
- `index.html` serve apenas para a sessão de desenvolvimento local (Webpack DevServer); em produção, o parcel é carregado pelo shell via o endereço registrado no `Menus.json`, não pelo `index.html`. A injeção no `index.html` é relevante apenas para o modo standalone.
- Builds sucessivos sem mudança de código ainda geram novas URLs (pois `BuildId` muda), impedindo o reaproveitamento de cache mesmo sem alterações.

## Referências

- `scripts/sspa-inject-html-cache-query.js` — script de injeção pós-build
- `pipelines/build.yml` — variável `SSPA_APP_BUILD_VERSION: $(Build.BuildId)`
- `pipelines/terraform/azurerm_cdn.tf` — configuração do CDN Azure
- ADR 0001 — Single-SPA como framework de micro frontend
