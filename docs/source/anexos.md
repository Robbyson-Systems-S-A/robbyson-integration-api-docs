# Anexos

## Documentação técnica interativa (Swagger)

Cada serviço da API expõe um Swagger UI para teste e depuração interativa:

| Recurso          | Swagger                                                                          |
|------------------|----------------------------------------------------------------------------------|
| Colaboradores    | `https://integration-api.robbyson.com/v1/collaborators/docs/endpoints/`          |
| Indicadores      | `https://integration-api.robbyson.com/v1/indicators/docs/endpoints/`             |
| Atributos        | `https://integration-api.robbyson.com/v1/attributes/docs/endpoints/`             |
| Hierarquias      | `https://integration-api.robbyson.com/v1/hierarchies/docs/endpoints/`            |
| Resultados       | `https://integration-api.robbyson.com/v1/results/docs/endpoints/`                |
| Transactions     | `https://integration-api.robbyson.com/v1/transactions/docs/endpoints/`           |
| AI Agent Runtime | `https://integration-api.robbyson.com/v1/agent-runtime/docs/endpoints/`          |

Os Swaggers de sandbox seguem o mesmo padrão, trocando `integration-api` por `integration-sandbox`.

## Tipos comuns

### Formato de datas

Todas as datas seguem ISO 8601:

* Data simples: `YYYY-MM-DD` (ex: `2026-05-22`)
* Timestamp UTC: `YYYY-MM-DDTHH:mm:ss.sssZ` (ex: `2026-05-22T20:35:47.535Z`)

### Formato de respostas de erro

**Integração de dados** (formato JSON Schema validation):

```json
{
    "params": { "key": "name" },
    "code": 302,
    "status": 400,
    "name": "Error",
    "message": "Missing required property: name"
}
```

**AI Agent Runtime** (formato OAuth-style):

```json
{
    "error": "invalid_token",
    "error_description": "Token has been revoked"
}
```

## Glossário

| Termo                 | Significado                                                              |
|-----------------------|--------------------------------------------------------------------------|
| **Token**             | Credencial de cliente integrador (formato opaco, único por cliente)      |
| **JWT**               | JSON Web Token usado pelo AI Agent Runtime (formato Bearer HS256)        |
| **Transaction**       | Bloco de envio em lote do grupo de integração de dados                   |
| **Stack**             | Sinônimo interno de transaction (aparece em respostas de erro/audit)     |
| **Agente**            | Assistente de IA cadastrado na plataforma, com identidade e visibilidade |
| **Runtime**           | Sistema externo que processa eventos do agente e responde via REST       |
| **Webhook**           | Endpoint HTTPS do seu runtime onde o Robbyson entrega eventos do agente  |
| **Idempotency-Key**   | Header HTTP que evita duplicação em retentativas do POST /messages       |
| **Scope**             | Permissão fina por endpoint no AI Agent Runtime (ex: `messages:write`)   |

## Suporte

* **Solicitação de credenciais** (Token de cliente, JWT de agente): equipe de implantação Robbyson.
* **Acompanhamento de transações com falha**: o status e a descrição dos erros estão na resposta do `GET /transactions/:id`. Para análise mais profunda, contate o suporte com o `transaction_id`.
* **Problemas de integração com o AI Agent Runtime**: contate o administrador da plataforma Robbyson do seu contratante para validar configuração do agente, escopo do token e cadastro do webhook.
