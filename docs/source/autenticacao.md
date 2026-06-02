# Autenticação

A API de Integração Robbyson opera com **duas modalidades de autenticação**, escolhidas conforme o grupo de recursos sendo consumido. Cada modalidade tem seu próprio tipo de credencial, header e ciclo de vida.

## Modalidade 1 — Token de cliente (recursos de integração de dados)

Aplicável a: `/collaborators`, `/indicators`, `/attributes`, `/hierarchies`, `/results`, `/transactions`.

A credencial é um **Token** (chave única por cliente integrador) fornecido pela equipe de implantação Robbyson. É enviado no header `token` em **todas** as requisições.

```http
token: ubogfszb2y9iti8njcmar4e39cg73m
```

O token é gerado automaticamente e pode ser renovado em caso de perda — basta solicitar à equipe de suporte. Não há revogação automática, nem TTL: o token é válido até ser explicitamente substituído.

> **Importante:** este token é a chave de acesso para consulta e atualização dos dados da sua empresa em nossa base. Mantenha-o em local seguro (gerenciador de credenciais, secret manager) e informe-o apenas aos responsáveis técnicos pela integração.

## Modalidade 2 — Bearer JWT por agente (AI Agent Runtime)

Aplicável a: `/agent-runtime/*`.

A credencial é um **JWT HS256** emitido pelo administrador da plataforma para um agente IA específico. É enviado no header `Authorization` no padrão HTTP Bearer.

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsImtpZCI6InYxIn0.eyJjb250cmFjdG9yX2lkIjoxLCJhZ2VudF9jb2RlIjoicm9iYnlzb24iLCJpbnRlZ3JhdGlvbl90b2tlbl9pZCI6IjQ5YjI1NTE1LTVjZjMtNDk0MC04MWRiLWQwZDhkYjk2YWUyZSIsInNjb3BlcyI6WyJhZ2VudHM6cmVhZCIsImNvbnZlcnNhdGlvbnM6cmVhZCIsImNvbnZlcnNhdGlvbnM6d3JpdGUiLCJtZXNzYWdlczp3cml0ZSJdLCJpYXQiOjE3Nzk0MDIwNzYsImV4cCI6MTc4MDYwOTI3Nn0.signature
```

### Como obter

1. Solicite ao administrador Robbyson do contratante que **emita um token** para o agente IA correspondente, na tela de edição do agente no painel administrativo.
2. Você receberá o JWT completo **uma única vez** no momento da emissão — guarde-o em local seguro (credentials store do seu runtime).
3. Configure o seu runtime (n8n Webhook node com Header Auth, middleware do seu Express, gateway de API, etc.) para enviar o header `Authorization: Bearer <jwt>` em todas as chamadas para `/agent-runtime/*`.

### Claims do JWT

O JWT carrega os seguintes claims (informativos — você não precisa decodificar para usar):

| Claim                  | Significado                                                                |
|------------------------|----------------------------------------------------------------------------|
| `contractor_id`        | Identificador do contratante na plataforma                                 |
| `agent_code`           | Código curto do agente                                                     |
| `integration_token_id` | UUID único do token emitido (usado para revogação)                         |
| `scopes`               | Lista de escopos autorizados (ex: `messages:write`, `conversations:read`)  |
| `iat` / `exp`          | Issued at / Expires at (padrão JWT)                                        |

### Ciclo de vida

* **TTL default:** 14 dias. O administrador pode pedir até 365 dias na emissão.
* **Revogação:** o administrador pode revogar o token a qualquer momento pelo painel — a API rejeita o token imediatamente, sem esperar o `exp` natural.
* **Sem renovação automática:** quando o JWT expira, o runtime precisa pedir um novo ao administrador.

### Escopos

Cada endpoint da API exige um escopo específico no JWT. Tokens emitidos pelo painel administrativo recebem o conjunto completo por default; escopos reduzidos serão suportados em versões futuras.

| Escopo                | Endpoints                                                      |
|-----------------------|----------------------------------------------------------------|
| `agents:read`         | `GET /agent-runtime/agents/me`                                 |
| `conversations:read`  | `GET /agent-runtime/conversations/:id/messages`                |
| `conversations:write` | `POST /agent-runtime/conversations/:id/typing`                 |
| `messages:write`      | `POST /agent-runtime/messages`                                 |

Tokens sem o escopo requerido recebem `403 insufficient_scope` na chamada.

> **Importante:** trate o JWT como credencial sensível — não comite em repositórios, não exponha em logs, não compartilhe entre runtimes. Vazamento permite que um terceiro envie mensagens em nome do agente até a revogação ou expiração.

## Resumo das diferenças

|                  | Token de cliente        | Bearer JWT por agente              |
|------------------|-------------------------|------------------------------------|
| Grupo de recursos| Integração de dados     | AI Agent Runtime                   |
| Header           | `token: <valor>`        | `Authorization: Bearer <jwt>`      |
| Escopo da credencial | Por cliente (uma para tudo) | Por agente (uma por agente IA) |
| TTL              | Indefinido              | 14 dias default (até 365)          |
| Revogação        | Manual (suporte)        | Imediata via painel administrativo |
| Granularidade    | Tudo ou nada            | Por escopo de endpoint             |
