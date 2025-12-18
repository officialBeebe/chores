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
import uuid
from decimal import Decimal
from datetime import datetime, timezone
from botocore.exceptions import ClientError

dynamodb = boto3.resource("dynamodb")
table = dynamodb.Table("chore-app-items")


class DecimalEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, Decimal):
            return int(obj) if obj % 1 == 0 else float(obj)
        return super().default(obj)


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

            now = datetime.now(timezone.utc).isoformat()
            item_id = str(uuid.uuid4())

            table.put_item(
                Item={
                    "id": item_id,
                    "name": requestJSON["name"],
                    "description": requestJSON.get("description"),
                    "frequency": requestJSON["frequency"],
                    "dueDate": requestJSON["dueDate"],
                    "createdDate": now,
                    "updatedDate": now,
                }
            )

            statusCode = 201
            body = {
                "message": "Created item",
                "id": item_id
            }

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
        "body": json.dumps(body, cls=DecimalEncoder),
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

# API Testing

- Insomnia REST Client
- Collection format: Insomnia v5

``` yaml
type: collection.insomnia.rest/5.0
schema_version: "5.1"
name: chore-app
meta:
  id: wrk_63bcba538275423fbfae94487bfddb52
  created: 1766030938267
  modified: 1766030938267
  description: ""
collection:
  - name: /items
    meta:
      id: fld_8e32419e56fa4ae386f1acafe1b53627
      created: 1766030995223
      modified: 1766031018149
      sortKey: -1766030995423
      description: ""
    children:
      - url: https://92rdl2pl35.execute-api.us-east-2.amazonaws.com/items
        name: Item
        meta:
          id: req_19f173033034439885886a39849d37f5
          created: 1766030947916
          modified: 1766034545521
          isPrivate: false
          description: ""
          sortKey: -1766031007870
        method: PUT
        body:
          mimeType: application/json
          text: |
            {
              "name": "Sweep upstairs",
              "description": "Sweep the upstairs hallway and bathroom.",
              "frequency": 3,
              "dueDate": "2025-01-01T00:00:00Z"
            }
        headers:
          - name: Content-Type
            value: application/json
          - name: User-Agent
            value: insomnia/12.1.0
            description: ""
            disabled: false
        settings:
          renderRequestBody: true
          encodeUrl: true
          followRedirects: global
          cookies:
            send: true
            store: true
          rebuildPath: true
      - url: https://92rdl2pl35.execute-api.us-east-2.amazonaws.com/items
        name: All Items
        meta:
          id: req_ba1a6e38204442f3b0a222d5c7f4d549
          created: 1766030980662
          modified: 1766034536849
          isPrivate: false
          description: ""
          sortKey: -1766031007920
        method: GET
        headers:
          - name: User-Agent
            value: insomnia/12.1.0
            description: ""
            disabled: false
        settings:
          renderRequestBody: true
          encodeUrl: true
          followRedirects: global
          cookies:
            send: true
            store: true
          rebuildPath: true
      - name: /{id}
        meta:
          id: fld_dc53f741dfdb4c95b2c5fe46639f3196
          created: 1766031028007
          modified: 1766031084283
          sortKey: -1766031007770
          description: ""
        children:
          - url: https://92rdl2pl35.execute-api.us-east-2.amazonaws.com/items/9681a9b2-81f1-45d6-ac37-da996ed7faf9
            name: Item by UUID
            meta:
              id: req_9bd469afb0ff482bb55f0130aa144c66
              created: 1766031039650
              modified: 1766034569728
              isPrivate: false
              description: ""
              sortKey: -1766031039650
            method: DELETE
            headers:
              - name: User-Agent
                value: insomnia/12.1.0
                description: ""
                disabled: false
            settings:
              renderRequestBody: true
              encodeUrl: true
              followRedirects: global
              cookies:
                send: true
                store: true
              rebuildPath: true
          - url: https://92rdl2pl35.execute-api.us-east-2.amazonaws.com/items/9681a9b2-81f1-45d6-ac37-da996ed7faf9
            name: Item by UUID
            meta:
              id: req_5a48e7eafa8547b989d898055b8cae46
              created: 1766031051696
              modified: 1766034557381
              isPrivate: false
              description: ""
              sortKey: -1766031039750
            method: GET
            headers:
              - name: User-Agent
                value: insomnia/12.1.0
                description: ""
                disabled: false
            settings:
              renderRequestBody: true
              encodeUrl: true
              followRedirects: global
              cookies:
                send: true
                store: true
              rebuildPath: true
cookieJar:
  name: Default Jar
  meta:
    id: jar_e24bffb7a79b888c9c75662c797b119cd7fa2420
    created: 1766030938269
    modified: 1766030938269
environments:
  name: Base Environment
  meta:
    id: env_e24bffb7a79b888c9c75662c797b119cd7fa2420
    created: 1766030938268
    modified: 1766030938268
    isPrivate: false
```
