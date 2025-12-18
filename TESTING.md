# API Testing

Manual API testing is performed using Insomnia.

## Tooling
- Insomnia REST Client
- Collection format: Insomnia v5

> File: `insomnia/chore-app.yaml`

## How to Run
1. Import the collection into Insomnia
2. Execute requests:
   - PUT Create Item
   - GET All Items
   - GET Item by ID
   - DELETE Item by ID

## Expected Results
- PUT returns 201 and generated UUID
- GET returns stored item(s)
- DELETE removes item
- GET after DELETE returns 404
