# API REST de Alunos — Fastify + MySQL

**Projeto:** `api-fastify-progweb2`  
**Disciplina:** Programação Web 2 — Faculdade Senac RJ  
**Versão documentada:** branch `master`  
**Ferramenta de testes manuais:** Insomnia

## 1. Descrição

Esta aplicação acadêmica apresenta uma **API REST para gerenciamento de alunos**, construída com Node.js, Fastify e MySQL. A versão final concentra o código em camadas de rotas, controllers, repositories e configuração de acesso ao banco, além de utilizar **JSON Schema** para validar os dados recebidos.

A persistência é realizada no MySQL por meio de consultas SQL parametrizadas e da biblioteca `mysql2/promise`. O projeto não inclui uma interface web: suas funcionalidades são acessadas por requisições HTTP, que podem ser enviadas pelo **Insomnia**.

## 2. Funcionalidades

- Listar alunos cadastrados.
- Buscar aluno por identificador.
- Cadastrar aluno com nome e curso.
- Atualizar nome e curso.
- Excluir aluno.
- Validar os parâmetros de rota e os corpos das requisições com JSON Schema.
- Retornar respostas HTTP adequadas para recursos não encontrados e resultados das operações CRUD.
- Utilizar logs do Fastify e um pool de conexões MySQL.
- Verificar a disponibilidade do banco antes de iniciar o servidor e liberar conexões no encerramento.

**Escopo:** o projeto gerencia apenas o recurso `alunos`; não implementa autenticação, autorização, interface gráfica ou CRUD de cursos separado.

## 3. Tecnologias Utilizadas

| Tecnologia | Utilização |
|---|---|
| Node.js | Execução do JavaScript no servidor e scripts de inicialização. |
| Fastify 5 (`^5.12.1`) | Framework HTTP, rotas, validação e logs. |
| JavaScript (ES Modules) | Organização de arquivos com `import` e `export`. |
| MySQL 8.4 | Banco relacional com persistência dos alunos. |
| `mysql2` (`^3.24.2`) | Pool de conexões e execução de consultas com Promises. |
| Docker Compose | Inicialização opcional do MySQL para o ambiente de estudo. |
| Insomnia | Execução e inspeção manual das requisições da API. |
| npm e Git | Instalação de dependências e versionamento. |

## 4. Arquitetura e Organização do Projeto

```text
api-fastify-progweb2/
├── .env.example
├── .gitignore
├── docker-compose.yml
├── package.json
├── package-lock.json
├── README.md
└── src/
    ├── app.js
    ├── server.js
    ├── controllers/
    │   └── alunoController.js
    ├── database/
    │   └── connection.js
    ├── repositories/
    │   └── alunoRepository.js
    ├── routes/
    │   └── alunoRoutes.js
    └── schemas/
        └── alunoSchemas.js
```

**Fluxo de uma requisição:**

```text
Insomnia → Fastify (app.js) → alunoRoutes.js → alunoController.js
                                                    ↓
                                            alunoRepository.js
                                                    ↓
                                        connection.js → MySQL
```

- `src/app.js`: cria o Fastify, registra rotas, define o tratamento global de erros e o hook de encerramento.
- `src/server.js`: testa o banco, inicia o HTTP e trata sinais de desligamento.
- `src/routes/alunoRoutes.js`: mapeia os métodos e URLs e associa schemas de validação.
- `src/controllers/alunoController.js`: recebe requisições e define respostas HTTP.
- `src/repositories/alunoRepository.js`: executa os comandos SQL.
- `src/database/connection.js`: configura o pool de conexões.
- `src/schemas/alunoSchemas.js`: declara os JSON Schemas de corpo e parâmetros.

Não há camada Service separada nesta implementação.

## 5. Pré-requisitos

- Node.js **20 ou superior**, compatível com Fastify 5 e os parâmetros `--watch` e `--env-file` dos scripts.
- npm e Git.
- Um servidor MySQL acessível; é possível utilizar Docker Compose, quando disponível.
- Um banco `api_rest` com a tabela `alunos`, conforme descrito na seção 15.
- Insomnia para os testes manuais.

## 6. Instalação

```bash
git clone https://github.com/Bentim-Luccas/api-fastify-progweb2.git
cd api-fastify-progweb2
git switch master
npm ci
```

O comando `npm ci` utiliza o `package-lock.json` para instalar as dependências registradas. A criação de `.env` e o preparo do MySQL são etapas adicionais.

## 7. Configuração das Variáveis de Ambiente

Copie o arquivo de exemplo para `.env` na raiz do projeto:

```cmd
copy .env.example .env
```

Exemplo para uso com o MySQL definido no `docker-compose.yml`:

```dotenv
PORT=3000
DB_HOST=localhost
DB_PORT=3306
DB_USER=api_user
DB_PASSWORD=api123
DB_NAME=api_rest
```

As credenciais acima **são didáticas**, conforme o Compose do repositório. Não devem ser reutilizadas em produção. O arquivo `.env` não deve ser enviado ao Git.

## 8. Execução do Projeto

**Caso utilize Docker**, dentro da raiz do repositório:

```bash
docker compose up -d
docker compose ps
```

O Docker Compose disponibiliza o MySQL na porta `3306` do computador e utiliza um volume nomeado para persistência dos dados. **O Compose cria o banco `api_rest`, mas não cria a tabela `alunos` automaticamente.** Prepare a tabela antes de testar o CRUD (seção 15).

Inicie a API:

```bash
npm run dev
```

Para iniciar sem monitoramento de arquivos:

```bash
npm start
```

O Node.js carrega `.env` e executa `src/server.js`. O servidor obtém uma conexão do pool para verificar o MySQL antes de iniciar o Fastify. A aplicação utiliza a porta configurada em `PORT`, por padrão `3000`, e escuta no endereço `0.0.0.0`.

**Sem MySQL acessível**, a inicialização falha; apenas instalar as dependências não permite testar os endpoints. Para encerrar o ambiente Docker após os testes, utilize `docker compose down` (sem `-v` para preservar o volume).

## 9. Endpoints da API

**URL base:** `http://localhost:3000/api/alunos`

| Método | Endpoint | Operação | Resposta esperada |
|---|---|---|---|
| `GET` | `/api/alunos` | Listar alunos | `200 OK` |
| `GET` | `/api/alunos/:id` | Buscar aluno pelo ID | `200 OK` ou `404 Not Found` |
| `POST` | `/api/alunos` | Cadastrar aluno | `201 Created` |
| `PUT` | `/api/alunos/:id` | Atualizar aluno | `200 OK` ou `404 Not Found` |
| `DELETE` | `/api/alunos/:id` | Excluir aluno | `204 No Content` ou `404 Not Found` |

**Atenção ao prefixo:** as rotas são registradas com `prefix: '/api/alunos'`; portanto, não utilize apenas `/alunos` nos testes.

## 10. Exemplos de Requisição e Resposta

**Cadastrar um aluno — POST `/api/alunos`:**

```json
{
  "nome": "Camila Ribeiro",
  "curso": "Análise e Desenvolvimento de Sistemas"
}
```

Resposta ilustrativa (`201`), com ID atribuído pelo banco:

```json
{
  "id": 1,
  "nome": "Camila Ribeiro",
  "curso": "Análise e Desenvolvimento de Sistemas"
}
```

**Listar — GET `/api/alunos`:** resposta com array JSON, incluindo os alunos presentes no MySQL.

**Buscar — GET `/api/alunos/1`:** retorna o objeto encontrado. Para um ID inexistente, o controller responde:

```json
{"message":"Aluno não encontrado"}
```

**Atualizar — PUT `/api/alunos/1`:**

```json
{
  "nome": "Camila R. Ribeiro",
  "curso": "Sistemas de Informação"
}
```

**Excluir — DELETE `/api/alunos/1`:** sucesso com `204 No Content`, sem corpo de resposta.

Os IDs e registros apresentados são exemplos; os valores reais dependem do banco utilizado.

## 11. Autenticação

Não há autenticação ou autorização implementada. A API de estudo não utiliza login, tokens JWT ou controle de permissões. O uso fora de um ambiente controlado exigiria mecanismos adicionais de segurança.

## 12. Tratamento de Erros

- Aluno não encontrado em consulta, atualização ou exclusão: o controller responde `404` com `message: 'Aluno não encontrado'`.
- Falhas internas: o `setErrorHandler()` registra o erro e devolve mensagem genérica com status `500`.
- Dados inválidos: há validação por JSON Schema para `nome`, `curso` e `id`.
- **Limitação identificada:** o tratamento global atual devolve **`404` para erros de validação** (`error.validation`), embora o código `400 Bad Request` seja normalmente mais adequado. Este README descreve o comportamento implementado, não uma correção.
- Falha inicial de conexão MySQL: `src/server.js` registra o erro e encerra o processo.

O tratamento de encerramento em `src/server.js` utiliza `app.close(0)`, trecho que merece revisão na prática; não foi alterado como parte desta documentação.

## 13. Testes

Os testes HTTP manuais são realizados com **Insomnia**. Configure a URL base `http://localhost:3000/api/alunos` e organize as cinco requisições da seção 9.

**Roteiro sugerido:**

1. Cadastrar um aluno com `POST` e conferir `201`.
2. Listar alunos com `GET` e identificar o registro criado.
3. Consultar o ID gerado, conferindo `200`.
4. Atualizar nome e curso com `PUT` e conferir o novo conteúdo.
5. Consultar um ID inexistente e verificar `404`.
6. Excluir o aluno com `DELETE` e conferir `204`.
7. Enviar um corpo inválido para observar a validação e o comportamento documentado na seção 12.

**Validações do JSON Schema:** `id` deve ser inteiro maior ou igual a `1`; `nome` e `curso` são strings obrigatórias com tamanho entre `2` e `100` caracteres; propriedades extras no corpo são rejeitadas conforme a configuração do schema e do Fastify. O projeto não apresenta suíte de testes automatizados configurada no `package.json`.

Não há resultados de testes executados anexados a este README; o roteiro descreve como realizá-los.

## 14. Documentação da API

Os endpoints e exemplos estão documentados neste README. As rotas estão definidas em `src/routes/alunoRoutes.js`, com os contratos de entrada em `src/schemas/alunoSchemas.js`. Não há interface Swagger/OpenAPI configurada no código analisado.

## 15. Modelo de Dados

**Banco:** `api_rest`. O Repository utiliza a tabela `alunos` com as colunas `id`, `nome` e `curso`.

Estrutura SQL compatível com as consultas do projeto, para um **banco de desenvolvimento vazio**:

```sql
USE api_rest;

CREATE TABLE alunos (
  id INT AUTO_INCREMENT PRIMARY KEY,
  nome VARCHAR(100) NOT NULL,
  curso VARCHAR(100) NOT NULL
);
```

Execute o `CREATE TABLE` apenas se a tabela ainda não existir. O SQL acima é uma **sugestão compatível com o Repository**, não um script versionado no repositório. Verifique o esquema existente antes de executar alterações.

## 16. Regras de Negócio

- Um aluno possui `id`, `nome` e `curso`.
- O ID é gerado pelo MySQL (`AUTO_INCREMENT`).
- `nome` e `curso` são obrigatórios nas requisições de cadastro e atualização.
- Consultas utilizam SQL parametrizado (`?`) para separar valores externos da instrução.
- Na atualização e exclusão, `affectedRows` ajuda a identificar a existência do registro.
- Nesta versão, `curso` é texto na tabela `alunos`; não há relacionamento com uma tabela separada de cursos.

## 17. Extras — Programação Web 2

O projeto consolida conceitos da unidade curricular: API REST com Fastify, separação entre Route/Controller/Repository, JSON Schema, async/await, comunicação com MySQL, pool de conexões, logs e ciclo de vida do servidor. A comparação com Express permite observar diferentes formas de configurar rotas e construir respostas HTTP.

## 18. Versionamento e Organização das Branches

Repositório: [Bentim-Luccas/api-fastify-progweb2](https://github.com/Bentim-Luccas/api-fastify-progweb2).

- `branch_20260901`: atividade versionada da aula.
- `master`: versão consolidada documentada neste arquivo.

Fluxo de trabalho recomendado: desenvolver em branch de atividade, registrar commits, revisar as alterações e integrar à `master`. O histórico pode ser conferido com `git log --oneline --graph --all`.

## 19. Autor

Repositório acadêmico: **Bentim-Luccas** — Programação Web 2, Faculdade Senac RJ.

## 20. Licença e Uso Acadêmico

O `package.json` declara a licença `ISC`. Este material integra atividades de estudo da disciplina Programação Web 2. Sua utilização deve observar as condições da licença aplicável e as orientações acadêmicas da instituição. Não contém configuração recomendada para produção.
