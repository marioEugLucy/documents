---
applyTo: "**/*.{ts,tsx,js,jsx}"
---

## Princípios Arquiteturais

- Siga os princípios SOLID em toda implementação
- Prefira composição sobre herança
- Mantenha baixo acoplamento e alta coesão entre módulos
- Respeite as fronteiras de camada do projeto:
  - `pages/` orquestra rotas e layout, delegando lógica de busca de dados e gerenciamento de estado para `services/` e `hooks/` respectivamente, e lógica de apresentação para `components/`
  - `components/` é reutilizável e não conhece `pages/`
  - `services/api/` e `shared/` não importam de `pages/` ou `components/`
  - `hooks/` pode importar de `services/` e `shared/`, mas nunca de `pages/`

  | Camada | Pode importar de | Não pode importar de |
  |---|---|---|
  | `pages/` | `components/`, `hooks/`, `services/`, `shared/` | — |
  | `components/` | `hooks/`, `services/`, `shared/` | `pages/` |
  | `hooks/` | `services/`, `shared/` | `pages/`, `components/` |
  | `services/api/` | `shared/` | `pages/`, `components/`, `hooks/` |
  | `shared/` | — | `pages/`, `components/`, `hooks/`, `services/` |

## Arquitetura single-spa (Micro Frontend)

- Cada parcel é isolado: não compartilhe estado global entre parcels — use eventos customizados do hub ou props via root-config
- Sempre limpe subscriptions, event listeners e timers no ciclo `unmount` ou no cleanup do `useEffect`
- Não importe módulos pesados no escopo de módulo fora dos ciclos de vida (`bootstrap`, `mount`, `unmount`)
- O parcel recebe `theme` e `informacoesDeUsuario` como props do root-config — nunca os busque diretamente por conta própria
- Não altere o DOM fora do elemento montado pelo single-spa

## Comunicação com o Painel GVDASA

### Método de Comunicação
- Toda comunicação com o shell (breadcrumbs, título, confirmações, feedback, não autorizado) deve usar os event dispatchers de `src/shared/events/`
- Nunca chame o shell ou outros parcels via importação direta — sempre via Custom Events

### Gerenciamento de Listeners
- Ao registrar um `addEventListener` para eventos do hub, sempre retorne a função de remoção e invoque-a no cleanup

### Nomeação de Eventos
- Nomeie novos eventos no padrão UPPER_SNAKE_CASE (ex: `CONFIRMATION_EVENT`, `CHANGE_BREADCRUMBS_EVENT`)

## Consumo de APIs

- Use exclusivamente o `Api` exportado de `src/services/api/axios-config/AxiosConfig.ts` (instância `GVHttpClient` do `@gvdasa/gv-core`)
- Não crie instâncias adicionais de `axios` — reutilize o singleton configurado
- Sempre envolva erros de chamadas HTTP com `ApiException` para padronizar o tratamento no parcel
- Não reimplemente retry ou interceptors de autenticação — já são responsabilidade do `GVHttpClient`
- Valide e sanitize entradas do usuário antes de enviá-las à API (use Yup + Formik para formulários)

## Resiliência e Observabilidade

- Use `GVObservable` do `@gvdasa/gv-core` para instrumentação com Application Insights — não implemente solução própria de telemetria
- Trate erros de `ApiException` de forma explícita e exiba feedback ao usuário via `FeedbackEventDispatcher`
- Redirecione para o fluxo de não-autorizado via `UnauthorizedEventDispatcher` em respostas 401/403

## Segurança

- Nunca exponha dados sensíveis em logs ou em estado de componente acessível pelo DevTools
- Valide e sanitize todas as entradas externas
- Use variáveis de ambiente runtime (`SSPA_APP_ENV`, `SSPA_APP_BUILD_VERSION`) — nunca hardcode valores de ambiente
- O parcel deve solicitar apenas as permissões declaradas em `menus/Menus.json` (princípio do menor privilégio)

## ADRs (Architecture Decision Records)

- Toda decisão arquitetural relevante deve ter um ADR em `docs/adr/`
- Formato: `docs/adr/NNNN-titulo-da-decisao.md`
- Status: Proposto → Aceito → Depreciado → Substituído
