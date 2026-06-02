## AI Agent Runtime

A **AI Agent Runtime** é a superfície REST da API de Integração consumida por *runtimes* externos de assistentes de IA que conversam com usuários da plataforma como participantes do chat. Pense em n8n, Zapier, AWS Lambda, ou qualquer orquestrador HTTP — todos consomem o mesmo contrato.

Diferente da [integração de dados](recursos.html), aqui o fluxo é **orientado a eventos**: a plataforma entrega o evento ao runtime via webhook, o runtime processa (tipicamente consultando o LLM da preferência dele), e responde via REST.

### Visão geral do fluxo

```
                    ┌────────────────────────────────────────────────┐
                    │  PLATAFORMA ROBBYSON                            │
                    └────────────────────────────────────────────────┘
                              │
        usuário envia         │
        mensagem pro agente   │
                              ▼
                    [evento user_message gerado]
                              │
                              │ POST webhook do cliente (com header de auth do cliente)
                              ▼
                    ┌────────────────────────────────────────────────┐
                    │  SEU RUNTIME (n8n, Lambda, etc.)                │
                    │                                                 │
                    │  1. valida o header de auth na entrada          │
                    │  2. (opcional) GET /agent-runtime/agents/me     │
                    │  3. (opcional) GET /conversations/:id/messages  │
                    │  4. monta prompt e consulta o LLM               │
                    │  5. POST /agent-runtime/messages                │
                    └────────────────────────────────────────────────┘
                              │
                              │ resposta entregue ao chat do usuário
                              ▼
                         [usuário vê a resposta]
```

Você cadastra **uma vez** o endpoint do seu runtime no painel administrativo Robbyson (junto com a credencial que ele espera receber). A partir daí, todo evento do agente é entregue automaticamente.

### Cadastro do webhook (uma vez, no admin)

Antes do primeiro evento chegar, o administrador Robbyson cadastra no painel do agente IA:

* **URL do webhook** — endpoint HTTPS exposto pelo seu runtime. URLs HTTP são aceitas apenas para destinos internos do cluster Robbyson (uso interno); endpoints externos precisam ser HTTPS.
* **Eventos assinados** — tipicamente `user_message` (mensagem do usuário pro agente). Outros eventos disponíveis: `conversation_opened`, `agent_updated`, `agent_disabled`, `session_started`.
* **Autenticação (opcional)** — você fornece o `header name` e `header value` que o Robbyson deve enviar em toda *delivery*. Pode ser `Authorization: Bearer <seu-token>`, `X-API-Key: <chave>`, ou qualquer combinação que seu runtime saiba validar. Webhooks também podem ser cadastrados sem autenticação, para endpoints internos protegidos por outro meio (rede privada, IP allowlist, mTLS no proxy).

### Recebendo eventos (delivery do webhook)

A plataforma faz `POST` para a URL cadastrada com **JSON envelope** e headers de auditoria:

**Headers:**

```http
Content-Type: application/json
User-Agent: Robbyson-Webhook/1.0
X-Robbyson-Event: user_message
X-Robbyson-Delivery-Id: <uuid v4>
X-Robbyson-Agent-Code: rh_assistant
X-Robbyson-Contractor-Id: 1
<seu-header-de-auth>: <seu-valor>
```

**Body:**

```json
{
    "event_id": "6a10be2216180d0019c5a30e",
    "event_type": "user_message",
    "contractor_id": 1,
    "agent_code": "rh_assistant",
    "occurred_at": "2026-05-22T20:35:47.535Z",
    "delivery_attempt": 1,
    "data": {
        "message_id": "6a10be2216180d0019c5a30e",
        "conversation_id": "6a0f8798bd61d60031f874aa",
        "target": "ai_agent_rh_assistant",
        "sender": "09270854612",
        "content": "Olá, qual o saldo do meu banco de horas?",
        "sent_at": "2026-05-22T20:35:47.535Z",
        "idempotency_key": "6a10be2216180d0019c5a30e"
    }
}
```

O `X-Robbyson-Delivery-Id` é **único por tentativa** — use-o para deduplicar caso a sua aplicação queira garantia extra (a plataforma também faz retentativa em falhas; cada uma terá `delivery_id` novo, mas o `event_id` se repete).

**Resposta esperada do seu runtime:** qualquer `2xx` em até 10s. Qualquer outro status, timeout ou network error é tratado como falha → retentativa.

#### Política de retry e auto-disable

| Tentativa | Atraso após falha |
|-----------|-------------------|
| 1         | 30s               |
| 2         | 2min              |
| 3         | 10min             |
| 4         | 1h                |
| 5         | 6h                |
| 6         | (esgotado)        |

Após 5 falhas consecutivas, o webhook é **auto-desabilitado** e o administrador precisa reabilitar manualmente no painel. Defesa contra bombardeio de endpoint quebrado.

### Endpoints REST (resposta do runtime)

Todos os endpoints abaixo estão sob `https://integration-api.robbyson.com/v1/agent-runtime/` e exigem header `Authorization: Bearer <jwt>` (veja [Autenticação](autenticacao.html#modalidade-2-bearer-jwt-por-agente-ai-agent-runtime)).

#### `GET /agent-runtime/agents/me`

Retorna a configuração do agente identificado pelo JWT — útil pra o runtime carregar `system_prompt`, `metadata`, etc. no início de cada execução.

**Escopo necessário:** `agents:read`.

**Resposta 200:**

```json
{
    "data": {
        "_id": "6a0f7ca81ffa1e001f25216b",
        "code": "rh_assistant",
        "name": "Agente de RH",
        "system_prompt": "Você é um assistente de RH...",
        "visibilityScope": "users",
        "event_subscriptions": ["user_message", "conversation_opened"],
        "metadata": { }
    }
}
```

#### `GET /agent-runtime/conversations/:id/messages`

Histórico recente da conversa, **decifrado** (plaintext) para que o runtime monte contexto para o LLM. Aceita query strings opcionais `limit` (default 10, máx 100) e `before` (ID de mensagem para paginação).

**Escopo necessário:** `conversations:read`.

**Resposta 200:**

```json
{
    "data": [
        {
            "_id": "...",
            "message_id": "...",
            "sender": "09270854612",
            "receiver": "ai_agent_rh_assistant",
            "content": "Mensagem em plaintext",
            "receivedAt": "2026-05-22T20:35:47.535Z",
            "metadata": { }
        }
    ]
}
```

#### `POST /agent-runtime/conversations/:id/typing`

Liga ou desliga o indicador "digitando..." na conversa do usuário. Best-effort — não bloqueia o fluxo se falhar.

**Escopo necessário:** `conversations:write`.

**Body:**

```json
{ "is_typing": true }
```

**Resposta 204** (sem corpo).

#### `POST /agent-runtime/messages`

Entrega a resposta do agente para o usuário. **Idempotente por header `Idempotency-Key`** (recomendado usar o `message_id` do evento original) — janela de dedup de 24h. Retentativas com a mesma key retornam o resultado original com `status: "duplicate"`.

**Escopo necessário:** `messages:write`.

**Headers obrigatórios:**

```http
Authorization: Bearer <jwt>
Idempotency-Key: <message_id do evento original>
```

**Body:**

```json
{
    "target_identification": "09270854612",
    "content": "Olá Geldo, seu saldo de banco de horas é...",
    "conversation_id": "6a0f8798bd61d60031f874aa",
    "in_reply_to": "6a10be2216180d0019c5a30e",
    "metadata": {
        "source": "n8n-rh",
        "model": "gemini-2.5-flash",
        "finish_reason": "STOP"
    }
}
```

A mensagem do agente pode usar **Markdown leve** (negrito, listas, tabelas, blocos de código). A plataforma sanitiza o HTML resultante e renderiza no chat. Links externos são automaticamente abertos em nova aba; URLs com protocolos perigosos (`javascript:`, `data:`, etc.) são descartadas — não tente embutir esses esquemas, eles não chegarão ao usuário.

**Resposta 202** (mensagem aceita e enfileirada):

```json
{
    "data": {
        "message_id": "6a0f8798bd61d60031f874ad",
        "conversation_id": "6a0f8798bd61d60031f874aa",
        "status": "accepted"
    }
}
```

**Resposta 200** com `status: "duplicate"` em retentativas com o mesmo `Idempotency-Key`.

### Rate limit

A plataforma aplica rate limit em **duas camadas** sobre as suas chamadas:

* **Por token** — 60 requisições/60s por `integration_token_id`. Defesa contra runtime malfuncionando.
* **Por par (agente, destinatário)** — 10 mensagens/60s entre o mesmo agente e o mesmo usuário. Defesa contra spam.

Quando excedido, a API retorna `429 Too Many Requests`:

```json
{
    "message": "Rate limit excedido para este destinatario — aguarde alguns segundos antes de reenviar",
    "retry_after_ms": 60000
}
```

Trate como falha temporária e re-tente após o tempo indicado.

### Idempotência ponta a ponta

A plataforma garante que **uma mensagem só é entregue uma vez** ao usuário final, mesmo em cenários de retry da sua aplicação ou da rede:

1. **Recebimento do webhook** — o `X-Robbyson-Delivery-Id` é único por tentativa; o `event_id` se repete entre retentativas do mesmo evento. Use o `delivery_id` para garantir dedup no seu runtime.
2. **Envio de resposta** — `Idempotency-Key` no `POST /messages` garante que retentativas com a mesma key convergem para uma única mensagem persistida. Janela de dedup de 24h.

Recomendamos usar o `message_id` do evento original como `Idempotency-Key` — facilita o rastreio.

### Erros

A API retorna erros no formato OAuth-style:

```json
{
    "error": "invalid_token",
    "error_description": "Token has expired"
}
```

Códigos relevantes:

| HTTP | `error`                    | Significado                                                       |
|------|----------------------------|-------------------------------------------------------------------|
| 400  | `invalid_request`          | Falta de Authorization ou de header obrigatório                   |
| 400  | `validation_failed`        | Body inválido (veja `error_description` para detalhe do campo)    |
| 401  | `invalid_token`            | JWT expirado, malformado, revogado, ou assinatura inválida        |
| 403  | `insufficient_scope`       | Token sem o escopo requerido para o endpoint                      |
| 403  | (sem `error`)              | Contratante não tem AI Agents habilitado, ou agente não está ativo|
| 404  | `not_found`                | Conversa, agente ou recurso não encontrado                        |
| 429  | (sem `error`)              | Rate limit excedido (veja `retry_after_ms`)                       |
| 5xx  | `internal_error`           | Erro interno da plataforma (re-tente com backoff)                 |

### Boas práticas

* **Valide o header de auth na entrada do seu webhook** antes de processar — qualquer requisição sem o header esperado deve receber 401, mesmo que pareça vir da plataforma.
* **Use `Idempotency-Key = message_id`** no POST de resposta — simplifica investigação de incidente.
* **Implemente backoff exponencial** em respostas 5xx ou 429.
* **Não armazene o JWT em código-fonte** — use o credentials store do seu runtime (n8n credentials, AWS Secrets Manager, etc.).
* **Trate o `system_prompt` como configuração da plataforma**, não do seu runtime — busque via `GET /agents/me` e não hardcode no workflow.
* **Em caso de mensagem com conteúdo muito longo da parte do LLM**, prefira dividir em múltiplas mensagens em vez de truncar — cada mensagem é uma chamada separada ao `POST /messages` com `Idempotency-Key` distinto.

### Workflow de referência

A plataforma disponibiliza um **workflow de referência em n8n** ilustrando o caso de uso completo (receber evento → buscar histórico → chamar LLM → responder via API), incluindo as três credenciais necessárias e os 10 nós do pipeline. Solicite à equipe Robbyson o template `Agente - Robbyson Operacao.json` para importar no seu n8n.
