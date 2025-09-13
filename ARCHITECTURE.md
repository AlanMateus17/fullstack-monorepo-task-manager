# Estrutura e Arquitetura do Projeto AgileFlow

## 1. Visão Geral

Este documento é o guia oficial para a estrutura de pastas e a arquitetura do projeto AgileFlow. O objetivo é manter o código organizado, consistente, escalável e de fácil manutenção.

O projeto utiliza uma arquitetura **Monorepo** gerenciada com `pnpm workspaces`, separando o código em três pacotes principais:

- `server`: A aplicação backend (API) construída com **NestJS**.
- `web`: A aplicação frontend (SPA) construída com **React**.
- `shared-types`: Um pacote para compartilhar tipos TypeScript entre o frontend e o backend.

## 2. Estrutura Raiz do Monorepo (`/`)

A raiz do projeto contém arquivos de configuração globais que governam o ambiente de desenvolvimento, a qualidade do código e a automação.

| Arquivo / Pasta           | Propósito                                                                                              |
| :------------------------ | :----------------------------------------------------------------------------------------------------- |
| **`packages/`**           | Contém os pacotes independentes do monorepo (`server`, `web`, `shared-types`).                         |
| **`.github/`**            | Configurações de automação do GitHub Actions (CI/CD, testes automáticos).                              |
| **`docker-compose.yml`**  | Orquestra os contêineres (backend, frontend, banco de dados) para o ambiente de desenvolvimento local. |
| **`.env.example`**        | Arquivo de exemplo que lista todas as variáveis de ambiente necessárias para rodar o projeto.          |
| **`.eslintrc.js`**        | Configuração principal do ESLint para garantir a qualidade e o padrão do código.                       |
| **`.prettierrc`**         | Regras para a formatação automática do código com o Prettier.                                          |
| **`.gitignore`**          | Lista de arquivos e pastas que devem ser ignorados pelo Git.                                           |
| **`pnpm-workspace.yaml`** | Arquivo que define os workspaces do pnpm (gerenciador do monorepo).                                    |
| **`tsconfig.base.json`**  | Configurações base do TypeScript que são estendidas por outros pacotes.                                |
| **`README.md`**           | Documentação principal e porta de entrada do projeto.                                                  |
| **`ARCHITECTURE.md`**     | Este próprio arquivo, detalhando a arquitetura.                                                        |

## 3. O Backend (`packages/server`)

Construído com **NestJS**, o backend é responsável pela API, lógica de negócio, autenticação e comunicação com o banco de dados.

| Localização                 | O que criar aqui                                  | Descrição                                                                                                                                                                                                                                            |
| :-------------------------- | :------------------------------------------------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`./Dockerfile`**          | `Dockerfile`                                      | Instruções para construir a imagem Docker da aplicação backend para produção.                                                                                                                                                                        |
| **`./prisma/`**             | `schema.prisma`, `migrations/`, `seed.ts`         | **schema.prisma**: define todo o esquema do banco de dados (tabelas, colunas, relações). **migrations/**: pasta gerenciada pelo Prisma com o histórico de alterações do banco. **seed.ts**: script para popular o banco de dados com dados iniciais. |
| **`src/`**                  | Módulos de funcionalidades.                       | Raiz do código-fonte do backend.                                                                                                                                                                                                                     |
| `src/auth/`                 | `auth.module.ts`, `auth.service.ts`, etc.         | Módulo dedicado à autenticação (login, registro, validação de JWT).                                                                                                                                                                                  |
| `src/projects/`             | `projects.module.ts`, `projects.service.ts`, etc. | Módulo para uma funcionalidade específica (ex: Projetos).                                                                                                                                                                                            |
| `src/tasks/`                | -                                                 | Exemplo de um módulo de funcionalidade.                                                                                                                                                                                                              |
| ↳ `entities/task.entity.ts` | Entidades / Models                                | Representa a estrutura de uma tabela do banco de dados.                                                                                                                                                                                              |
| ↳ `dto/create-task.dto.ts`  | Data Transfer Objects (DTOs)                      | Define o formato e as regras de validação para os dados que chegam pela API.                                                                                                                                                                         |
| ↳ `tasks.resolver.ts`       | Resolvers (GraphQL)                               | Define os pontos de entrada da API para uma funcionalidade, recebendo requisições e retornando respostas.                                                                                                                                            |
| ↳ `tasks.service.ts`        | Serviços                                          | **Contém a lógica de negócio principal.** É o "cérebro" da funcionalidade.                                                                                                                                                                           |
| ↳ `tasks.module.ts`         | Módulos                                           | Agrupa todos os arquivos relacionados a uma funcionalidade, organizando a injeção de dependências.                                                                                                                                                   |
| `src/app.module.ts`         | Módulo Raiz                                       | O módulo principal que importa todos os outros módulos de funcionalidades.                                                                                                                                                                           |
| `src/main.ts`               | Ponto de Entrada                                  | O arquivo que inicializa e executa a aplicação NestJS.                                                                                                                                                                                               |

## 4. O Frontend (`packages/web`)

Construído com **React (Vite)**, o frontend é uma **Single Page Application (SPA)**. Existe apenas um `index.html`, e todas as "páginas" são componentes React (`.tsx`) renderizados dinamicamente.

| Localização                   | O que criar aqui                          | Descrição                                                                                                                     |
| :---------------------------- | :---------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------- |
| **`./index.html`**            | `index.html`                              | A única página HTML do projeto. Serve como a "casca" onde a aplicação React é montada.                                        |
| **`./Dockerfile`**            | `Dockerfile`                              | Instruções para construir os arquivos estáticos do frontend e servi-los com Nginx em produção.                                |
| **`public/`**                 | `favicon.ico`, `robots.txt`               | Arquivos estáticos que são copiados diretamente para a pasta de build.                                                        |
| **`src/`**                    | Pastas de código-fonte.                   | Raiz do código-fonte do frontend.                                                                                             |
| `src/main.tsx`                | Ponto de Entrada JS                       | Arquivo que "monta" a aplicação React na `<div id="root">` do `index.html`.                                                   |
| `src/App.tsx`                 | Componente Raiz                           | O primeiro componente da aplicação, onde geralmente o tema e o roteador são configurados.                                     |
| `src/routes/`                 | `index.tsx`                               | Configuração central do **React Router**, mapeando URLs (ex: `/login`) para componentes de página.                            |
| `src/components/`             | Componentes Reutilizáveis                 | **`common/`**: componentes atômicos (Button, Input, Card). **`layout/`**: componentes de estrutura (Header, Sidebar, Footer). |
| `src/features/`               | Componentes de Funcionalidade / "Páginas" | Onde as telas principais são construídas, montando componentes menores. Ex: `features/Kanban/KanbanView.tsx`.                 |
| `src/hooks/`                  | `useAuth.ts`, `useApi.ts`                 | Hooks customizados para encapsular e reutilizar lógica de estado e efeitos.                                                   |
| `src/lib/` ou `src/services/` | `apiClient.ts`, `taskService.ts`          | Lógica para se comunicar com a API do backend. Centraliza as chamadas GraphQL/HTTP.                                           |
| `src/store/`                  | `userStore.ts`                            | Configuração e definições do estado global da aplicação (usando Zustand/Redux).                                               |
| `src/styles/`                 | `theme.ts`, `global.css`                  | Arquivos para estilos globais e definição de tema (cores, fontes).                                                            |
| `src/types/`                  | `index.ts`                                | Definições de tipos e interfaces específicas do frontend.                                                                     |

## 5. Pacotes Compartilhados (`packages/shared-types`)

| Localização    | O que criar aqui                                | Descrição                                                                                                                                                                                  |
| :------------- | :---------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`index.ts`** | `interface Task { ... }`, `type User = { ... }` | Definições de tipos e interfaces TypeScript que precisam ser consistentes entre o backend e o frontend. Isso evita duplicar código e garante que a API e o cliente "falem a mesma língua". |
