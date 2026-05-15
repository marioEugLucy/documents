# 0001 — Single-SPA como Framework de Micro Frontend

**Data:** 2026-05-12
**Status:** Aceito

## Contexto

O Painel GVDASA hospeda múltiplos produtos independentes sob um único portal. Cada produto é desenvolvido por times distintos, com ciclos de release e stacks próprias. É necessário um modelo de composição de frontend que permita implantação independente de cada produto sem interferência entre eles e sem exigir que o portal inteiro seja redeploy.

## Decisão

Adotar **single-spa 6** como framework de orquestração de micro frontends. Cada produto é entregue como um **parcel** — unidade de composição do single-spa — com ciclo de vida próprio (`bootstrap`, `mount`, `unmount`) exposto via `gvdasa-index.ts`. O shell GVDASA (root-config) é responsável por carregar e montar os parcels conforme o roteamento ativo.

## Alternativas Consideradas

| Alternativa | Motivo do descarte |
|---|---|
| Module Federation (Webpack 5) | Requer maior acoplamento de configuração entre o shell e cada parcel; harder to support runtime-independent deploys without coordinated versioning |
| iframe por produto | Isolamento perfeito, mas experiência de usuário fragmentada (sem tema unificado, cookies separados, navegação descontinuada) |
| Monolito com lazy-loading de rotas | Inviável para times independentes; acoplamento de release entre produtos |

## Consequências

**Positivos:**
- Deploy independente de cada parcel sem impacto nos demais.
- Times isolados com autonomia de stack dentro de convenções GVdasa.
- Reutilização de dependências compartilhadas (React, Axios, MUI) via `externals` do Webpack, reduzindo tamanho dos bundles.

**Negativos:**
- Necessidade de alinhar versões de dependências compartilhadas entre shell e parcels (potencial breaking change se o shell atualizar React antes dos parcels).
- Depuração mais complexa: erros em boundary do single-spa são capturados pelo `errorBoundary` e podem suprimir stack traces detalhados.
- A limpeza de side effects no `unmount` exige disciplina — memory leaks acontecem quando subscriptions não são removidas.

## Referências

- [single-spa docs](https://single-spa.js.org/)
- `src/gvdasa-index.ts` — entry point do parcel
- `public-dev/gvdasa-root-config.ts` — mock do root-config para desenvolvimento local
- ADR 0003 — Webpack como bundler de produção
