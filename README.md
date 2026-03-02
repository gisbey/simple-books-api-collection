# Simple Books API — Postman Collection

A complete Postman collection for testing and exploring the **Simple Books API** — a RESTful API for browsing books and managing orders. This collection doubles as both a reference guide and an end-to-end workflow test suite.

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

This collection is structured as an **end-to-end workflow** — requests are ordered to be run sequentially. Each request builds on the previous one, 
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

Before placing orders, you must register as an API client to receive an **access token**. Send the **Register API Client** request (see [API Clients](#api-clients) below). The token returned should be stored in the `accessToken` collection variable.

### 4. Set Your Customer Name

Update the `customerName` collection variable with the name you want to use when placing orders.

---

## Authentication

Most read operations (listing books, checking status) are **unauthenticated**. However, all **order management** endpoints require a Bearer token.

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
Authorization: Bearer {{accessToken}}
```

> **Note:** Each email address can only be registered once. If you receive a `"API client already registered"` error, use a different email address or retrieve your existing token.

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

> Variables marked as "set automatically by tests" are populated by the test scripts in earlier requests — no manual input needed when running the full collection in order.

---

## Endpoints by Resource

### Status

#### `GET /status` — Check API Status

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

**Tests:** Asserts the response status code is `200 OK`.

---

### Books

#### `GET /books?type=non-fiction` — List Books

Returns a list of available books. Supports optional filtering by type.

```http
GET {{baseUrl}}/books?type=non-fiction
```

**Query Parameters:**

| Parameter | Type   | Required | Description                          |
|-----------|--------|----------|--------------------------------------|
| `type`    | string | No       | Filter by type: `fiction` or `non-fiction` |
| `limit`   | number | No       | Limit the number of results (1–20)   |

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
    "id": 2,
    "name": "Just as I Am",
    "type": "non-fiction",
    "available": true
  }
]
```

**Tests:** Finds the first available non-fiction book and saves its `id` to the `bookId` collection variable.

---

#### `GET /books/:bookId` — Get Single Book

Returns detailed information about a specific book.

```http
GET {{baseUrl}}/books/:bookId
```

**Path Parameters:**

| Parameter | Type   | Required | Description         |
|-----------|--------|----------|---------------------|
| `bookId`  | number | Yes      | The ID of the book  |

**Response:**

```json
{
  "id": 2,
  "name": "Just as I Am",
  "author": "Billy Graham",
  "isbn": "978-0061176210",
  "type": "non-fiction",
  "price": 10.99,
  "current-stock": 12,
  "available": true
}
```

**Tests:** Verifies the returned book matches the `bookId` stored in the collection variable.

---

### Orders

> ⚠️ All order endpoints require the `Authorization: Bearer {{accessToken}}` header.

#### `POST /orders` — Place an Order

Creates a new book order.

```http
POST {{baseUrl}}/orders
Authorization: Bearer {{accessToken}}
Content-Type: application/json

{
  "bookId": {{bookId}},
  "customerName": "{{customerName}}"
}
```

**Response:**

```json
{
  "created": true,
  "orderId": "PF6MflPDcuhWobZcgmJy5"
}
```

**Tests:** Saves the returned `orderId` to the `orderId` collection variable.

---

#### `GET /orders` — Get All Orders

Returns a list of all orders placed by the authenticated client.

```http
GET {{baseUrl}}/orders
Authorization: Bearer {{accessToken}}
```

**Response:**

```json
[
  {
    "id": "PF6MflPDcuhWobZcgmJy5",
    "bookId": 2,
    "customerName": "John Doe",
    "createdBy": "client-id",
    "quantity": 1,
    "timestamp": 1636100000000
  }
]
```

**Tests:** Verifies the recently placed order appears in the list.

---

#### `GET /orders/:orderId` — Get a Single Order

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
  "createdBy": "client-id",
  "quantity": 1,
  "timestamp": 1636100000000
}
```

This request appears **three times** in the collection, each with a different test assertion:

| Request Name                        | Purpose                                      |
|-------------------------------------|----------------------------------------------|
| Get an order — get your specified order | Verifies the order exists after creation     |
| Get an order — verify name changed  | Confirms the customer name was updated       |
| Get an order — verify order deleted | Confirms the order returns `404` after deletion |

---

#### `PATCH /orders/:orderId` — Update an Order

Updates the customer name on an existing order.

```http
PATCH {{baseUrl}}/orders/:orderId
Authorization: Bearer {{accessToken}}
Content-Type: application/json

{
  "customerName": "Updated Name"
}
```

**Response:** `204 No Content` (no body returned on success)

**Tests:** Asserts the response status is `204`.

---

#### `DELETE /orders/:orderId` — Delete an Order

Permanently deletes an order.

```http
DELETE {{baseUrl}}/orders/:orderId
Authorization: Bearer {{accessToken}}
```

**Response:** `204 No Content`

**Tests:** Asserts the response status is `204`.

---

### API Clients

#### `POST /api-clients` — Register an API Client

Registers a new client and returns an access token. Each email can only be registered once.

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

> This request is labelled **"client already registered"** in the collection — it is included to demonstrate the error response when re-registering with the same email.

---

## Example Workflows

### End-to-End Order Workflow

This is the primary workflow the collection is designed to demonstrate. Run all requests **in order** using the Collection Runner:

```
1.  GET  /status                    → Verify API is online
2.  GET  /books?type=non-fiction    → Find an available book (saves bookId)
3.  GET  /books/:bookId             → Confirm book details
4.  POST /orders                    → Place an order (saves orderId)
5.  GET  /orders                    → Verify order appears in list
6.  GET  /orders/:orderId           → Retrieve the specific order
7.  PATCH /orders/:orderId          → Update the customer name
8.  GET  /orders/:orderId           → Verify the name was updated
9.  DELETE /orders/:orderId         → Delete the order
10. GET  /orders/:orderId           → Verify the order is gone (expects 404)
11. POST /api-clients               → Demonstrates "already registered" error
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
3. Ensure all 11 requests are selected and in order
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
- The `bookId` and `orderId` variables are set automatically by test scripts — you do not need to set them manually when running the full workflow.
- Orders are scoped to your `accessToken` — you can only view and manage orders created with your own token.
