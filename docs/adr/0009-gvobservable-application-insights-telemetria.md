# 0009 — GVObservable (Application Insights) para Telemetria

**Data:** 2026-05-12
**Status:** Aceito

## Contexto

O time de plataforma precisa de visibilidade sobre erros, performance e uso dos parcels em produção. Cada produto deve enviar telemetria para o Azure Application Insights sem que cada time implemente sua própria integração com o SDK do Application Insights, evitando divergências de configuração, duplicação de código e risco de exposição de dados sensíveis por instrumentação incorreta.

## Decisão

Utilizar **`GVObservable`** do pacote `@gvdasa/gv-core` para inicializar a telemetria com Application Insights. A inicialização ocorre uma única vez no nível do componente `App`, condicionada à presença da `APP_INSIGHTS_CONNECTION_STRING` no `Environment`. A connection string é derivada do ambiente em runtime (`SSPA_APP_ENV`) — em ambiente `development` (local) a string é vazia e a telemetria não é inicializada, evitando poluição de dados de produção com eventos de desenvolvimento.

```tsx
// src/App.tsx
if (Environment.APP_INSIGHTS_CONNECTION_STRING !== '') {
  core.GVObservable(Environment.APP_INSIGHTS_CONNECTION_STRING);
}
```

## Alternativas Consideradas

| Alternativa | Motivo do descarte |
|---|---|
| SDK `@microsoft/applicationinsights-web` diretamente | Configuração manual de cada parcel para correlação de traces, amostragem e PII; risco de configurações divergentes entre produtos |
| Sentry | Requer contrato separado e integração customizada; o ecossistema GVDASA já usa Application Insights como plataforma de observabilidade |
| Logs via `console.*` apenas | Sem agregação centralizada, sem alertas automáticos, sem correlação de traces entre chamadas de API |
| OpenTelemetry + exportador customizado | Overhead de configuração elevado; `GVObservable` já abstrai isso com padrões corporativos |

## Consequências

**Positivos:**
- Inicialização padronizada garante que todos os parcels enviam telemetria com as mesmas configurações de correlação e amostragem.
- Não inicializar em `development` evita que eventos locais poluam dashboards de produção/homologação.
- Atualizações de configuração de telemetria (ex.: sampling rate, PII filtering) propagam-se a todos os parcels via atualização do `@gvdasa/gv-core`.

**Negativos:**
- `GVObservable` é chamado no escopo de módulo de `App.tsx` (fora do ciclo de vida React), o que pode causar dupla inicialização se o App for remontado pelo single-spa — monitorar se a biblioteca aceita múltiplas chamadas sem duplicar listeners.
- A connection string é hardcoded por ambiente na função `getAppInsightsConnectionString()` do `Environment` — uma rotação de connection string exige novo build e deploy.
- Sem a `APP_INSIGHTS_CONNECTION_STRING`, erros em produção não são capturados automaticamente pelo Application Insights.

## Referências

- `src/App.tsx` — inicialização do `GVObservable`
- `src/shared/environment/index.ts` — `APP_INSIGHTS_CONNECTION_STRING` por ambiente
- [Azure Application Insights documentation](https://learn.microsoft.com/azure/azure-monitor/app/app-insights-overview)
- ADR 0002 — React 18 + TypeScript como stack de desenvolvimento
