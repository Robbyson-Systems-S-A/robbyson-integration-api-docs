## Endereços e formas de consulta

A API de Integração do Robbyson foi desenvolvida na arquitetura [REST](https://en.wikipedia.org/wiki/Representational_state_transfer). Desta forma, toda comunicação é baseada em requisições `https`, não dependendo da linguagem escolhida como cliente desta API. Esta arquitetura também prevê a utilização dos [verbos http](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods) para denotar quais ações estão sendo executadas em determinado momento. 

De forma geral, entende-se que podem ser executadas três tipos diferentes de ações:

| Método | Descrição |
| - | - |
| `POST` | Este método denota a criação de um novo recurso na API. Ele é responsável pelo envio de dados novos à nossa API. Outra função, que será detalhada mais à frente, é a criação e conclusão de transações na API. |
| `PUT` | Método também de escrita de dados na API, porém com responsabilidade de atualização dos dados. |
| `GET` | Método de consulta de dados na API. Este método não produz nenhum tipo de alteração nos dados enviados para a API |

Dado o funcionamento via requisições aos serviços, deve-se notar que a construção da URL consultada é feita pelo endereço do serviço a ser consumido - base de produção ou de testes - mais o caminho do recurso. Os recursos também serão detalhados nos próximos tópicos.

| Ambiente | Endereço |
| - | - | 
| Produção | https://integration.api.robbyson.com/v1 |
| Testes | https://integration.sandbox.robbyson.com/v1 |