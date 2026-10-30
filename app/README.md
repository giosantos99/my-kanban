# MyKanban

Aplicação de gerenciamento de tarefas baseada no modelo Kanban, desenvolvida com Vue.js e Quasar Framework.

O projeto permite criar, editar, excluir e organizar tarefas entre diferentes etapas do fluxo de trabalho, com persistência dos dados utilizando o armazenamento local do navegador (localStorage).

![MyKanban](./public/mykanban-preview.gif)

## Funcionalidades

- Criar tarefas
- Editar tarefas
- Excluir tarefas
- Excluir todas as tarefas de uma coluna
- Mover tarefas entre as colunas
- Reordenar tarefas dentro de uma coluna
- Persistir as tarefas utilizando JSON Server
- Interface responsiva para desktop e dispositivos móveis
- Feedback visual através de notificações
- Modal para movimentação de tarefas em dispositivos móveis

## Tecnologias

- Vue.js 3
- Quasar Framework
- JavaScript
- Vue Draggable
- Sass
- localStorage

## Estrutura do Kanban

As tarefas são organizadas em três etapas:

- **Fazer**
- **Fazendo**
- **Feito**

É possível mover uma tarefa entre as etapas e alterar sua posição dentro de cada coluna.

## Instalação

1. Clone o repositório:

```bash
git clone <https://github.com/giosantos99/my-kanban.git>
```

2. Entre na pasta do projeto:

cd my-kanban/app

3. Instale as dependências:

npm install


## Executando o projeto

Inicie a aplicação em ambiente de desenvolvimento:

```bash
npm run dev
```

O Quasar iniciará a aplicação em:

http://localhost:9000

## Persistência dos dados

As tarefas são armazenadas no localStorage do navegador, permitindo que os dados permaneçam salvos mesmo após atualizar ou fechar a página.

**Observação**: como os dados são armazenados localmente, eles ficam disponíveis apenas no navegador e dispositivo utilizados. Não existe sincronização automática entre diferentes dispositivos ou navegadores.

## Build para produção

Para gerar a versão de produção da aplicação, execute:

```bash
npm run build
```

Os arquivos gerados estarão disponíveis no diretório de build configurado pelo Quasar.