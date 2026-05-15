# 0007 — SWR para Cache e Sincronização de Dados do Servidor

**Data:** 2026-05-12
**Status:** Aceito

## Contexto

Os parcels realizam frequentes buscas de dados do servidor que precisam de: cache local para evitar re-fetches desnecessários ao navegar entre rotas, atualização automática quando os dados mudam, suporte a debounce em buscas com filtros digitados pelo usuário e atualização otimista do cache após mutações. A gestão manual desses comportamentos com `useState` + `useEffect` resulta em código verboso e propenso a race conditions.

## Decisão

Adotar **SWR** (`swr`) como biblioteca de cache e sincronização de dados do servidor, abstraído pelo hook customizado `useGetAndCache` em `src/hooks/useGetAndCache.ts`. O hook encapsula: cache por chave, debounce configurável (para filtros de busca), mapeamento de estados (`loading`, `error`, `data`) e função `mutate` para atualização otimista. O parcel também inclui **TanStack Query** (`@tanstack/react-query`) como dependência disponível para casos mais complexos que exijam paginação ou invalidação por tag.

## Alternativas Consideradas

| Alternativa | Motivo do descarte |
|---|---|
| `useState` + `useEffect` puro | Verboso; race conditions entre re-renders e requisições paralelas; sem cache automático |
| TanStack Query (React Query) como única solução | Configuração mais verbosa para casos simples; o `useGetAndCache` baseado em SWR é suficiente para 90% dos casos de listagem e detalhe |
| Apollo Client | Específico para GraphQL; as APIs do gateway GVDASA são REST |

## Consequências

**Positivos:**
- Re-fetches são evitados automaticamente para chaves já em cache, reduzindo carga no servidor.
- `revalidateOnFocus: false` evita re-fetches indesejados ao alternar entre abas/janelas.
- O debounce integrado ao `useGetAndCache` evita requisições excessivas durante digitação de filtros.
- Atualização otimista via `mutate` permite UX responsiva em operações de CRUD sem aguardar round-trip.

**Negativos:**
- Duas bibliotecas de cache no projeto (SWR + TanStack Query) — times devem ter um critério claro de quando usar cada uma (SWR para buscas simples via `useGetAndCache`; TanStack Query para queries complexas com paginação/mutations).
- Caches separados: SWR e TanStack Query não compartilham o mesmo cache store, podendo gerar inconsistências se os dois forem usados para a mesma chave de recurso.
- Necessidade de invalidar manualmente o cache SWR após mutações que não usem o `mutate` local.

## Referências

- `src/hooks/useGetAndCache.ts` — abstração do SWR com debounce
- `src/hooks/useDebounce.ts` — hook de debounce utilizado internamente
- `package.json` — dependências `swr`, `@tanstack/react-query`
- ADR 0005 — GVHttpClient singleton para requisições HTTP
