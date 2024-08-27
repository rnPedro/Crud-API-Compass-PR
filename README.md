

# Projeto API CRUD com AWS

## Introdução
Este projeto demonstra a criação de uma API CRUD usando serviços da AWS, incluindo DynamoDB, Lambda, API Gateway e Cloud9. O objetivo é fornecer uma API simples para armazenar, recuperar, atualizar e excluir itens.

## Criar uma Tabela no DynamoDB
O Amazon DynamoDB é um serviço de banco de dados NoSQL gerenciado. Para criar uma tabela:
1. Acesse o console do DynamoDB em [DynamoDB Console](https://console.aws.amazon.com/dynamodb/)
2. Clique em **Criar tabela**.
3. Nomeie a tabela como `http-crud-tutorial-items` e defina a chave primária como `id`.
4. Clique em **Criar**.

## Criar uma Função Lambda
O AWS Lambda permite executar código sem gerenciar servidores. Para criar uma função Lambda:
1. Acesse o console do Lambda em [Lambda Console](https://console.aws.amazon.com/lambda)
2. Selecione **Criar função**.
3. Escolha **Autor do zero** e nomeie a função como `http-crud-tutorial-function`.
4. Selecione Node.js 14.x como tempo de execução e crie uma nova função de execução.
5. Use o seguinte código:

```javascript
const AWS = require("aws-sdk");
const dynamo = new AWS.DynamoDB.DocumentClient();

exports.handler = async (event, context) => {
  let body;
  let statusCode = 200;
  const headers = {
    "Content-Type": "application/json"
  };

  try {
    switch (event.routeKey) {
      case "DELETE /items/{id}":
        await dynamo.delete({
          TableName: "http-crud-tutorial-items",
          Key: { id: event.pathParameters.id }
        }).promise();
        body = `Deleted item ${event.pathParameters.id}`;
        break;
      case "GET /items/{id}":
        body = await dynamo.get({
          TableName: "http-crud-tutorial-items",
          Key: { id: event.pathParameters.id }
        }).promise();
        break;
      case "GET /items":
        body = await dynamo.scan({ TableName: "http-crud-tutorial-items" }).promise();
        break;
      case "PUT /items":
        let requestJSON = JSON.parse(event.body);
        await dynamo.put({
          TableName: "http-crud-tutorial-items",
          Item: {
            id: requestJSON.id,
            price: requestJSON.price,
            name: requestJSON.name
          }
        }).promise();
        body = `Put item ${requestJSON.id}`;
        break;
      default:
        throw new Error(`Unsupported route: "${event.routeKey}"`);
    }
  } catch (err) {
    statusCode = 400;
    body = err.message;
  } finally {
    body = JSON.stringify(body);
  }

  return {
    statusCode,
    body,
    headers
  };
};
```

## Criar uma API HTTP
Para criar uma API HTTP usando o API Gateway:
1. Acesse o console do API Gateway em [API Gateway Console](https://console.aws.amazon.com/apigateway)
2. Clique em **Criar API** e escolha **Build API HTTP**.
3. Nomeie a API como `http-crud-tutorial-api`.
4. Pular a configuração de rotas por enquanto e clique em **Criar**.

## Criar Rotas
As rotas para a API incluem:
- `GET /items/{id}`
- `GET /items`
- `PUT /items`
- `DELETE /items/{id}`

Para criar rotas:
1. Selecione sua API no console do API Gateway.
2. Vá para **Rotas** e clique em **Criar**.
3. Para cada rota, escolha o método e insira o caminho correspondente.

## Criar uma Integração
Conecte a API a uma função Lambda:
1. Vá para **Integrações** na sua API.
2. Selecione **Gerenciar integrações** e clique em **Criar**.
3. Escolha a função Lambda `http-crud-tutorial-function` e clique em **Criar**.

## Anexar Integração às Rotas
Anexe a integração a todas as rotas para que a função Lambda seja invocada:
1. Na seção **Integrações**, escolha uma rota.
2. Selecione a integração existente e clique em **Anexar integração**.
3. Repita para todas as rotas.

## Testar a API
Utilize `curl` para testar a API:
1. Copie a URL de invocação da sua API.
2. No Cloud9, configure a variável de ambiente:

```bash
export INVOKE_URL="https://abcdef123.execute-api.eu-west-1.amazonaws.com"
```

3. Para criar ou atualizar um item:

```bash
curl -X "PUT" -H "Content-Type: application/json" -d '{
    "id": "abcdef234",
    "price": 12345,
    "name": "myitem"
}' $INVOKE_URL/items
```

4. Para listar todos os itens:

```bash
curl -s $INVOKE_URL/items | js-beautify
```

5. Para obter um item específico:

```bash
curl -s $INVOKE_URL/items/abcdef234 | js-beautify
```

6. Para excluir um item:

```bash
curl -X "DELETE" $INVOKE_URL/items/abcdef234
```

7. Para verificar se o item foi excluído:

```bash
curl -s $INVOKE_URL/items | js-beautify
```

## Limpar Recursos
Para evitar custos, exclua todos os recursos criados:
1. **Tabela do DynamoDB**: Acesse o console e exclua a tabela.
2. **API HTTP**: Exclua a API no console do API Gateway.
3. **Função Lambda**: Exclua a função no console do Lambda.
4. **Grupo de Log**: Exclua o grupo de log no CloudWatch.
5. **Função de Execução**: Exclua a função no IAM.
6. **Cloud9 IDE**: Exclua o ambiente no Cloud9.

## Conclusão
Este projeto ilustra como construir uma API CRUD simples usando a infraestrutura serverless da AWS, envolvendo vários serviços. Ao seguir as etapas, você pode gerenciar facilmente os dados com DynamoDB e funções Lambda, integrando tudo por meio do API Gateway.

