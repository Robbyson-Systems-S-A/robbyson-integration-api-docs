Workflow de integração de dados
-------------------------------

Esta seção descreve o fluxo de **integração de dados de operação**
(``/collaborators``, ``/indicators``, ``/attributes``, ``/hierarchies``,
``/results``) — autenticado por **Token de cliente** e organizado em
*transactions*.

O grupo **AI Agent Runtime** tem fluxo próprio, baseado em eventos via
webhook + resposta REST — veja :doc:`ai-agent-runtime`.

Por que transactions?
~~~~~~~~~~~~~~~~~~~~~

As informações trafegadas pela API são complexas, em geral dependentes
entre si e enviadas em pacotes grandes. Para simplificar essa
complexidade, usamos o conceito de ``transactions`` — separadores de
blocos a serem processados pelo integrador.

O ciclo é composto de quatro fases:

1. **Declaração** da transação (abertura)
2. **Envio** dos pacotes de dados nos recursos desejados
3. **Commit** da transação
4. **Acompanhamento** do processamento via consulta de status

Declaração da transação
^^^^^^^^^^^^^^^^^^^^^^^

O processo começa com um ``POST`` ao endpoint ``/transactions/`` para
abrir a transação. A resposta contém um campo ``_id`` — este é o
identificador da *transaction* e deverá ser enviado, via *query
string*, em **todas** as requisições subsequentes do pacote.

.. code:: bash

   $ curl \
       -X POST "https://integration-api.robbyson.com/v1/transactions/" \
       -H "token: ubogfszb2y9iti8njcmar4e39cg73m"

   $ {"data":{"_id":"8201903116563853","apiIntegrationScope":[],"status":1,"descStatus":"Created","date":"2019-01-31T16:56:03.824Z"},"statusLog":[{"type":3}]}

O campo ``descStatus`` indica o estado atual da transação. Ela inicia
em ``Created`` (1) — disponível para receber pacotes de dados.

Envio dos pacotes de dados
^^^^^^^^^^^^^^^^^^^^^^^^^^

Com a transação aberta, envie os dados nos recursos correspondentes
(``POST`` para criar, ``PUT`` para atualizar). Todas as requisições
exigem:

* O ``transaction_id`` via *query string*.
* O ``token`` do cliente via header.

Os cinco recursos seguem a **mesma anatomia de pacote** (descritos em
:doc:`recursos`):

* É possível enviar um pacote com múltiplos itens em uma mesma requisição.
* Recomenda-se pacotes com diversos dados, mas **nunca em quantidade
  maior que 500 itens**.
* Em caso de sucesso, a API retorna o pacote persistido.
* Em caso de erro, o retorno detalha o campo problemático e o motivo.

.. code:: bash

   # envio bem-sucedido de um pacote de colaboradores
   $ curl \
     -X POST "https://integration-api.robbyson.com/v1/collaborators/?transaction_id=8201903116563853" \
     -H "accept: application/json" \
     -H "token: ubogfszb2y9iti8njcmar4e39cg73m" \
     -H "content-type: application/json" \
     -d '[
       {
         "name": "Alan Turing",
         "identification": "1234",
         "contractorControlId": "1234",
         "genre": "M",
         "active": false,
         "birthDate": "1912-06-23"
       },
       {
         "name": "Grace Hopper",
         "identification": "5678",
         "contractorControlId": "5678",
         "genre": "M",
         "active": true,
         "birthDate": "1906-12-09"
       }
     ]'

   $ [{"__v":0,"name":"Alan Turing","identification":"1234",...,"stackId":"8201903116563853","contractorId":"8"},
      {"__v":0,"name":"Grace Hopper","identification":"5678",...,"stackId":"8201903116563853","contractorId":"8"}]

.. code:: bash

   # atualização de um colaborador existente
   $ curl \
     -X PUT "https://integration-api.robbyson.com/v1/collaborators/?transaction_id=8201903116563853" \
     -H "accept: application/json" \
     -H "token: ubogfszb2y9iti8njcmar4e39cg73m" \
     -H "content-type: application/json" \
     -d '[
       {
         "name": "Alan Turing",
         "identification": "1234",
         "contractorControlId": "1234",
         "genre": "M",
         "active": true,
         "birthDate": "1912-06-23"
       }
     ]'

   $ [{"__v":0,"name":"Alan Turing","identification":"1234","active":true,...}]

.. code:: bash

   # exemplo de erro de validação — falta o campo `name`, obrigatório para colaboradores
   $ curl \
     -X POST "https://integration-api.robbyson.com/v1/collaborators/?transaction_id=8201903116563853" \
     -H "accept: application/json" \
     -H "token: ubogfszb2y9iti8njcmar4e39cg73m" \
     -H "content-type: application/json" \
     -d '[
       {
         "identification": "1234",
         "contractorControlId": "1234",
         "genre": "M",
         "active": true,
         "birthDate": "1912-06-23"
       }
     ]'

   $ {"params":{"key":"name"},"code":302,"status":400,"name":"Error",
      "message":"Missing required property: name"}

Commit da transação
^^^^^^^^^^^^^^^^^^^

Após enviar todos os dados, sinalize **commit** com um ``POST`` ao
endpoint da própria transação. Isso marca o pacote como pronto para a
esteira de processamento da plataforma.

.. code:: bash

   $ curl \
       -X POST "https://integration-api.robbyson.com/v1/transactions/8201903116563853" \
       -H "token: ubogfszb2y9iti8njcmar4e39cg73m"

   $ {"data":{"n":1,"nModified":1,"ok":1},"statusLog":[{"type":3}]}

Opcionalmente, o commit aceita ``startDate`` e ``endDate`` via *query
string* para informar o intervalo de dados a serem processados —
útil quando o pacote enviado contém dados de várias datas mas você
quer restringir o processamento.

.. code:: bash

   $ curl \
       -X POST "https://integration-api.robbyson.com/v1/transactions/8201903116563853?startDate=2000-01-01&endDate=2000-01-20" \
       -H "token: ubogfszb2y9iti8njcmar4e39cg73m"

   $ {"data":{"n":1,"nModified":1,"ok":1},"statusLog":[{"type":3}]}

Regras de empacotamento
^^^^^^^^^^^^^^^^^^^^^^^

Em geral não há limite no que pode ser enviado em uma transação — você
pode embarcar múltiplas requisições e dados de diferentes recursos no
mesmo pacote. Duas exceções importantes:

* **Hierarquias e atributos devem ser enviados na mesma transação.** A
  validação interna depende dos dois conjuntos coexistirem.
* Recomenda-se que **dados de contexto único** (ex: importação inicial
  de uma empresa) viajem na mesma transação para coerência referencial.

Esteira de processamento
^^^^^^^^^^^^^^^^^^^^^^^^

Após o commit, a transação entra na esteira de processamento da
plataforma — uma sequência de etapas automáticas de validação,
consolidação e escrita no banco Robbyson. O processo é assíncrono e
tipicamente leva alguns minutos, dependendo do volume de dados.

Você pode acompanhar o progresso consultando o status da transação a
qualquer momento:

.. code:: bash

   $ curl "https://integration-api.robbyson.com/v1/transactions/8201903116563853" \
       -H "token: ubogfszb2y9iti8njcmar4e39cg73m"

   $ {"data":{"_id":"8201903116563853","apiIntegrationScope":[],"status":1,"descStatus":"Created","date":"2019-01-31T18:20:37.402Z"},"statusLog":[{"type":3}]}

A resposta carrega dois campos relevantes para o acompanhamento:

* ``status`` — código numérico do estado atual.
* ``descStatus`` — descrição literal do estado.

Estados possíveis:

+--------+---------------------+--------------------------------------------------------+
| Código | Nome                | Descrição                                              |
+========+=====================+========================================================+
| 1      | Created             | Transação criada. Disponível para envio de pacotes.    |
+--------+---------------------+--------------------------------------------------------+
| 2      | Importing           | Commit recebido. Esteira de processamento iniciada.    |
+--------+---------------------+--------------------------------------------------------+
| 3      | Imported            | Dados preparados para validação.                       |
+--------+---------------------+--------------------------------------------------------+
| 4      | Validating          | Validação de dados em curso.                           |
+--------+---------------------+--------------------------------------------------------+
| 5      | Validated           | Validação finalizada.                                  |
+--------+---------------------+--------------------------------------------------------+
| 6      | Consolidating       | Escrita no banco Robbyson iniciada.                    |
+--------+---------------------+--------------------------------------------------------+
| 7      | Done                | Processamento concluído com sucesso.                   |
+--------+---------------------+--------------------------------------------------------+
| 8      | Invalidated         | Erros impedem a integração — todo o pacote dispensado. |
+--------+---------------------+--------------------------------------------------------+
| 9      | ImportedWithErrors  | Validação acusou erros que não impedem a integração.   |
+--------+---------------------+--------------------------------------------------------+
| 10     | DoneWithErrors      | Concluído. Parte do pacote integrada, parte invalidada.|
+--------+---------------------+--------------------------------------------------------+
| 11     | ImportError         | Erro interno do algoritmo de integração.               |
+--------+---------------------+--------------------------------------------------------+

A transação está **finalizada** quando atinge um destes estados:

* ``Done`` (7) — todos os dados integrados com sucesso.
* ``DoneWithErrors`` (10) — parte dos dados integrada; demais
  invalidados com detalhe na resposta do *stack*.
* ``Invalidated`` (8) — pacote inteiro rejeitado.

Nos casos de invalidação parcial ou total, a resposta detalha quais
itens falharam e o motivo — corrija e reenvie em uma nova transação.
