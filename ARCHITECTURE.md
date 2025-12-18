## DynamoDB (Datestore)

Table name: `chore-app-items`

Partition key: `id`

## Lambda Function (Compute)

Function name: `chore-app-function`

Role name: `chore-app-role`

Role policy template: `Simple microservice permissions`

## API Gateway (IPv4 HTTP API)

API name: `chore-app-api`

Routes:

- `GET /items/{id}`
- `GET /items`
- `PUT /items`
- `DELETE /items/{id}`

Integration: `chore-app-function` (Lambda function) on all routes

## Testing

The API is tested manually using Insomnia.
See `TESTING.md` and the exported Insomnia collection for details.
