 PZaaS — API Gateway

Projeto acadêmico desenvolvido para a disciplina de **Arquitetura de Serviços em Nuvem**.

O **PZaaS (Pizza as a Service)** simula uma plataforma distribuída de pizzaria baseada em microsserviços. Este repositório contém a implementação e a documentação do **Serviço 01 — API Gateway**, responsável por atuar como ponto único de entrada e coordenar o acesso aos serviços do ecossistema.

 Integrantes

- Davis
- Vitor Hugo

 Serviço 01 — API Gateway

O API Gateway foi desenvolvido utilizando **n8n Cloud** e centraliza as requisições dos clientes para os diferentes serviços do PZaaS.

Entre suas responsabilidades estão:

- Validação da API Key
- Geração e propagação do `x-pedido-id`
- Rate Limit
- Integração com os microsserviços
- Tratamento padronizado de erros HTTP
- Retry para falhas de comunicação
- Integração com métricas e logs
- Orquestração do fluxo de Checkout

 Arquitetura

Fluxo principal do Checkout:

```text
Cliente
   │
   ▼
API Gateway
   │
   ├──► Identidade
   │
   ├──► Cardápio
   │
   ├──► Estoque (consulta)
   │
   ├──► Pagamento
   │       │
   │       └──► Orquestrador
   │
   └──► Estoque (baixa)
           │
           ▼
        Resposta
```

O serviço de **Pagamento** realiza a integração com o **Orquestrador** durante o fluxo de pagamento. Por isso, o Gateway não realiza uma segunda criação de pedido após o pagamento, evitando duplicidade.

Endpoints do Gateway

| Método | Endpoint | Descrição |
|---|---|---|
| GET | `/health` | Verifica a disponibilidade do Gateway |
| GET | `/v1/menu` | Consulta o Cardápio |
| GET | `/v1/estoque/consulta` | Consulta ingredientes no Estoque |
| POST | `/v1/pagamento` | Encaminha pagamentos |
| POST | `/v1/pedido` | Integração isolada com o Orquestrador |
| POST | `/v1/auth/validar` | Validação de identidade |
| POST | `/v1/clientes` | Cadastro de cliente |
| POST | `/v1/checkout` | Executa o fluxo principal de compra |
| GET | `/v1/metrics` | Exibe métricas operacionais do Gateway |

Base URL:

```text
https://veagabronx2k.app.n8n.cloud/webhook
```

Exemplo:

```text
POST /v1/checkout
```

Autenticação

As requisições protegidas utilizam o header:

```http
x-api-key: <API_KEY>
```

A chave utilizada no ambiente acadêmico não é publicada neste repositório.

O Gateway também gera um identificador de correlação:

```http
x-pedido-id: <ID_GERADO_PELO_GATEWAY>
```

Esse identificador é propagado entre os serviços para permitir rastreabilidade das requisições.

Credenciais, senhas, tokens de clientes e chaves de infraestrutura não são armazenados neste repositório.

Exemplo de Checkout

```json
{
  "token": "<TOKEN_CLIENTE>",
  "metodo_pagamento": "pix",
  "itens": [
    {
      "pizza": "Calabresa",
      "quantidade": 1
    }
  ]
}
```

O Gateway executa as validações necessárias antes de encaminhar a operação aos serviços responsáveis.

Caso uma pizza esteja marcada como indisponível pelo serviço de Cardápio, o Gateway mantém essa condição e retorna, por exemplo:

```json
{
  "error": "PIZZA_INDISPONIVEL",
  "message": "Pizza 'Calabresa' está indisponível."
}
```

Rate Limit

O Gateway possui controle de limite de requisições.

Configuração utilizada:

```text
Limite: 5 requisições
Janela: 60 segundos
```

Ao ultrapassar o limite, o cliente recebe:

```http
HTTP 429 Too Many Requests
```

O comportamento foi validado durante os testes do projeto.

Retry e Fallback

O projeto contempla mecanismo de **Retry** para falhas temporárias dos serviços, como erros `5xx` e timeout.

O contrato do PZaaS também prevê **Fallback A → B** para serviços que possuem versões redundantes.

As URLs oficiais das versões B que não foram disponibilizadas às equipes são registradas na documentação como dependência externa, sem criação de endpoints fictícios.

Redis

Foi utilizada uma instância Redis para demonstração de persistência auxiliar dos clientes cadastrados.

Os registros de demonstração utilizam chaves no formato:

```text
cliente:<id>
```

Dados sensíveis, como senha do cliente, não são armazenados nessa cópia auxiliar.

O Redis utilizado é destinado à demonstração acadêmica e não representa uma arquitetura de persistência de produção.

Métricas

Endpoint:

```http
GET /v1/metrics
```

As métricas disponibilizam informações operacionais do Gateway, incluindo:

- status do serviço;
- versão;
- configuração de Rate Limit;
- configuração de Retry;
- integrações disponíveis;
- estado do Fallback.

Logs

O Gateway possui integração preparada para envio de logs de execução.

Durante os testes, o endpoint externo do serviço de Logger apresentou indisponibilidade/inconsistência de rota. Por esse motivo, a integração foi configurada de forma a **não bloquear o fluxo principal de negócio**.

Tratamento de erros

Entre os principais códigos tratados pelo Gateway estão:

| Código | Significado |
|---:|---|
| 200 | Operação realizada com sucesso |
| 201 | Recurso criado |
| 400 | Requisição inválida |
| 401 | Não autorizado |
| 403 | Operação proibida |
| 404 | Recurso não encontrado |
| 422 | Regra de negócio não atendida |
| 429 | Limite de requisições excedido |
| 500 | Erro interno de serviço |
| 503 | Serviço temporariamente indisponível |

Testes

Os endpoints foram testados utilizando **Postman**.

Entre os cenários validados estão:

- API Key válida e inválida
- Token de identidade válido e inválido
- Pizza inexistente
- Pizza indisponível
- Consulta de estoque
- Pagamento
- Criação de pedido
- Cadastro de cliente
- Persistência auxiliar no Redis
- Rate Limit com retorno HTTP 429
- Métricas do Gateway
- Tratamento de falhas dos serviços

A disponibilidade das pizzas é determinada exclusivamente pelo serviço de **Cardápio**. O API Gateway não altera esse estado para produzir artificialmente um cenário de sucesso.

Tecnologias

- n8n Cloud
- APIs REST
- HTTP / JSON
- Redis
- Postman
- GitHub

Documentação

A documentação técnica completa do projeto está disponível na pasta:

```text
/docs
```

Ela contém detalhes sobre contratos, arquitetura, endpoints, integrações, testes, códigos HTTP e limitações encontradas durante o desenvolvimento.

Observações

Este projeto foi desenvolvido para fins acadêmicos.

Alguns serviços do ecossistema PZaaS são mantidos por outras equipes. Dessa forma, determinadas funcionalidades dependem da disponibilidade e do contrato fornecido por esses serviços externos.

---

**PZaaS — Pizza as a Service**  
**Serviço 01 — API Gateway**
