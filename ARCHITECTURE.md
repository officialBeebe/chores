# DynamoDB (Datestore)

Table name: `chore-app-items`

Partition key: `id`

# Lambda Function (Compute)

Function name: `chore-app-function`

Role name: `chore-app-role`

Role policy template: `Simple microservice permissions`

Function code:

``` python
import json
import boto3
from datetime import datetime
from botocore.exceptions import ClientError

dynamodb = boto3.resource("dynamodb")
table = dynamodb.Table("chore-app-items")


def lambda_handler(event, context):
    print(event)

    statusCode = 200
    headers = {"Content-Type": "application/json"}
    body = None

    try:
        route = event["routeKey"]

        # DELETE /items/{id}
        if route == "DELETE /items/{id}":
            item_id = event["pathParameters"]["id"]

            table.delete_item(Key={"id": item_id})
            body = {"message": f"Deleted item {item_id}"}

        # GET /items/{id}
        elif route == "GET /items/{id}":
            item_id = event["pathParameters"]["id"]

            response = table.get_item(Key={"id": item_id})
            if "Item" not in response:
                statusCode = 404
                body = {"message": "Item not found"}
            else:
                body = response["Item"]

        # GET /items
        elif route == "GET /items":
            response = table.scan()
            body = response.get("Items", [])

        # PUT /items
        elif route == "PUT /items":
            requestJSON = json.loads(event["body"])
            now = datetime.utcnow().isoformat()

            table.put_item(
                Item={
                    "id": requestJSON["id"],
                    "name": requestJSON["name"],
                    "description": requestJSON.get("description"),
                    "frequency": requestJSON["frequency"],
                    "dueDate": requestJSON["dueDate"],
                    "createdDate": now,
                    "updatedDate": now,
                },
                ConditionExpression="attribute_not_exists(id)"
            )

            statusCode = 201
            body = {"message": f"Created item {requestJSON['id']}"}

        else:
            statusCode = 400
            body = {"message": f"Unsupported route: {route}"}

    except ClientError as e:
        if e.response["Error"]["Code"] == "ConditionalCheckFailedException":
            statusCode = 409
            body = {"message": "Item already exists"}
        else:
            print(e)
            statusCode = 500
            body = {"message": "Internal server error"}

    return {
        "statusCode": statusCode,
        "headers": headers,
        "body": json.dumps(body),
    }

```

> Tutorial code needs adjusting for Chore App

# API Gateway (IPv4 HTTP API)

API name: `chore-app-api`

Routes:

- `GET /items/{id}`
- `GET /items`
- `PUT /items`
- `DELETE /items/{id}`

Integration: `chore-app-function` (Lambda function) on all routes

Testing: Insomnia
