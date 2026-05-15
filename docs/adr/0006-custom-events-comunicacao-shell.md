# 0006 — Custom Events para Comunicação entre Parcel e Shell GVDASA

**Data:** 2026-05-12
**Status:** Aceito

## Contexto

Os parcels precisam comunicar-se com o shell GVDASA para atualizar o título da página, os breadcrumbs, exibir mensagens de feedback (toasts/snackbars), solicitar confirmações de ação e sinalizar situações de não-autorização (401/403). O parcel não pode importar diretamente módulos do shell ou de outros parcels, pois isso criaria acoplamento estático entre bundles independentes e quebraria o isolamento do modelo micro frontend.

## Decisão

Toda comunicação do parcel com o shell é feita via **Custom Events** da Web API (`CustomEvent` + `dispatchEvent`). Cada tipo de comunicação possui um dispatcher dedicado em `src/shared/events/`, com nomes em UPPER_SNAKE_CASE:

| Dispatcher | Evento | Finalidade |
|---|---|---|
| `ChangeBreadcrumbsEventDispatcher` | `CHANGE_BREADCRUMBS_EVENT` | Atualizar breadcrumbs |
| `ChangeTitleEventDispatcher` | `CHANGE_TITLE_EVENT` | Atualizar título da página |
| `FeedbackEventDispatcher` | `FEEDBACK_EVENT` | Exibir toast/snackbar |
| `ConfirmationEventDispatcher` | `CONFIRMATION_EVENT` | Solicitar confirmação ao usuário |
| `UnauthorizedEventDispatcher` | `UNAUTHORIZED_EVENT` | Redirecionar fluxo de não-autorizado |

O shell escuta esses eventos no `window` e reage adequadamente. O parcel nunca referencia o shell por importação direta.

## Alternativas Consideradas

| Alternativa | Motivo do descarte |
|---|---|
| Importação direta de módulos do shell | Cria acoplamento estático entre bundles; viola o isolamento do single-spa |
| Props drilling do root-config | Funciona apenas para dados conhecidos em tempo de montagem; não suporta comunicação assíncrona durante o ciclo de vida |
| Zustand/Redux global compartilhado | Exigiria que shell e parcels importem o mesmo módulo de store, recriando acoplamento de bundle |
| `postMessage` / BroadcastChannel | Mais complexo e destinado a contextos de iframe ou workers; excessivo para comunicação same-origin |

## Consequências

**Positivos:**
- Desacoplamento total entre parcel e shell: cada um evolui independentemente sem quebrar contratos de importação.
- Fácil de testar: basta verificar que o evento correto foi disparado com `window.addEventListener` + spy.
- Novos tipos de evento podem ser adicionados no shell sem alteração nos parcels existentes.

**Negativos:**
- Contrato dos eventos é implícito (sem tipagem no `CustomEvent.detail` entre shell e parcel) — uma mudança no formato do payload pode quebrar silenciosamente.
- Eventos devem ser removidos com `removeEventListener` no cleanup do `useEffect` para evitar listeners fantasmas após `unmount` do parcel.
- Sem mecanismo de resposta síncrona: confirmações (`CONFIRMATION_EVENT`) são assíncronas por natureza, exigindo callbacks ou Promises no payload do evento.

## Referências

- `src/shared/events/` — todos os dispatchers de eventos
- `src/shared/events/FeedbackEventDispatcher.ts` — exemplo completo com payload tipado
- `src/shared/events/UnauthorizedEventDispatcher.ts` — dispatcher de não-autorizado
- ADR 0001 — Single-SPA como framework de micro frontend
