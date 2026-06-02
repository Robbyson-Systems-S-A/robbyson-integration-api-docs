# Recursos de integração de dados

Esta seção descreve os recursos do grupo de **integração de dados de operação**: colaboradores, indicadores, atributos, hierarquias e resultados. Todos seguem o mesmo fluxo de envio em lote orquestrado por *transactions* (veja [Workflow](workflow.rst)) e são autenticados via **Token de cliente** (veja [Autenticação](autenticacao.md#modalidade-1--token-de-cliente-recursos-de-integração-de-dados)).

O grupo **AI Agent Runtime** (resposta de assistentes de IA) tem seção própria — veja [AI Agent Runtime](ai-agent-runtime.md).

A URL final é composta pelo endereço do ambiente (`https://integration-api.robbyson.com/v1`) + caminho do recurso descrito abaixo.

## Colaboradores

**Caminho:** `/collaborators`

Cadastro do colaborador. Campos `name`, `identification`, `matricula`, `genre`, `active` e `birthDate` são obrigatórios.

```json
[
  {
    "name": "string",
    "identification": "string",
    "matricula": "string",
    "genre": "M",
    "active": true,
    "birthDate": "2018-12-07",
    "admissionDate": "2018-12-07",
    "email": "user@example.com",
    "civilState": "string",
    "street": "string",
    "number": 0,
    "neighborhood": "string",
    "city": "string",
    "state": "string",
    "country": "string",
    "placeOfBirth": "string",
    "homeNumber": "string",
    "phoneNumber": "string",
    "schooling": "string",
    "streetWorkPlace": "string",
    "numberWorkPlace": 0,
    "neighborhoodWorkPlace": "string",
    "cityWorkPlace": "string",
    "stateWorkPlace": "string",
    "countryWorkPlace": "string",
    "contractorControlId": "string",
    "dependantsNumber": 0
  }
]
```

## Indicadores

**Caminho:** `/indicators`

Catálogo de indicadores que serão pontuados nos resultados. `indicatorId` precisa ser único por contratante.

```json
[
  {
    "name": "string",
    "indicatorId": "string",
    "description": "string"
  }
]
```

## Atributos

**Caminho:** `/attributes`

Atributos qualificadores do colaborador (área, função, equipe, etc.) usados pela plataforma para segmentação de visibilidade.

```json
[
  {
    "collaboratorIdentification": "string",
    "name": "string",
    "level": 0,
    "value": "string",
    "date": "2018-12-07"
  }
]
```

> **Importante:** atributos podem ser informados para um intervalo completo se o mesmo valor se repetir no período. Para isso, substitua `date` por `startDate` + `endDate` contemplando o intervalo.

## Hierarquia

**Caminho:** `/hierarchies`

Posição do colaborador na estrutura organizacional. `levelWeight` define a profundidade do nível (1 = topo, valores maiores = subordinados); `parentIdentification` aponta para o `identification` do superior imediato.

```json
[
  {
    "collaboratorIdentification": "string",
    "levelName": "string",
    "levelWeight": 0,
    "parentIdentification": "string",
    "contractorControlId": "string",
    "date": "2018-12-07"
  }
]
```

> **Importante:** assim como atributos, hierarquias podem ser informadas para um intervalo completo usando `startDate` + `endDate` em vez de `date`. **Dados de hierarquias e atributos devem obrigatoriamente ser enviados na mesma transação** — veja [Workflow](workflow.rst).

## Resultados

**Caminho:** `/results`

Pontuação ou medição do colaborador em um indicador específico em uma data. `factors` é opcional e carrega os componentes que compuseram o resultado, quando aplicável.

```json
[
  {
    "collaboratorIdentification": "string",
    "indicatorId": "string",
    "result": 0,
    "date": "2018-12-07",
    "factors": [
      1,
      2,
      3
    ]
  }
]
```

## Transactions

**Caminho:** `/transactions`

Não é um recurso de dados propriamente dito, mas o **mecanismo de controle** do envio em lote — abertura, commit e consulta de status. Cada pacote enviado para os recursos acima é referenciado por um `transaction_id`. Detalhamento completo em [Workflow](workflow.rst).
