## Recursos da API

Entende-ser por recursos da API todos os tipos de dados que são trafegados por ela.

Conforme dito anteriormente, os recursos são acessados através de uma url, que será composta pelo endereço do serviço utilizado (produção ou sandbox) mais uma identificação do recurso consumido. Abaixo serão descritos cada um deles, com os detalhes para acesso.

### Colaboradores
**Identificação:** `/collaborators`

```js
// Descever aqui o recurso
```

### Indicadores
**Identificação:** `/indicators`

```js
// Descever aqui o recurso
```

### Atributos
**Identificação:** `/attributes`

```js
// Descever aqui o recurso
```

### Hierarquia
**Identificação:** `/hierarchies`

```js
// Descever aqui o recurso
```

### Resultados
**Identificação:** `/results`

```js
// Descever aqui o recurso
```

### Workflow

As informações trafegadas pela API são complexas. Em geral, dependentes entre elas e constituídas de pacotes enormes de dados. Para simplificar esta complexidade, utilizamos o conceito de `transactions`. Transactions são separadores de blocos a serem processados pelo integrador de forma atômica.

O processo de transferência de dados deve ser iniciado com um `POST` de declaração do início de uma transação. Este post retorna como resultado um objeto contendo um `"_id"`. Este `_id` é a identificação da transaction, e deverá ser informado em todas as requisições do pacote.

Segue abaixo um exemplo de criação de uma nova transação: note a presença dos campos de _id, que como foi dito anteriormente, deverá ser trafegado para todas as próximas requisições, e o campo `descStatus`, que demonstra o status da transação. O status dela será abordado logo em seguida.

```bash
$ curl \
    -X POST "http://127.0.0.1:40/transactions/" \
    -H "token: ubogfszb2y9iti8njcmar4e39cg73m"

$ {"data":{"_id":"8201903116563853","apiIntegrationScope":[],"status":1,"descStatus":"Created","date":"2019-01-31T16:56:03.824Z"},"statusLog":[{"type":3}]}
```

Após a criação do stack, o próximo passo é o envio dos dados dos recursos diversos para a API. Este envio deve ser feito através das rotas de escrita e atualização de informações exclusivos dos próprios recursos. 