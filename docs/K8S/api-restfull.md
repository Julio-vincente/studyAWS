Boa 👌 entendi, você quer uma documentação em **Markdown**, no mesmo estilo da do *Istio* que me mostrou, mas dessa vez explicando como montar uma **self-service REST API** usando **Lambda + API Gateway + DynamoDB**.
E dentro da doc vou colocar exemplos de Lambda function em **Node.js**, **Python** e até outro (ex: Go).

Aqui vai o `.md`:

---

# Self-Service RESTful API com AWS Lambda

## Referências

* [Amazon API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html)
* [AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)
* [Amazon DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html)

---

## Arquitetura

1. **API Gateway** expõe endpoints REST (`/employees`).
2. **Lambda Functions** recebem requisições HTTP e processam lógica.
3. **DynamoDB** armazena dados de forma escalável.
4. **IAM Roles** garantem permissões seguras.

---

## Infraestrutura Básica

### DynamoDB

Crie a tabela `Employees`:

* **Table name**: `Employees`
* **Partition key**: `employeeId (String)`

### Variável de Ambiente no Lambda

```env
DYNAMODB_TABLE=Employees
```

---

## Exemplo Lambda – Python

```python
import json
import boto3
import os

dynamodb = boto3.resource("dynamodb")
table = dynamodb.Table(os.environ["DYNAMODB_TABLE"])

def lambda_handler(event, context):
    method = event["httpMethod"]
    path = event["path"]

    if path == "/employees" and method == "GET":
        result = table.scan()
        return response(200, result.get("Items", []))
    elif path == "/employees" and method == "POST":
        body = json.loads(event["body"])
        table.put_item(Item=body)
        return response(201, {"message": "Employee created", "employee": body})
    else:
        return response(400, {"error": "Unsupported route"})

def response(status, body):
    return {
        "statusCode": status,
        "headers": {"Content-Type": "application/json"},
        "body": json.dumps(body)
    }
```

---

## Exemplo Lambda – Node.js

```javascript
const AWS = require("aws-sdk");
const dynamodb = new AWS.DynamoDB.DocumentClient();
const TABLE_NAME = process.env.DYNAMODB_TABLE;

exports.handler = async (event) => {
  const method = event.httpMethod;
  const path = event.path;

  if (path === "/employees" && method === "GET") {
    const data = await dynamodb.scan({ TableName: TABLE_NAME }).promise();
    return response(200, data.Items);
  } else if (path === "/employees" && method === "POST") {
    const body = JSON.parse(event.body);
    await dynamodb.put({ TableName: TABLE_NAME, Item: body }).promise();
    return response(201, { message: "Employee created", employee: body });
  } else {
    return response(400, { error: "Unsupported route" });
  }
};

function response(statusCode, body) {
  return {
    statusCode,
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(body),
  };
}
```

---

## Exemplo Lambda – Go

```go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "os"

    "github.com/aws/aws-lambda-go/events"
    "github.com/aws/aws-lambda-go/lambda"
    "github.com/aws/aws-sdk-go/aws/session"
    "github.com/aws/aws-sdk-go/service/dynamodb"
    "github.com/aws/aws-sdk-go/service/dynamodb/dynamodbattribute"
)

var (
    db    = dynamodb.New(session.Must(session.NewSession()))
    table = os.Getenv("DYNAMODB_TABLE")
)

type Employee struct {
    EmployeeID string `json:"employeeId"`
    Name       string `json:"name"`
}

func handler(ctx context.Context, req events.APIGatewayProxyRequest) (events.APIGatewayProxyResponse, error) {
    switch req.HTTPMethod {
    case "GET":
        result, _ := db.Scan(&dynamodb.ScanInput{TableName: &table})
        var employees []Employee
        _ = dynamodbattribute.UnmarshalListOfMaps(result.Items, &employees)
        body, _ := json.Marshal(employees)
        return events.APIGatewayProxyResponse{StatusCode: 200, Body: string(body)}, nil
    case "POST":
        var emp Employee
        json.Unmarshal([]byte(req.Body), &emp)
        item, _ := dynamodbattribute.MarshalMap(emp)
        db.PutItem(&dynamodb.PutItemInput{TableName: &table, Item: item})
        return events.APIGatewayProxyResponse{StatusCode: 201, Body: fmt.Sprintf("Employee %s created", emp.EmployeeID)}, nil
    default:
        return events.APIGatewayProxyResponse{StatusCode: 400, Body: "Unsupported route"}, nil
    }
}

func main() {
    lambda.Start(handler)
}
```

---

## Validação

### Teste com curl

```bash
# Criar funcionário
curl -X POST https://<api-id>.execute-api.<region>.amazonaws.com/prod/employees \
  -H "Content-Type: application/json" \
  -d '{"employeeId": "123", "name": "Julio"}'

# Listar funcionários
curl https://<api-id>.execute-api.<region>.amazonaws.com/prod/employees
```

---

👉 Essa doc já cobre: **infra mínima + exemplos em Python, Node.js e Go**.

Quer que eu adicione também uma parte de **infra como código (CloudFormation/SAM ou Terraform)** nessa doc, pra já provisionar a API, Lambda e DynamoDB de uma vez?
