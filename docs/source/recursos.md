## Recursos da API

Entende-ser por recursos da API todos os tipos de dados que são trafegados por ela.

Conforme dito anteriormente, os recursos são acessados através de uma url, que será composta pelo endereço do serviço utilizado (produção ou sandbox) mais uma identificação do recurso consumido. Abaixo serão descritos cada um deles, com os detalhes para acesso.

### Colaboradores
**Identificação:** `/collaborators`

```js
[
  {
    "name": "string",               // campo obrigatório
    "identification": "string",     // campo obrigatório
    "matricula": "string",          // campo obrigatório
    "genre": "M",                   // campo obrigatório
    "active": true,                 // campo obrigatório
    "birthDate": "2018-12-07",      // campo obrigatório
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

### Indicadores
**Identificação:** `/indicators`

```js
[
  {
    "name": "string",
    "indicatorId": "string",
    "description": "string"
  }
]
```

### Atributos
**Identificação:** `/attributes`

```js
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

> **Importante:** As datas dos atributos podem ser informadas para um intervalo completo, se estas se repetirem por todo este período. Para isso, basta substituir o campo `date` por `startDate` e `endDate`, contemplando todo o intervalo a ser integrado.

### Hierarquia
**Identificação:** `/hierarchies`

```js
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

> **Importante:** As datas da  Hierarquia podem ser informadas para um intervalo completo, se estas se repetirem por todo este período. Para isso, basta substituir o campo `date` por `startDate` e `endDate`, contemplando todo o intervalo a ser integrado.

### Resultados
**Identificação:** `/results`

```js
[
  {
    "collaboratorIdentification": "string",
    "indicadorId": "string",
    "resultado": "string",
    "date": "2018-12-07",
    "factors": [
      1,
      2,
      3
    ]
  }
]
```