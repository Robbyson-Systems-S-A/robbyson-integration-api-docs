Workflow
~~~~~~~~

As informações trafegadas pela API são complexas. Em geral, dependentes
entre elas e constituídas de pacotes enormes de dados. Para simplificar
esta complexidade, utilizamos o conceito de ``transactions``.
Transactions são separadores de blocos a serem processados pelo
integrador.

Declaração da transaction
^^^^^^^^^^^^^^^^^^^^^^^^^

O processo de transferência de dados deve ser iniciado com um ``POST``
de declaração do início de uma transação. Este post retorna como
resultado um objeto contendo um ``"_id"``. Este ``_id`` é a
identificação da transaction, e deverá ser informado em todas as
requisições do pacote.

Segue abaixo um exemplo de criação de uma nova transação: note a
presença dos campos de \_id, que como foi dito anteriormente, deverá ser
trafegado para todas as próximas requisições, e o campo ``descStatus``,
que demonstra o status da transação. O status dela será abordado logo em
seguida.

.. code:: bash

   $ curl \
       -X POST "https://integration-api.robbyson.com/v1/transactions/" \
       -H "token: ubogfszb2y9iti8njcmar4e39cg73m"

   $ {"data":{"_id":"8201903116563853","apiIntegrationScope":[],"status":1,"descStatus":"Created","date":"2019-01-31T16:56:03.824Z"},"statusLog":[{"type":3}]}


Envio de pacotes de dados
^^^^^^^^^^^^^^^^^^^^^^^^^

Após a criação da transação, o próximo passo é o envio dos dados dos
recursos diversos para a API. Este envio deve ser feito através das
rotas de escrita e atualização de informações sob os mesmos recursos.

Abaixo, pode-ser ver o envio de itens para o recurso de
``collaborators``. Todos os cinco recursos possuem a mesma anatomia de
pacote enviado, variando entre eles, obviamente, os dados que cada
recurso requer (estes serão descritos em outro tópico):

-  É possível enviar um pacote de dados com diversos itens em uma mesma
   requisição. Recomenda-se o envio de pacotes com diversos dados, mas
   nunca em quantidade maior que 500;
-  Deve-se sempre enviar o parâmetro ``transaction_id`` via `query
   string`_.
-  É obrigatório também o envio, via header, do token do cliente.
-  Em caso de sucesso, a API deverá retornar o pacote de dados
   persistido no banco da API. Em caso de erro, este será explicitado
   também no retorno da API.

.. _query string: https://en.wikipedia.org/wiki/Query_string

.. code:: bash

   # envio de um pacote de dados de colaboradores concluído com sucesso.
   $ curl \
     -X POST "https://integration-api.robbyson.com/v1/collaborators/?transaction_id=8201903116563853" \
     -H  "accept: application/json" \
     -H "token: ubogfszb2y9iti8njcmar4e39cg73m" \
     -H  "content-type: application/json" \
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
   $ [{"__v":0,"name":"Alan Turing","identification":"1234","contractorControlId":"1234","genre":"M","active":false,"birthDate":"1912-06-23","stackId":"8201903116563853","contractorId":"8"},{"__v":0,"name":"Grace Hopper","identification":"5678","contractorControlId":"5678","genre":"M","active":true,"birthDate":"1906-12-09","stackId":"8201903116563853","contractorId":"8"}]

.. code:: bash

   # envio de um pacote de atualização de colaboradores.
   $ curl \
     -X PUT "https://integration-api.robbyson.com/v1/collaborators/?transaction_id=8201903116563853" \
     -H  "accept: application/json" \
     -H "token: ubogfszb2y9iti8njcmar4e39cg73m" \
     -H  "content-type: application/json" \
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
   $ [{"__v":0,"name":"Alan Turing","identification":"1234","contractorControlId":"1234","genre":"M","active":true,"birthDate":"1912-06-23","stackId":"8201903116563853","contractorId":"8"}]

.. code:: bash

   # Envio de pacote de dados de colaboradores, sem o campo `name`, que é obrigatório para este recurso.
   $ curl \
     -X POST "https://integration-api.robbyson.com/v1/collaborators/?transaction_id=8201903116563853" \
     -H  "accept: application/json" \
     -H "token: ubogfszb2y9iti8njcmar4e39cg73m" \
     -H  "content-type: application/json" \
     -d '[
       {
         "identification": "1234",
         "contractorControlId": "1234",
         "genre": "M",
         "active": true,
         "birthDate": "1912-06-23"
       }
     ]'
   $ {"params":{"key":"name"},"code":302,"dataPath":"","schemaPath":"/required/0","subErrors":null,"status":400,"name":"Error","message":"The \"collaborators\" body parameter is invalid ([{\"identification\":\"1234\",\"contractorControlId\":\"1234\",\"genre\":\"M\",\"active\":true,\"birthDate\":\"1912-06-23\"}]) \nUnable to parse array item at index 0 ({\"identification\":\"1234\",\"contractorControlId\":\"1234\",\"genre\":\"M\",\"active\":true,\"birthDate\":\"1912-06-23\"}). \nJSON Schema validation error. \nData path: \"undefined[0].\" \nSchema path: \"/required/0\" \nMissing required property: name"}

Commit da transaction
^^^^^^^^^^^^^^^^^^^^^

Via de regra, não existe limite ao pacote enviado para cada transaction:
além de poder embarcar qualquer quantidade de requisições dentro desta
transaction, também é possível enviar nela dados de um ou mais dos
recursos. A única exceção para esta regra é que os dados de hierarquias
e atributos devem ser enviados **obrigatoriamente** na mesma transação.
Apesar de maleável, recomenda-se que todos os dados de determinado
contexto sejam enviados na mesma transação.

Finalmente, após enviar todos os dados da transação, é necessário enviar
um sinal de ``commit``. Este sinal é necessário para informar ao sistema
de integração que o pacote está pronto para ser processado.

O commit da transação é outra requisição com o verbo ``POST``,
informando o id dela.

.. code:: bash

   $ curl \
       -X POST "https://integration-api.robbyson.com/v1/transactions/8201903116563853" \
       -H "token: ubogfszb2y9iti8njcmar4e39cg73m"

   $ {"data":{"n":1,"nModified":1,"ok":1},"statusLog":[{"type":3}]}

Opcionalmente, é possível enviar as opções de ``startDate`` e ``endDate`` na query do commit. Estes campos vão informar o intervalo de dados a serem processados pelo Robbyson, apesar do pacote enviado possuir dados de datas diferentes. Estes campos são úteis para os casos onde deseja-se processar um pacote menor de dados, mesmo que o script produtor de dados seja o mesmo que produza um pacote maior.

.. code:: bash

   $ curl \
       -X POST "https://integration-api.robbyson.com/v1/transactions/8201903116563853?startDate=2000-01-01&endDate=2000-01-20" \
       -H "token: ubogfszb2y9iti8njcmar4e39cg73m"
 
   $ {"data":{"n":1,"nModified":1,"ok":1},"statusLog":[{"type":3}]}

Esteira de processamento
^^^^^^^^^^^^^^^^^^^^^^^^

O último passo da API que requer intervenção do cliente é o commit da
transação. Após o commit, entra em ação os mecanismos de integração de
dados do Robbyson.

Este mecanismo age de forma totalmente automática, e passa por processos
de validação e integração dos dados das transações ao banco do Robbyson.

Este processo em geral leva alguns minutos para ser finalizado,
dependendo sempre do volume de dados a serem processados. É possível
acompanhar o progresso da integração através da consulta ao status da
transação.

.. code:: bash

   # consulta ao status da transação
   $ curl "https://integration-api.robbyson.com/v1/transactions/8201903116563853" \
       -H "token: ubogfszb2y9iti8njcmar4e39cg73m"

   $ {"data":{"_id":"82019031182037426","apiIntegrationScope":[],"status":1,"descStatus":"Created","date":"2019-01-31T18:20:37.402Z"},"statusLog":[{"type":3}]}

Esta consulta pode ser executada a qualquer momento, desde a criação até
após a conclusão do processamento da transaction. Ela retorna dois
campos importantes para o acompanhamento da integração: o ``status``,
campo nunérico com o valor de controle do andamento do processamento e o
``descStatus``, campo com descrição literal da transaction.

Os passos de integração do sistema Robbyson atualizam continuamente o
status desta transação. Pode-se considerar que a transação foi
finalizada assim que esta ``transaction`` estiver nos status ``done``
(todos os dados foram importados com sucesso e estão disponíveis no
Robbyson), ``done with errors`` (alguns erros de validação foram
encontrados. Dados corretos estão disponíveis no Robbyson) ou
``invalidated`` (todo o pacote foi invalidado). Nos casos de invalidação
de dados, estas informações estarão descritas no retorno do stack e
podem ser tratados em uma nova transação.

+-----------------------+-----------------------+-----------------------+
| Código                | Nome                  | Descrição             |
+=======================+=======================+=======================+
| 1                     | Created               | transaction criada.   |
|                       |                       | Disponível para envio |
|                       |                       | de pacotes            |
+-----------------------+-----------------------+-----------------------+
| 2                     | Importing             | Processo de envio     |
|                       |                       | finalizado. A esteira |
|                       |                       | de processamento      |
|                       |                       | iniciou a integração  |
+-----------------------+-----------------------+-----------------------+
| 3                     | Imported              | Dados preparados para |
|                       |                       | validação             |
+-----------------------+-----------------------+-----------------------+
| 4                     | Validating            | Validação de dados    |
|                       |                       | iniciada              |
+-----------------------+-----------------------+-----------------------+
| 5                     | Validated             | Validação finalizada  |
+-----------------------+-----------------------+-----------------------+
| 6                     | Consolidating         | Processo de escrita   |
|                       |                       | no banco Robbyson     |
|                       |                       | iniciado              |
+-----------------------+-----------------------+-----------------------+
| 7                     | Done                  | Processo de escrita   |
|                       |                       | no banco Robbyson     |
|                       |                       | finalizado com        |
|                       |                       | sucesso               |
+-----------------------+-----------------------+-----------------------+
| 8                     | Invalidated           | Algum item dos dados  |
|                       |                       | enviados à integração |
|                       |                       | possui erros que      |
|                       |                       | impossibilitam a      |
|                       |                       | utilização deste      |
|                       |                       | pacote. Todo ele será |
|                       |                       | dispensado            |
+-----------------------+-----------------------+-----------------------+
| 9                     | ImportedWithErrors    | Processo de           |
|                       |                       | importação e          |
|                       |                       | validação acusaram    |
|                       |                       | erros que não impedem |
|                       |                       | o pacote de ser       |
|                       |                       | integrado             |
+-----------------------+-----------------------+-----------------------+
| 10                    | DoneWithErrors        | Processamento         |
|                       |                       | finalizado. Parte do  |
|                       |                       | pacote foi integrada  |
|                       |                       | com sucesso. Outra    |
|                       |                       | parte foi invalidada  |
+-----------------------+-----------------------+-----------------------+
| 11                    | ImportError           | Erro interno no       |
|                       |                       | algoritmo de          |
|                       |                       | integração do         |
|                       |                       | Robbyson.             |
+-----------------------+-----------------------+-----------------------+