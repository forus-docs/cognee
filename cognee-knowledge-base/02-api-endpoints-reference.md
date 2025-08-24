# Cognee API Endpoints Reference

## Base URL
```
http://localhost:8000/api/v1
```

## Authentication
All endpoints require JWT authentication token in header:
```
Authorization: Bearer <token>
```

## Core Endpoints

### Data Management

#### Add Data
```http
POST /add
```
Add text or files to the knowledge base.

**Request Body:**
```json
{
  "data": "string | file",
  "dataset_name": "string (optional)",
  "node_set": "string (optional)"
}
```

**Response:**
```json
{
  "data_id": "uuid",
  "dataset_id": "uuid",
  "status": "success",
  "message": "Data added successfully"
}
```

#### List Datasets
```http
GET /datasets
```
Retrieve all datasets for the authenticated user.

**Response:**
```json
{
  "datasets": [
    {
      "id": "uuid",
      "name": "string",
      "created_at": "datetime",
      "item_count": "integer"
    }
  ]
}
```

#### Get Dataset Data
```http
GET /datasets/{dataset_id}/data
```
Get all data items in a specific dataset.

**Response:**
```json
{
  "data_items": [
    {
      "id": "uuid",
      "name": "string",
      "type": "string",
      "created_at": "datetime",
      "size": "integer"
    }
  ]
}
```

### Processing

#### Cognify
```http
POST /cognify
```
Process data into knowledge graph.

**Request Body:**
```json
{
  "dataset_name": "string (optional)",
  "graph_model_file": "string (optional)",
  "graph_model_name": "string (optional)"
}
```

**Response:**
```json
{
  "pipeline_run_id": "uuid",
  "status": "processing",
  "message": "Cognify pipeline started"
}
```

#### Pipeline Status
```http
GET /pipelines/{pipeline_run_id}/status
```
Check the status of a processing pipeline.

**Response:**
```json
{
  "pipeline_run_id": "uuid",
  "status": "processing | completed | failed",
  "progress": 0.75,
  "current_task": "extracting entities",
  "error": "string (if failed)"
}
```

### Search & Retrieval

#### Search
```http
POST /search
```
Query the knowledge graph.

**Request Body:**
```json
{
  "query": "string",
  "search_type": "GRAPH_COMPLETION | CHUNKS | INSIGHTS | SUMMARIES | CODE | CYPHER | FEELING_LUCKY",
  "dataset_name": "string (optional)",
  "limit": "integer (optional, default: 10)"
}
```

**Response:**
```json
{
  "results": [
    {
      "content": "string",
      "score": "float",
      "metadata": "object"
    }
  ],
  "search_type_used": "string",
  "query_time": "float"
}
```

#### Get Search History
```http
GET /search/history
```
Retrieve search history for the user.

**Response:**
```json
{
  "queries": [
    {
      "id": "uuid",
      "query": "string",
      "search_type": "string",
      "timestamp": "datetime",
      "result_count": "integer"
    }
  ]
}
```

### Data Deletion

#### Delete Data Item
```http
DELETE /data/{data_id}
```
Remove specific data from a dataset.

**Query Parameters:**
- `dataset_id`: UUID of the dataset
- `mode`: "soft" | "hard" (default: "soft")

**Response:**
```json
{
  "status": "success",
  "deleted_nodes": "integer",
  "message": "Data deleted successfully"
}
```

#### Prune All Data
```http
POST /prune
```
Reset entire knowledge graph (dangerous!).

**Request Body:**
```json
{
  "confirm": true
}
```

**Response:**
```json
{
  "status": "success",
  "message": "All data pruned successfully"
}
```

### Configuration

#### Get Settings
```http
GET /settings
```
Retrieve current configuration.

**Response:**
```json
{
  "llm": {
    "provider": "string",
    "model": "string"
  },
  "databases": {
    "vector": "string",
    "graph": "string",
    "relational": "string"
  },
  "storage": {
    "provider": "string"
  }
}
```

#### Update LLM Settings
```http
PUT /settings/llm
```
Update LLM configuration.

**Request Body:**
```json
{
  "provider": "openai | litellm | ollama",
  "model": "string",
  "api_key": "string (optional)"
}
```

### User Management

#### Create User
```http
POST /users
```
Register a new user.

**Request Body:**
```json
{
  "email": "string",
  "password": "string",
  "name": "string (optional)"
}
```

#### Get Current User
```http
GET /users/me
```
Get authenticated user information.

**Response:**
```json
{
  "id": "uuid",
  "email": "string",
  "name": "string",
  "created_at": "datetime",
  "permissions": ["string"]
}
```

### Visualization

#### Get Graph Visualization
```http
GET /visualize/graph
```
Get graph data for visualization.

**Query Parameters:**
- `dataset_name`: Filter by dataset
- `limit`: Maximum nodes to return

**Response:**
```json
{
  "nodes": [
    {
      "id": "string",
      "label": "string",
      "type": "string",
      "properties": "object"
    }
  ],
  "edges": [
    {
      "source": "string",
      "target": "string",
      "type": "string",
      "weight": "float"
    }
  ]
}
```

### Health & Monitoring

#### Health Check
```http
GET /health
```
Check service status.

**Response:**
```json
{
  "status": "healthy",
  "version": "0.2.3",
  "databases": {
    "vector": "connected",
    "graph": "connected",
    "relational": "connected"
  }
}
```

## WebSocket Endpoints

### Pipeline Updates
```
ws://localhost:8000/ws/pipeline/{pipeline_run_id}
```
Real-time pipeline status updates.

**Messages:**
```json
{
  "type": "progress",
  "progress": 0.5,
  "current_task": "string",
  "message": "string"
}
```

## Error Responses

All endpoints return standardized error responses:

```json
{
  "error": {
    "code": "string",
    "message": "string",
    "details": "object (optional)"
  },
  "status_code": "integer"
}
```

Common status codes:
- `400`: Bad Request (invalid input)
- `401`: Unauthorized (missing/invalid token)
- `403`: Forbidden (insufficient permissions)
- `404`: Not Found
- `429`: Too Many Requests (rate limited)
- `500`: Internal Server Error

## Rate Limiting

Default limits:
- 60 requests per minute per user
- 1000 requests per hour per user
- Configurable via environment variables

Headers in response:
- `X-RateLimit-Limit`: Maximum requests
- `X-RateLimit-Remaining`: Requests remaining
- `X-RateLimit-Reset`: Reset timestamp

## Pagination

List endpoints support pagination:

**Query Parameters:**
- `page`: Page number (default: 1)
- `per_page`: Items per page (default: 20, max: 100)

**Response Headers:**
- `X-Total-Count`: Total items
- `X-Page`: Current page
- `X-Per-Page`: Items per page