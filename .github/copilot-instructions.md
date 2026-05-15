---
applyTo: "*"
---

# Instruções do Copilot — Template Frontend PCL (single-spa)

## 1. Visão Geral

Repositório de referência para criar **parcels** (micro frontends) React alojados sob domínios GVdasa (`pcl-<nome>.gvdasa.com.br`). Integrado ao hub GVDASA via **single-spa**, pipelines Azure DevOps e infraestrutura Terraform. O template fornece:

- Estrutura completa de parcel single-spa com React 18 + TypeScript
- Build webpack com injeção de cache busting (`?v=<BuildId>`)
- Integração com hub GVDASA (menus, autenticação, tema corporativo)
- Pipelines Azure DevOps (build, deploy, pull-request)
- Infraestrutura Terraform (storage, CDN, configurações Azure)

**Ao clonar**: Executar uma troca sistemática dos identificadores de produto (ex.: `exemplo-de-produto` → `seu-modulo`).

## 2. Tecnologias

- **Runtime**: Node.js ≥ 18.18.0
- **Frontend**: React 18, TypeScript, single-spa 6
- **Build**: Webpack 5, Babel, Vite (testes)
- **UI**: Material-UI 7 + componentes GVdasa (`@gvdasa/react-components`)
- **Roteamento**: React Router 6 (por parcel)
- **Requisições HTTP**: Axios + TanStack Query (SWR como alternativa)
- **Formulários**: Formik + Yup (validação)
- **Testes**: Vitest + Testing Library (Happy DOM)
- **Linting**: ESLint + Prettier
- **CI/CD**: Azure DevOps Pipelines
- **Infraestrutura**: Terraform 1.7.3 (Storage, CDN, Resource Group)

## 3. Comandos Essenciais

### Instalação e Setup
```bash
# Instalar dependências
npm install

# (Opcional) Renovar token VSTS (se falhar npm install)
npm run refreshVSToken
```

### Desenvolvimento Local
```bash
# Iniciar dev server (localhost:3003)
# Servirá gvdasa-root-config + gvdasa-index para integração local
npm start

# Erros comuns:
# - "Cannot find module '@gvdasa/...'": Executar refreshVSToken + npm install
# - Porta 3003 em uso: Alterar --port 3003 em package.json > start
# - CORS localhost: Usar gvdasa-root-config.ts em public-dev/ para mock
```

### Build e Testes
```bash
# Build production (webpack mode=production)
npm run build

# Executar testes uma única vez
npm test

# Modo watch (desenvolvimento)
npm run test:watch

# Linting com auto-fix
npm run lint
```

### Artefatos
- **dist/gvdasa-index.js** — bundle público do parcel
- **dist/index.html** — HTML com import map (dev local)
- **dist/gvdasa-index.d.ts** — tipos TypeScript exportados

## 4. Estrutura de Pastas

```
.github/                 → Configs GitHub (instruções, workflows)
pipelines/               → Azure DevOps pipelines (build/deploy/PR + Terraform)
  ├─ build.yml         → Webpack build + versioning (BuildId)
  ├─ deploy.yml        → Deploy to Storage + CDN (via template DEVOPS)
  ├─ pullrequest.yml   → Validação de Pull Request (lint, testes, build)
  ├─ config.yml        → Variáveis (Ambiente, ArtifactName, Project)
  └─ terraform/        → IaC (storage, CDN, resource group)
public-dev/             → Assets para dev local
  ├─ gvdasa-root-config.ts → Mock root-config (single-spa)
  └─ index.ejs         → Template HTML (import map, estilos corporativos)
src/
  ├─ App.tsx           → Componente raiz (BrowserRouter + basename)
  ├─ gvdasa-index.ts   → Entry single-spa (bootstrap, mount, unmount)
  ├─ routes/index.tsx  → Rotas do parcel (React Router)
  ├─ components/       → Componentes React reutilizáveis
  ├─ pages/            → Páginas por rota
  ├─ services/api/     → Axios config + endpoints
  ├─ contexts/         → React contexts (autenticação, usuário)
  ├─ hooks/            → Custom hooks
  ├─ shared/           → Utilitários (environment, events, formatters)
  └─ @types/           → Declarações TypeScript customizadas
test/                   → Testes Vitest + setup
menus/Menus.json       → Registro de menus no hub (id, rotas, ícones, permissões)
webpack.config.js      → Webpack customizado (orgName="gvdasa", alias '@')
vite.config.ts         → Vite para testes (vitest)
tsconfig.json          → TypeScript (strict, paths: '@/*' → src/*)
babel.config.json      → Babel (presets React/TS, plugins runtime)
```

## 5. Convenções

- **Nomenclatura parcel**: kebab-case (ex.: `exemplo-de-produto`, `seu-modulo-contabil`)
- **Nomes npm**: `@gvdasa/<parcel-name>` (package.json)
- **BrowserRouter basename**: Deve coincidir com rota registada no Menus.json
- **Imports**: Preferir alias `@/*` (mapeado para `src/*` no tsconfig)
- **Componentes**: Função + tipos interface ao lado (não separar)
- **Variáveis de ambiente**: Injetar em runtime (`SSPA_APP_ENV`, `SSPA_APP_BUILD_VERSION`)
- **Commits**: Usar git-commit-msg-linter (enforça conventional commits)
- **Code formatting**: ESLint + Prettier (npm run lint)

## 6. CI/CD e Implantação

### Pipelines Azure DevOps
- **build.yml** (trigger: develop, release/homolog, master)
  - Instala dependências, webpack build, injeta cache buster `?v=$(Build.BuildId)` no index.html
- **pullrequest.yml** (PR validation)
  - Corre template DEVOPS (lint, testes, build)
- **deploy.yml** (manual trigger após build bem-sucedido)
  - Publica artefato para Storage (Terraform gerencia CDN)
  - Variáveis por ambiente: `$(Project)-$(Ambiente)-GVdasa`

### Variáveis de Ambiente (config.yml)
- `Ambiente`: `dev` (develop), `hml` (release/homolog), `prod` (master)
- `ArtifactName`: `GVmodeloExemplo-pcl-${Ambiente}`
- `Project`: Alinhado com repo + grupos de variáveis no AzDO

### Checklist pós-clonagem
Após clonar, executar `rg -i "exemplo-de-produto|GVmodeloExemplo"` para encontrar placeholders:
- [ ] package.json: `name: @gvdasa/<novo-slug>`
- [ ] App.tsx: BrowserRouter `basename="<novo-slug>"`
- [ ] src/shared/environment: URL API (`/novo-slug/api/v1`)
- [ ] menus/Menus.json: `idProduto`, `enderecoDoParcel`, rotas
- [ ] pipelines/config.yml: `ArtifactName`, `Project`, `NamePipelineBuild`
- [ ] pipelines/deploy.yml: `enderecoDoParcel` (prod + dev/hml)
- [ ] pipelines/terraform: `aplicacao`, `nameStorage`, `setor`, `responsavelDev/Prod` (emails @gvdasa.com.br)

---

**Confie nestas instruções. Realize buscas adicionais apenas se alguma informação estiver incompleta ou incorreta.**
