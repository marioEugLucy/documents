# 0004 — Material-UI + Design System GVdasa como Biblioteca de Componentes

**Data:** 2026-05-12
**Status:** Aceito

## Contexto

O portal GVDASA exige consistência visual entre todos os produtos hospedados como parcels. O shell injeta um objeto `theme` (Material-UI Theme) em cada parcel via props do root-config. Para que a aparência seja uniforme, os parcels precisam consumir esse tema centralizado e usar os mesmos primitivos de UI.

## Decisão

Adotar **Material-UI 7 (MUI)** como biblioteca base de componentes e **`@gvdasa/react-components`** como design system corporativo que estende o MUI com componentes, tokens e utilitários específicos da plataforma GVDASA. O parcel envolve toda a árvore de componentes com `<GVComponentsProvider theme={theme}>`, onde `theme` é recebido como prop do root-config, garantindo que o tema corporativo seja aplicado globalmente sem que o parcel o defina autonomamente.

## Alternativas Consideradas

| Alternativa | Motivo do descarte |
|---|---|
| Tailwind CSS puro | Sem suporte a theming dinâmico via props; cada parcel teria sua própria folha de estilos, quebrando consistência visual |
| Ant Design | Theming menos flexível e não integrado ao `@gvdasa/react-components`; requer sobrescrita extensiva de CSS |
| Styled-components sem MUI | Aumenta a superfície de implementação de acessibilidade e componentes complexos (DataGrid, DatePicker) que já existem no MUI |

## Consequências

**Positivos:**
- O tema (cores, tipografia, espaçamentos) é definido uma única vez no shell e distribuído a todos os parcels via props — atualização visual sem redeployar produtos.
- `@gvdasa/react-components` fornece DataGrid, formulários, layouts e feedbacks pré-estilizados, reduzindo drasticamente o código de UI repetitivo.
- Acessibilidade (ARIA, foco, contraste) já tratada pelo MUI em componentes complexos.
- `@emotion/react` e `@emotion/styled` são externalizados no Webpack, evitando bundles duplicados entre parcels.

**Negativos:**
- Versão do MUI deve ser mantida em sincronia entre shell e parcels para evitar conflitos de contexto do `ThemeProvider`.
- `@mui/x-data-grid` e `@mui/x-date-pickers` têm licença paga para recursos avançados (Pro/Premium) — usar apenas a versão Community.
- Atualizações de major do MUI (ex.: v5 → v7) podem exigir refatoração de API em todos os parcels simultaneamente.

## Referências

- `src/App.tsx` — uso de `GVComponentsProvider` com o `theme` recebido via props
- `package.json` — dependências `@mui/material`, `@gvdasa/react-components`, `@emotion/react`
- `webpack.config.js` — `externals` para React e Axios (MUI/Emotion não são externalizados, pois o shell não os expõe via SystemJS)
