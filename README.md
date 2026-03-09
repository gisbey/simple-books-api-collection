# Simple Books API – Postman Collection

A complete Postman collection for testing and exploring the **Simple Books API** – a RESTful API for browsing books and managing orders. This collection doubles as both a reference guide and an end-to-end workflow test suite.

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites & Setup](#prerequisites--setup)
3. [Authentication](#authentication)
4. [Collection Variables](#collection-variables)
5. [Endpoints by Resource](#endpoints-by-resource)
   - [Status](#status)
   - [Books](#books)
   - [Orders](#orders)
   - [API Clients](#api-clients)
6. [Example Workflows](#example-workflows)
7. [Running the Collection](#running-the-collection)

---

## Overview

The **Simple Books API** allows clients to:

- Check the health/status of the API
- Register as an API client to obtain an access token
- Browse a catalogue of books (fiction and non-fiction)
- Place, retrieve, update, and delete book orders

**Base URL:** `https://simple-books-api.click`

This collection is structured as an **end-to-end workflow** – requests are ordered to be run sequentially. Each request builds on the previous one, 
with test scripts that automatically capture values (e.g. `bookId`, `orderId`) into collection variables for use in subsequent requests.

---

## Prerequisites & Setup

### 1. Import the Collection

Import the collection into Postman via **File → Import** or by using the Postman link shared with you.

### 2. Set the Base URL

The `baseUrl` collection variable is pre-configured to:

```
https://simple-books-api.click
```

### 3. Register as an API Client

Before placing orders, you must register as an API client to receive an **access token**. Send the **Register API Client** request – (see [API Clients](#api-clients) below). The token returned should be stored in the `accessToken` collection variable.

## Authentication

Most read operations (checking status, listing books) are **unauthenticated**. However, all **order management** endpoints require a Bearer token.

### Obtaining a Token

Send a `POST` request to `/api-clients` with your client details:

```http
POST {{baseUrl}}/api-clients
Content-Type: application/json

{
  "clientName": "Your Name",
  "clientEmail": "your@email.com"
}
```

**Response:**

```json
{
  "accessToken": "your-access-token-here"
}
```

### Using the Token

Include the token in the `Authorization` header for all order-related requests:

```http
Authorization: Bearer Token {{accessToken}}
```

> **Note:** Each email address can only be registered once. If you receive a `"API client already registered.  Try a different email."` error, use a different email address or retrieve your existing token.

Store the token in the `accessToken` collection variable so all requests can reference it automatically via `{{accessToken}}`.

---

## Collection Variables

| Variable       | Description                                                                | Example Value                          |
|----------------|----------------------------------------------------------------------------|----------------------------------------|
| `baseUrl`      | The root URL of the API                                                    | `https://simple-books-api.click`       |
| `accessToken`  | Bearer token obtained after registering an API client                      | `abc123xyz...`                         |
| `bookId`       | ID of the book to order (set automatically by tests)                       | `1`                                    |
| `orderId`      | ID of the created order (set automatically by tests)                       | `PF6MflPDcuhWobZcgmJy5`                |
| `customerName` | Name of the customer placing the order (set automatically by tests)        | `John Doe`                             |
| `email`        | Email used to register API client (set automatically by tests)             | `John.Doe@example.com`                 |

> Variables marked as "set automatically by tests" are populated by the test scripts in earlier requests – no manual input needed when running the full collection in order.

---

## Endpoints by Resource

### Status

#### `GET /status` – Check API Status

Verifies the API is online and operational.

```http
GET {{baseUrl}}/status
```

**Response:**

```json
{
  "status": "OK"
}
```

**Tests:** 
- Certifies the response status code is 200 OK.
- Verifies the response time is under 500ms.
- Confirms the response body is JSON.
- Checks that status is "OK". 

---

### Books

#### `GET /books` – List Books

Returns a list of available books. Supports optional filtering by type.

```http
GET {{baseUrl}}/books
```

**Query Parameters:**

| Parameter | Type   | Required | Description                                |
|-----------|--------|----------|--------------------------------------------|
| `type`    | string | No       | Filter by type: `fiction` or `non-fiction` |
| `limit`   | number | No       | Limit the number of results (1–20)         |

**Response:**

```json
[
    {
        "id": 1,
        "name": "The Russian",
        "type": "fiction",
        "available": true
    },
    {
        "id": 3,
        "name": "The Vanishing Half",
        "type": "fiction",
        "available": true
    },
    {
        "id": 4,
        "name": "The Midnight Library",
        "type": "fiction",
        "available": true
    },
    {
        "id": 6,
        "name": "Viscount Who Loved Me",
        "type": "fiction",
        "available": true
    },
    {
        "id": 2,
        "name": "Just as I Am",
        "type": "non-fiction",
        "available": false
    },
    {
        "id": 5,
        "name": "Untamed",
        "type": "non-fiction",
        "available": true
    }
]
```

**Tests:** 
- Certifies the response status code is `200 OK`.
- Verifies the response time is under `500ms`.
- Confirms the response body is JSON.
- Checks the expected array objects are present.
- Selects a random available fiction book.
- Saves its `id` to the `bookId` collection variable.
- Save its `name` to the `customerName` collection variable.

---

#### `GET /books/:bookId` – Get Single Book

Returns detailed information about a specific book.

```http
GET {{baseUrl}}/books/:bookId
```

**Path Parameters:**

| Parameter | Type     | Required | Description         |
|-----------|----------|----------|---------------------|
| `bookId`  | integer  | Yes      | The ID of the book  |

**Response:**

```json
{
    "id": 4,
    "entityType": "BOOK",
    "available": true,
    "timestamp": 1752239490804,
    "created": "2025-07-11T13:11:30.804Z",
    "GSI1SK": "BOOK#4",
    "name": "The Midnight Library",
    "current-stock": 87,
    "GSI1PK": "TYPE#fiction",
    "price": 15.6,
    "PK": "BOOK#4",
    "author": "Matt Haig",
    "type": "fiction",
    "SK": "METADATA"
}
```

**Tests:** 
- Certifies the response status code is `200 OK`.
- Verifies the response time is under `500ms`.
- Confirms the response body is JSON.
- Checks the expected array objects are present.
- Validates that your desired book is in stock.

---

### Orders

> ⚠️ All order endpoints require the `Authorization: Bearer {{accessToken}}` header.

#### `POST /orders` – Place Order

Creates a new book order under a random name.

```http
POST {{baseUrl}}/orders
Authorization: Bearer {{accessToken}}
Content-Type: application/json

{
  "bookId": {{bookId}},
  "customerName": "{{$randomFullName}}"
}
```

**Response:**

```json
{
  "created": true,
  "orderId": "PF6MflPDcuhWobZcgmJy5"
}
```

**Tests:** 
- Certifies the response status code is `201 Created`.
- Verifies the response time is under `500ms`.
- Confirms the response body is JSON.
- Checks the expected array objects are present.
- Verifies that the order was created.
- Saves the returned `orderId` to the `orderId` collection variable.

---

#### `GET /orders` – Get All Orders

Returns a list of all orders placed by an authenticated client.

```http
GET {{baseUrl}}/orders
Authorization: Bearer {{accessToken}}
```

**Response:**

```json
[
    {
        "id": "E-tHTTa4mZisAKyo-1q95",
        "bookId": 5,
        "customerName": "John doe",
        "quantity": 1,
        "timestamp": 1772464784992
    }
]
```

**Tests:** 
- Certifies the response status code is `200 OK`.
- Verifies the response time is under `500ms`.
- Confirms the response body is JSON.
- Checks the expected array objects are present.
- Verifies that your order is present.
- Adds the `customerName` from the order to the `customerOrder` collection variable.

---

#### `GET /orders/:orderId` – Get Single Order

Returns details of a specific order.

```http
GET {{baseUrl}}/orders/:orderId
Authorization: Bearer {{accessToken}}
```

**Path Parameters:**

| Parameter | Type   | Required | Description          |
|-----------|--------|----------|----------------------|
| `orderId` | string | Yes      | The ID of the order  |

**Response:**

```json
{
  "id": "PF6MflPDcuhWobZcgmJy5",
  "bookId": 2,
  "customerName": "John Doe",
  "quantity": 1,
  "timestamp": 1636100000000
}
```

This request appears **three times** in the collection, each with a different test assertion:

| Request Name            | Purpose                                                     |
|-------------------------|-------------------------------------------------------------|
| Get Single Order        | Verifies the order exists after creation                    |
| Get Single Order        | Confirms the customer name was updated                      |
| Get Single Order        | Confirms the order returns `404 Not Found` after deletion   |

---

#### `PATCH /orders/:orderId` – Update Order

Updates the customer name on an existing order with a new random name.

```http
PATCH {{baseUrl}}/orders/:orderId
Authorization: Bearer {{accessToken}}
Content-Type: application/json

{
  "customerName": "Updated Name"
}
```

**Response:** `204 No Content` (no body returned on success)

**Response:** `404 Not Found` (when order id cannot be found)

```json
{
  "error": "No order with id To92ErcmDlfJTfYyanj52."
}
```

**Tests:** 
- Certifies the response status code is `200 OK`.
- Verifies the response time is under `500ms`.
- Confirms the response body is empty.

---

#### `DELETE /orders/:orderId` – Delete Order

Permanently deletes an order.

```http
DELETE {{baseUrl}}/orders/:orderId
Authorization: Bearer {{accessToken}}
```

**Response:** `204 No Content`

This request appears **twice** in the collection, each with a different test assertion:

| Request Name          | Purpose                                                     |
|-----------------------|-------------------------------------------------------------|
| Delete Order          | Confirms the response body is empty on `204 No Content`     |
| Delete Order          | Confirms the correct error takes place on `404 Not Found`   |

---

### API Clients

#### `POST /api-clients` – Register API Client

Registers a new client and returns an access token. Each email can only be registered once.

The randomly generated email will be saved to `email` in collection variables and reused later.

```http
POST {{baseUrl}}/api-clients
Content-Type: application/json

{
  "clientName": "Your Name",
  "clientEmail": "your@email.com"
}
```

**Response (success):**

```json
{
  "accessToken": "your-access-token-here"
}
```

**Response (already registered):**

```json
{
  "error": "API client already registered. Try a different email."
}
```

This request appears **twice** in the collection, each with a different test assertion:

| Request Name               | Purpose                                                                                    |
|----------------------------|--------------------------------------------------------------------------------------------|
| Register API client        | Checks the response status is `201 Created`, as a new user.                                |
| Register API client        | Verifies the response status is `409 Conflict`, as the user that is already registered.    |

---

## Example Workflows

### End-to-End Order Workflow

This is the primary workflow the collection is designed to demonstrate. Run all requests **in order** using the Collection Runner:

```
1.  GET  /status                    → Verify API is online
2.  POST /api-clients               → Registers new user
3.  GET  /books                     → Find an available book
4.  GET  /books/:bookId             → Confirm book details
5.  POST /orders                    → Place an order
6.  GET  /orders                    → Verify order appears in list
7.  GET  /orders/:orderId           → Retrieve the specific order
8.  PATCH /orders/:orderId          → Update the customer name
9.  GET  /orders/:orderId           → Verify the name was updated
10. DELETE /orders/:orderId         → Delete the order
11. GET  /orders/:orderId           → Verify the order is gone
12. DELETE /orders/:orderId         → Triggers "No order with id" error
13. POST /api-clients               → Triggers "API client already registered" error
```

### Quick Book Browse (No Auth Required)

```
1. GET /status
2. GET /books?type=fiction
3. GET /books/:bookId
```

---

## Running the Collection

### Option 1: Collection Runner (Manual)

1. Open the [Simple Books API](collection/17318486-9a7cbdad-23ba-4797-ae5c-59bbf3206fe7) collection in Postman
2. Click **Run collection**
3. Ensure all 13 requests are selected and in order
4. Click **Run Simple Books API**

### Option 2: Postman CLI (CI/CD)

```bash
# Install the Postman CLI
curl -o- "https://dl-cli.pstmn.io/install.sh" | sh

# Login
postman login --with-api-key YOUR_POSTMAN_API_KEY

# Run the collection
postman collection run 17318486-9a7cbdad-23ba-4797-ae5c-59bbf3206fe7
```

### Option 3: Scheduled Monitor

Set up a Postman Monitor to run this collection on a schedule for continuous API health monitoring:

1. In the collection, click **...** → **Monitor collection**
2. Set your desired schedule (e.g. every hour)
3. Postman will alert you if any tests fail

---

## Notes

- The API is hosted on [Glitch](https://glitch.me) and may have a **cold start delay** of a few seconds if it hasn't been accessed recently.
- The `bookId`, `orderId`, and `customerName` variables are set automatically by test scripts – you do not need to set them manually when running the full workflow.
- Orders are scoped to your `accessToken` – you can only view and manage orders created with your own token.
