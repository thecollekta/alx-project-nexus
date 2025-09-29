# REST API Documentation

## Base URL

All endpoints are prefixed with:
<https://project-nexus-backend-q5ai.onrender.com/schema/swagger-ui/> (for production)
<http://localhost:8000/schema/swagger-ui/> (for local development)

## Authentication

```http
POST /api/v1/accounts/login/
Content-Type: application/json

### Request Body

```json
{
    "email": "user@example.com",
    "password": "Securepassword123"
}
```

### Response Body

```json
{
    "refresh": "xxx.yyy.zzz",
    "access": "aaa.bbb.ccc"
}
```

## Endpoints

### Products

#### List Products

```http
GET /api/v1/products/products/
Authorization
```

**Query Parameters**:

- `category` (string): Filter by category slug
- `min_price` (number): Minimum price
- `max_price` (number): Maximum price
- `search` (string): Search term
- `ordering` (string): Field to order by (e.g., price, -created_at)

### Orders

#### Create Order

```http
POST /api/v1/orders/
Authorization: Bearer <access_token>
Content-Type: application/json

```json
{
  "email": "user@example.com",
  "phone_number": "string",
  "shipping_first_name": "string",
  "shipping_last_name": "string",
  "shipping_address_line_1": "string",
  "shipping_address_line_2": "string",
  "shipping_city": "string",
  "shipping_state": "string",
  "shipping_postal_code": "string",
  "shipping_country": "United States",
  "billing_same_as_shipping": true,
  "billing_first_name": "string",
  "billing_last_name": "string",
  "billing_address_line_1": "string",
  "billing_address_line_2": "string",
  "billing_city": "string",
  "billing_state": "string",
  "billing_postal_code": "string",
  "billing_country": "string",
  "customer_notes": "string"
}
```

### Error Handling

Standard HTTP status codes are used:

- `200 OK`: Successful GET requests
- `201 Created`: Successful POST requests
- `400 Bad Request`: Invalid request data
- `401 Unauthorized`: Authentication required
- `403 Forbidden`: Insufficient permissions
- `404 Not Found`: Resource not found
- `500 Internal Server Error`: Server error

### Rate Limiting

1000 requests per hour per IP for public endpoints
10000 requests per hour per user for authenticated endpoints
