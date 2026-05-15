---
applyTo: "src/components/**/*.{tsx,jsx,ts,js}"
description: "Regras de componentes React: estrutura, nomenclatura, estado, estilização, acessibilidade e segurança."
---

## Estrutura de Componentes

- Use componentes funcionais com hooks (sem class components)
- Mantenha componentes com menos de 200 linhas
- Extraia lógica reutilizável em custom hooks (prefixo `use`)
- Props sempre tipadas via `interface` (TypeScript)
- Prefira o padrão `export const ComponentName: React.FC<Props>` já adotado no projeto

## Nomenclatura

- Componentes: PascalCase (ex: `UserCard.tsx`)
- Hooks: camelCase com prefixo `use` (ex: `useAuthUser.ts`)
- Constantes: UPPER_CASE
- Funções e variáveis: camelCase

## Estado e Efeitos

- Prefira `useState` para estado local
- Evite prop drilling além de 2 componentes intermediários entre a origem e o consumidor — use Context ou estado global
- Sempre limpe side effects no retorno do `useEffect`

## Estilização

- Use os componentes de `@gvdasa/react-components` e `@mui/material` como base visual
- Estilizações customizadas devem usar `sx` prop (MUI) ou `styled` do `@emotion/styled`
- Nunca use estilos inline (`style={{}}`) para layout estrutural
- Respeite os tokens de design (cores, espaçamentos, tipografia) do tema GVdasa

## Acessibilidade

- Todos os elementos interativos devem ser acessíveis via teclado
- Inclua atributos `aria-label` quando necessário
- Garanta contraste mínimo WCAG AA

## Segurança

- Nunca use `dangerouslySetInnerHTML` sem sanitização
- Não exponha chaves de API no código cliente
- Valide todas as entradas do usuário antes de processar (use Yup + Formik para formulários)
