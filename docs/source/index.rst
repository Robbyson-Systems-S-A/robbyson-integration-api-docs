API de Integração Robbyson
==========================

.. image:: logo.svg
    :width: 300px
    :alt: Robbyson logo
    :align: center

A **API de Integração Robbyson** é a superfície pública para sistemas externos
trocarem dados com a plataforma. Hoje cobre dois grupos de recursos:

* **Integração de dados de operação** — colaboradores, hierarquias, atributos,
  indicadores e resultados. Fluxo orientado a *transactions* (carga em lote,
  validação, esteira de processamento assíncrona).
* **AI Agent Runtime** — endpoints REST consumidos por *runtimes* externos de
  assistentes de IA (ex: n8n, Zapier, funções serverless de parceiros) que
  conversam com usuários da plataforma como participantes do chat. Fluxo
  orientado a *eventos* (webhook → REST de resposta).

Os dois grupos compartilham o mesmo gateway público e as mesmas convenções de
HTTP/REST, mas têm **modelos de autenticação distintos** (Token de cliente
*vs* Bearer JWT por agente) — escolha a seção apropriada conforme o seu caso
de uso.

Esta documentação assume conhecimentos básicos de requisições HTTP e
manipulação de dados `JSON <https://www.json.org/>`_.


Sumário
=======

.. toctree::
   :maxdepth: 2

   enderecos-forma-consulta.rst
   autenticacao.md
   recursos.md
   workflow.rst
   ai-agent-runtime.md
   anexos.md
