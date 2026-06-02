Endereços e formas de consulta
------------------------------

A API de Integração Robbyson é uma API
`REST <https://en.wikipedia.org/wiki/Representational_state_transfer>`_
sobre HTTPS. Toda comunicação é feita por requisições HTTP padrão — não
depende da linguagem do cliente — e utiliza os
`verbos HTTP <https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods>`_
para denotar a ação executada.

Verbos suportados:

+----------+----------------------------------------------------------+
| Método   | Descrição                                                |
+==========+==========================================================+
| ``GET``  | Consulta de dados. Não altera estado.                    |
+----------+----------------------------------------------------------+
| ``POST`` | Criação de um novo recurso ou disparo de uma ação        |
|          | (envio de pacote, abertura/commit de *transaction*,      |
|          | entrega de mensagem do agente).                          |
+----------+----------------------------------------------------------+
| ``PUT``  | Atualização de dados existentes.                         |
+----------+----------------------------------------------------------+
| ``PATCH``| Atualização parcial de dados existentes.                 |
+----------+----------------------------------------------------------+
|``DELETE``| Remoção de dados existentes.                             |
+----------+----------------------------------------------------------+

Ambientes
~~~~~~~~~

A URL final é composta pelo endereço do ambiente + caminho do recurso.

+----------+-------------------------------------------------+
| Ambiente | Endereço                                        |
+==========+=================================================+
| Produção | ``https://integration-api.robbyson.com/v1``     |
+----------+-------------------------------------------------+
| Sandbox  | ``https://integration-sandbox.robbyson.com/v1`` |
+----------+-------------------------------------------------+

.. note::

    A URL do sandbox espelha estruturalmente a de produção (mesmos endpoints,
    mesmos contratos), mas os dados são isolados e podem ser apagados a
    qualquer momento. Use-a para validação inicial de integrações antes de
    apontar para produção.

Grupos de recursos
~~~~~~~~~~~~~~~~~~

**Integração de dados de operação** (envio em lote, autenticação por *Token*):

+--------------------+--------------------------------------------------+
| Recurso            | Caminho base                                     |
+====================+==================================================+
| Colaboradores      | ``/collaborators``                               |
+--------------------+--------------------------------------------------+
| Indicadores        | ``/indicators``                                  |
+--------------------+--------------------------------------------------+
| Atributos          | ``/attributes``                                  |
+--------------------+--------------------------------------------------+
| Hierarquias        | ``/hierarchies``                                 |
+--------------------+--------------------------------------------------+
| Resultados         | ``/results``                                     |
+--------------------+--------------------------------------------------+
| Transactions       | ``/transactions``                                |
+--------------------+--------------------------------------------------+

**AI Agent Runtime** (resposta de assistentes de IA, autenticação por
*Bearer JWT*):

+--------------------+---------------------------------------------------------+
| Recurso            | Caminho base                                            |
+====================+=========================================================+
| Agente (config)    | ``/agent-runtime/agents/me``                            |
+--------------------+---------------------------------------------------------+
| Mensagens          | ``/agent-runtime/messages``                             |
+--------------------+---------------------------------------------------------+
| Conversas          | ``/agent-runtime/conversations/:id/messages``           |
|                    |                                                         |
|                    | ``/agent-runtime/conversations/:id/typing``             |
+--------------------+---------------------------------------------------------+

Os caminhos dos dois grupos podem ser consumidos pelo mesmo cliente, mas
**utilizam credenciais e fluxos distintos** — veja :doc:`autenticacao`,
:doc:`workflow` e :doc:`ai-agent-runtime`.

Convenções gerais
~~~~~~~~~~~~~~~~~

* Content-Type esperado: ``application/json``.
* Datas em formato ISO 8601 (``YYYY-MM-DD`` para datas simples,
  ``YYYY-MM-DDTHH:mm:ssZ`` para timestamps).
* Códigos de resposta seguem o padrão HTTP:

  * ``2xx`` — sucesso
  * ``4xx`` — erro do cliente (validação, autenticação, autorização)
  * ``5xx`` — erro do servidor

* Em erros de validação, o corpo da resposta detalha o campo e o motivo.
