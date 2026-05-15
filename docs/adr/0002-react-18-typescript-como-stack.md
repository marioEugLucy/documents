# 0002 — React 18 + TypeScript como Stack de Desenvolvimento

**Data:** 2026-05-12
**Status:** Aceito

## Contexto

Os parcels do Painel GVDASA precisam de uma stack frontend produtiva, com ecossistema maduro, suporte a Concurrent Features do React, e tipagem estática para reduzir erros em tempo de desenvolvimento. A padronização da stack entre produtos facilita a mobilidade de desenvolvedores entre times.

## Decisão

Adotar **React 18** como biblioteca de UI e **TypeScript** (com `strict` habilitado em `tsconfig.json`) como linguagem de desenvolvimento. O Babel é utilizado para transpilar TypeScript durante o build (via `@babel/preset-typescript`) e não como type-checker — a verificação de tipos é feita via `tsc --noEmit` no CI.

## Alternativas Consideradas

| Alternativa | Motivo do descarte |
|---|---|
| Vue 3 | Ecossistema interno GVdasa consolidado em React; custo de migração dos componentes existentes e treinamento dos times |
| Angular | Curva de aprendizado mais alta e peso do framework incompatível com o modelo de parcel leve |
| React sem TypeScript (JavaScript puro) | Perde-se autocomplete, detecção de erros em tempo de desenvolvimento e autodocumentação via tipos |

## Consequências

**Positivos:**
- Tipagem estática previne erros comuns de contrato de props e retorno de funções.
- React 18 habilita Concurrent Features (Suspense, Transitions) para UX mais responsiva.
- Ecossistema amplamente suportado pelos componentes `@gvdasa/react-components` e `@gvdasa/gv-core`.
- Aliases `@/*` no `tsconfig.json` e webpack/vite evitam caminhos relativos longos nos imports.

**Negativos:**
- Babel transpila TypeScript sem checar tipos em tempo de build — erros de tipo só são capturados pelo `tsc` ou pela IDE, não pelo `npm run build`.
- A configuração `strict` pode gerar fricção inicial em migrações de código legado JavaScript.

## Referências

- `tsconfig.json` — configuração TypeScript do projeto
- `babel.config.json` — configuração Babel (presets React/TS)
- `package.json` — dependências React 18, TypeScript
