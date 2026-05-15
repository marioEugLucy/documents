# 0005 — GVHttpClient Singleton para Requisições HTTP

**Data:** 2026-05-12
**Status:** Aceito

## Contexto

Os parcels GVDASA precisam realizar chamadas autenticadas às APIs do gateway. As preocupações transversais como injeção de tokens de autenticação, interceptors de refresh, retry em erros transitórios e logging de diagnóstico são responsabilidades da plataforma — não de cada produto individualmente. Reimplementar essas preocupações em cada parcel introduziria inconsistências e duplicação de lógica crítica de segurança.

## Decisão

Utilizar exclusivamente o **`GVHttpClient`** do pacote `@gvdasa/gv-core` como cliente HTTP, exposto como singleton via `src/services/api/axios-config/AxiosConfig.ts`. O singleton é criado uma única vez, recebendo o objeto `Environment` do parcel (que contém as URLs de API por ambiente). Todos os services de API do parcel importam esta instância `Api`; nenhum parcel cria instâncias adicionais de Axios diretamente. Erros de chamadas HTTP são capturados e relançados como `ApiException` para padronizar o tratamento na camada de componentes.

## Alternativas Consideradas

| Alternativa | Motivo do descarte |
|---|---|
| `fetch` nativo do browser | Sem interceptors nativos; reimplementação de autenticação e retry seria necessária em cada parcel |
| Instância própria de `axios` por parcel | Duplicação de lógica de autenticação, interceptors e logging; risco de divergências de comportamento entre produtos |
| React Query com `fetchClient` customizado | Complementar (TanStack Query é usado para caching), mas não substitui o cliente HTTP base com autenticação |

## Consequências

**Positivos:**
- Autenticação, refresh de token e retry são gerenciados centralmente pelo `GVHttpClient` sem código nos parcels.
- Atualização de comportamento de autenticação no `@gvdasa/gv-core` propaga-se a todos os parcels sem mudanças de código.
- `ApiException` padroniza o contrato de erros: todos os services lançam a mesma classe, simplificando o tratamento de erros nos hooks e componentes.

**Negativos:**
- O parcel fica dependente da versão do `@gvdasa/gv-core` — breaking changes no `GVHttpClient` exigem atualização coordenada.
- Em testes unitários, o `GVHttpClient` precisa ser mockado (importação indireta via `Api`), exigindo setup de `vi.mock` nos testes de services.
- A abstração oculta a configuração do Axios — desenvolvedores não podem adicionar interceptors customizados diretamente.

## Referências

- `src/services/api/axios-config/AxiosConfig.ts` — instância singleton
- `src/services/api/ApiException.ts` — wrapper de erros HTTP
- `src/shared/environment/index.ts` — variáveis de ambiente injetadas no cliente
- ADR 0002 — React 18 + TypeScript como stack de desenvolvimento
