-----

### **Documento de Projeto e Estruturação: Microsserviço de Tarefas (`tasks-service`)**

**Versão:** 1.0  
**Data:** 10 de setembro de 2025  
**Autor:** Alan Mateus

#### **1. Visão Geral e Arquitetura**

O `tasks-service` é um microsserviço independente cuja única responsabilidade é gerenciar todas as operações relacionadas a tarefas, atribuições e seus vínculos com outras entidades do ecossistema AMTech.

  * **Arquitetura:** Microsserviço RESTful.
  * **Princípio Chave:** **Multi-Tenant.** O serviço será projetado desde a primeira linha de código para ser multi-inquilino. O isolamento de dados por `tenant_id` é um requisito não-funcional crítico.
  * **Comunicação:** O serviço exporá uma API REST que será consumida primariamente pelo Gateway BFF/API, garantindo que nenhuma lógica de negócio seja exposta diretamente ao frontend.

-----

#### **2. Stack Tecnológica Detalhada**

A stack foi escolhida para garantir consistência, performance e produtividade dentro do ecossistema AMTech.

| Categoria                | Tecnologia / Ferramenta                   | Justificativa                                                                                             |
| :----------------------- | :---------------------------------------- | :-------------------------------------------------------------------------------------------------------- |
| **Framework & Linguagem** | **NestJS (v10+) & TypeScript (v5+)** | Padrão do ecossistema. Fornece arquitetura modular, injeção de dependência e forte integração com TypeScript. |
| **Banco de Dados** | **PostgreSQL (v15+)** | Banco de dados relacional robusto e confiável, já utilizado pela plataforma principal.                   |
| **ORM / Acesso a Dados** | **Prisma (v5+)** | ORM moderno e type-safe que acelera o desenvolvimento, automatiza migrações e previne erros de tipagem com o BD. |
| **Validação de Dados** | `class-validator` & `class-transformer`   | Padrão no NestJS para validação automática e transformação de DTOs (Data Transfer Objects) na camada de API. |
| **Autenticação** | **Passport.js (`passport-jwt` strategy)** | Biblioteca padrão para implementar a validação dos JWTs que virão do BFF, garantindo acesso seguro aos endpoints. |
| **Containerização** | **Docker & Docker Compose** | Garante um ambiente de desenvolvimento e produção consistente e facilita a orquestração com Kubernetes no futuro. |
| **Testes** | **Jest** | Framework de testes padrão do NestJS, para testes unitários e de integração.                             |
| **Qualidade de Código** | **ESLint & Prettier** | Mantém a consistência e a qualidade do código em conformidade com o restante do monorepo.                 |

-----

#### **3. Estrutura do Repositório (Monorepo-Ready)**

O repositório `tasks-service` deve seguir a estrutura padrão de um projeto NestJS, que é naturalmente modular e fácil de migrar.

```
/tasks-service
├── prisma/
│   ├── schema.prisma             # Definição do modelo de dados
│   └── migrations/               # Migrações automáticas do banco
├── src/
│   ├── modules/
│   │   └── tasks/
│   │       ├── dto/              # DTOs de criação e atualização
│   │       ├── entities/         # Entidades do Prisma
│   │       ├── tasks.controller.ts # Endpoints da API
│   │       ├── tasks.service.ts    # Lógica de negócio
│   │       └── tasks.module.ts     # Módulo NestJS
│   ├── core/
│   │   ├── auth/                 # Lógica de autenticação (validação JWT)
│   │   └── common/               # Decorators, guards, etc.
│   ├── app.module.ts
│   └── main.ts
├── test/
│   ├── tasks.e2e-spec.ts         # Testes End-to-End
│   └── ...
├── .env                          # Variáveis de ambiente (NÃO versionar)
├── .eslintrc.js
├── .prettierrc
├── docker-compose.yml            # Orquestração do container do serviço e do BD
├── Dockerfile                    # Definição do container da aplicação
├── package.json
└── tsconfig.json
```

-----

#### **4. Modelo de Dados Inicial (`prisma/schema.prisma`)**

Este é o esquema inicial para o banco de dados. Ele já inclui a lógica multi-tenant e os relacionamentos.

```prisma
// This is your Prisma schema file,
// learn more about it in the docs: https://pris.ly/d/prisma-schema

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Task {
  id          String   @id @default(uuid())
  tenantId    String   @map("tenant_id") @db.Uuid
  title       String
  description String?
  status      String   @default("todo") // todo, in_progress, done
  priority    String   @default("medium") // low, medium, high
  dueDate     DateTime? @map("due_date")

  assignedToUserId String? @map("assigned_to_user_id") @db.Uuid
  createdByUserId  String  @map("created_by_user_id") @db.Uuid

  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")

  associations TaskAssociation[]

  @@map("tasks")
}

model TaskAssociation {
  id         String @id @default(uuid())
  tenantId   String @map("tenant_id") @db.Uuid
  entityType String @map("entity_type") // customer, product, sale
  entityId   String @map("entity_id") @db.Uuid

  task   Task   @relation(fields: [taskId], references: [id], onDelete: Cascade)
  taskId String @map("task_id")

  @@map("task_associations")
}
```

-----

#### **5. Definição dos Endpoints da API (v1)**

A seguir, a lista inicial de endpoints que o serviço deve expor.

| Método | Endpoint                    | Descrição                                         |
| :----- | :-------------------------- | :------------------------------------------------ |
| `POST`   | `/tasks`                    | Cria uma nova tarefa.                             |
| `GET`    | `/tasks`                    | Lista todas as tarefas (com filtros via query params). |
| `GET`    | `/tasks/:id`                | Busca uma tarefa específica pelo seu ID.          |
| `PATCH`  | `/tasks/:id`                | Atualiza parcialmente uma tarefa (título, status, etc). |
| `DELETE` | `/tasks/:id`                | Exclui uma tarefa.                                |
| `POST`   | `/tasks/:id/associations`   | Associa uma tarefa a uma entidade (cliente, produto). |
| `DELETE` | `/associations/:assocId`    | Remove uma associação de uma tarefa.              |

-----

#### **6. Guia de Setup (Passo a Passo para Iniciar)**

1.  **Criar Repositório e Iniciar Projeto:**

    ```sh
    # Crie o repositório no GitHub
    git clone [URL_DO_SEU_REPO] tasks-service
    cd tasks-service
    pnpm init # ou use o NestJS CLI para um bootstrap completo
    nest new . --skip-git --package-manager=pnpm
    ```

2.  **Adicionar Prisma:**

    ```sh
    pnpm add prisma
    pnpm exec prisma init --datasource-provider postgresql
    # Copie o conteúdo do Modelo de Dados acima para o arquivo `prisma/schema.prisma`
    ```

3.  **Configurar Docker para Ambiente de Desenvolvimento:**

      * Crie o arquivo `Dockerfile` na raiz do projeto.
        ```dockerfile
        FROM node:20-alpine
        WORKDIR /usr/src/app
        COPY package*.json ./
        RUN pnpm install --frozen-lockfile
        COPY . .
        RUN pnpm build
        EXPOSE 3000
        CMD ["node", "dist/main"]
        ```
      * Crie o arquivo `docker-compose.yml`.
        ```yaml
        version: '3.8'
        services:
          tasks-db:
            image: postgres:15
            container_name: tasks-db
            environment:
              POSTGRES_USER: user
              POSTGRES_PASSWORD: password
              POSTGRES_DB: tasks_db
            ports:
              - '5433:5432' # Use uma porta diferente para não conflitar com outros BDs
            volumes:
              - tasks-db-data:/var/lib/postgresql/data
          
          tasks-service:
            build: .
            container_name: tasks-service
            depends_on:
              - tasks-db
            ports:
              - '3001:3000' # Exponha o serviço em uma porta local
            environment:
              DATABASE_URL: "postgresql://user:password@tasks-db:5432/tasks_db?schema=public"
            command: >
              sh -c "pnpm exec prisma migrate deploy && pnpm start:prod"

        volumes:
          tasks-db-data:
        ```

4.  **Executar o Ambiente:**

    ```sh
    # Crie um arquivo .env e adicione a variável DATABASE_URL para uso local sem docker
    # DATABASE_URL="postgresql://user:password@localhost:5433/tasks_db?schema=public"

    # Rodar os containers
    docker-compose up -d --build

    # Para desenvolvimento local (sem docker, mas com o BD do docker rodando)
    pnpm exec prisma migrate dev --name init
    pnpm start:dev
    ```

### **Conclusão e Próximos Passos Imediatos**

Este documento fornece um blueprint completo para iniciar o desenvolvimento do `tasks-service`. Os próximos passos para o desenvolvedor são:

1.  **Setup do Ambiente:** Seguir o guia passo a passo para clonar o repositório, instalar dependências e iniciar o ambiente Docker.
2.  **Implementar Autenticação:** Criar um `AuthGuard` que valide o JWT e extraia o `tenant_id` do token, disponibilizando-o para os serviços.
3.  **Desenvolver o CRUD:** Implementar o `TasksModule` com o controller e service para gerenciar as operações básicas definidas nos endpoints.
4.  **Escrever Testes:** Criar testes unitários para o `TasksService` para garantir que a lógica de negócio (especialmente a de multi-tenancy) esteja correta.
