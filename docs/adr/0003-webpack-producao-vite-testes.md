# 0003 — Webpack para Build de Produção e Vite/Vitest para Testes

**Data:** 2026-05-12
**Status:** Aceito

## Contexto

O single-spa recomenda explicitamente Webpack como bundler para produção de parcels, pois o `webpack-config-single-spa-react-ts` fornece opções pré-configuradas alinhadas ao modelo de SystemJS modules exigido pelo shell GVDASA. Em paralelo, o Webpack tem limitações para execução de testes unitários (slow startup, sem suporte nativo a watch rápido), necessitando de uma ferramenta dedicada para o ciclo de desenvolvimento de testes.

## Decisão

Utilizar **Webpack 5** (via `webpack-config-single-spa-react-ts`) para o build de produção do parcel, e **Vite + Vitest** como ambiente de testes unitários. Os dois toolings coexistem: `webpack.config.js` é acionado por `npm run build`, enquanto `vite.config.ts` é acionado por `npm test` e `npm run test:watch`. Ambos compartilham o alias `@/` mapeado para `src/`.

## Alternativas Consideradas

| Alternativa | Motivo do descarte |
|---|---|
| Jest como runner de testes | Configuração complexa com TypeScript e módulos ESM; Vitest oferece API compatível com Jest mas com inicialização até 10× mais rápida |
| Vite também para produção | O output de SystemJS que o shell GVDASA consume é melhor suportado pelo preset Webpack single-spa; Vite em modo library não tem preset oficial single-spa |
| Rollup para produção | Mesma limitação do Vite: ausência de preset single-spa pronto |

## Consequências

**Positivos:**
- Build de produção plenamente compatível com o formato SystemJS do shell GVDASA.
- Testes com Vitest/Happy-DOM são ordens de magnitude mais rápidos em modo watch que Jest com JSDOM.
- `vite.config.ts` reutiliza o mesmo alias `@/` do Webpack, evitando divergências de resolução de módulos entre build e testes.

**Negativos:**
- Dois arquivos de configuração de tooling para manter (`webpack.config.js` e `vite.config.ts`).
- Divergências sutis entre Vite (ESM nativo) e Webpack (CJS/SystemJS) podem causar falsos negativos em testes se módulos externos não forem compatíveis com ambos os resolvers.
- O arquivo `vite.config.ts` precisa declarar `SSPA_APP_BUILD_VERSION` via `define` para evitar `ReferenceError` em testes que importem `src/shared/environment`.

## Referências

- `webpack.config.js` — configuração de build de produção
- `vite.config.ts` — configuração do ambiente de testes
- `test/setup.ts` — setup global dos testes Vitest
- ADR 0001 — Single-SPA como framework de micro frontend
