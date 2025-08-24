# Cognee Database Configuration Guide

## Overview

Cognee uses a triple-database architecture where each database serves a specific purpose. All three databases work together to provide a complete knowledge graph system.

## Database Types & Purposes

### 1. Vector Database
**Purpose**: Store and search embeddings for semantic similarity

### 2. Graph Database  
**Purpose**: Store entities and relationships for graph traversal

### 3. Relational Database
**Purpose**: Store metadata, configuration, and system data

## Configuration Methods

### 1. Environment Variables (.env file)
```bash
# LLM Configuration (Required for some search types)
LLM_API_KEY=your-api-key
LLM_PROVIDER=openai  # or litellm, ollama
LLM_MODEL=gpt-4  # or specific model name

# Database Selection
GRAPH_DB_PROVIDER=kuzu  # or neo4j, falkordb
VECTOR_DB_PROVIDER=lancedb  # or chromadb, pgvector, neptune
RELATIONAL_DB_PROVIDER=sqlite  # or postgres

# Storage Configuration
FILE_STORAGE_PROVIDER=local  # or s3
```

### 2. Python Configuration
```python
import cognee
from cognee.config import Config

config = Config()
config.llm_provider = "openai"
config.graph_db_provider = "neo4j"
config.vector_db_provider = "chromadb"
config.relational_db_provider = "postgres"
```

## Vector Database Options

### LanceDB (Default - Embedded)
```bash
VECTOR_DB_PROVIDER=lancedb
# No additional configuration needed
# Data stored in: .cognee_system/databases/cognee.lancedb/
```

**Pros**:
- No external dependencies
- Fast for small-medium datasets
- Embedded, serverless
- Good for development

**Cons**:
- Limited scalability
- Single-machine only

### ChromaDB
```bash
VECTOR_DB_PROVIDER=chromadb
CHROMADB_HOST=localhost
CHROMADB_PORT=8000
```

**Setup**:
```bash
# Run ChromaDB server
pip install chromadb
chroma run --host localhost --port 8000
```

**Pros**:
- Good performance
- Client-server architecture
- Persistent storage
- Metadata filtering

**Cons**:
- Requires separate server
- Memory intensive

### PGVector (PostgreSQL)
```bash
VECTOR_DB_PROVIDER=pgvector
DB_HOST=localhost
DB_PORT=5432
DB_NAME=cognee_vectors
DB_USERNAME=postgres
DB_PASSWORD=password
```

**Setup**:
```sql
-- Enable pgvector extension
CREATE EXTENSION IF NOT EXISTS vector;
```

**Pros**:
- Production-ready
- ACID compliance
- Integrates with existing PostgreSQL
- Scalable

**Cons**:
- Requires PostgreSQL setup
- Slower than specialized vector DBs

### Neptune Analytics (AWS)
```bash
VECTOR_DB_PROVIDER=neptune
NEPTUNE_ANALYTICS_GRAPH_ID=g-abc123
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=your-key
AWS_SECRET_ACCESS_KEY=your-secret
```

**Pros**:
- Fully managed
- Highly scalable
- Integrated with AWS

**Cons**:
- AWS-only
- Cost considerations

## Graph Database Options

### Kuzu (Default - Embedded)
```bash
GRAPH_DB_PROVIDER=kuzu
# No additional configuration needed
# Data stored in: .cognee_system/databases/cognee_graph_kuzu/
```

**Pros**:
- Embedded, no server needed
- Fast for analytical queries
- Good for development
- Columnar storage

**Cons**:
- Limited to single machine
- Less mature ecosystem

### Neo4j
```bash
GRAPH_DB_PROVIDER=neo4j
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=password
```

**Setup**:
```bash
# Docker setup
docker run -d \
  --name neo4j \
  -p 7687:7687 -p 7474:7474 \
  -e NEO4J_AUTH=neo4j/password \
  neo4j:latest
```

**Pros**:
- Industry standard
- Rich query language (Cypher)
- Visualization tools
- Large ecosystem

**Cons**:
- Resource intensive
- Requires separate server
- Complex configuration

### FalkorDB (Redis-based)
```bash
GRAPH_DB_PROVIDER=falkordb
FALKORDB_HOST=localhost
FALKORDB_PORT=6379
FALKORDB_PASSWORD=optional
```

**Setup**:
```bash
# Docker setup
docker run -d \
  --name falkordb \
  -p 6379:6379 \
  falkordb/falkordb:latest
```

**Pros**:
- Very fast
- Redis-based (familiar)
- Low memory footprint
- Good for real-time

**Cons**:
- Less feature-rich
- Smaller community

## Relational Database Options

### SQLite (Default - Embedded)
```bash
RELATIONAL_DB_PROVIDER=sqlite
# No additional configuration needed
# Data stored in: .cognee_system/databases/cognee.db
```

**Pros**:
- Zero configuration
- Embedded
- Perfect for development
- Portable

**Cons**:
- Limited concurrency
- Not for production scale

### PostgreSQL
```bash
RELATIONAL_DB_PROVIDER=postgres
DB_HOST=localhost
DB_PORT=5432
DB_NAME=cognee_metadata
DB_USERNAME=postgres
DB_PASSWORD=password
```

**Setup**:
```bash
# Docker setup
docker run -d \
  --name postgres \
  -p 5432:5432 \
  -e POSTGRES_PASSWORD=password \
  -e POSTGRES_DB=cognee_metadata \
  postgres:latest

# Run migrations
alembic upgrade head
```

**Pros**:
- Production-ready
- ACID compliance
- Advanced features
- Scalable

**Cons**:
- Requires setup
- More complex

## Storage Configuration

### Local Storage (Default)
```bash
FILE_STORAGE_PROVIDER=local
LOCAL_STORAGE_PATH=./.cognee_system/files
```

### S3 Storage
```bash
FILE_STORAGE_PROVIDER=s3
S3_BUCKET_NAME=cognee-storage
S3_REGION=us-east-1
AWS_ACCESS_KEY_ID=your-key
AWS_SECRET_ACCESS_KEY=your-secret
```

## Common Configurations

### Development Setup (All Embedded)
```bash
# .env.development
LLM_PROVIDER=ollama
LLM_API_BASE=http://localhost:11434
LLM_MODEL=llama3.2:3b

GRAPH_DB_PROVIDER=kuzu
VECTOR_DB_PROVIDER=lancedb
RELATIONAL_DB_PROVIDER=sqlite
FILE_STORAGE_PROVIDER=local
```

### Production Setup (Distributed)
```bash
# .env.production
LLM_PROVIDER=openai
LLM_API_KEY=sk-...
LLM_MODEL=gpt-4

GRAPH_DB_PROVIDER=neo4j
NEO4J_URI=bolt://neo4j.internal:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=${NEO4J_PASSWORD}

VECTOR_DB_PROVIDER=pgvector
DB_HOST=postgres.internal
DB_PORT=5432
DB_NAME=cognee_vectors
DB_USERNAME=cognee
DB_PASSWORD=${DB_PASSWORD}

RELATIONAL_DB_PROVIDER=postgres
FILE_STORAGE_PROVIDER=s3
S3_BUCKET_NAME=cognee-production
```

### Docker Compose Setup
```yaml
version: '3.8'

services:
  cognee:
    image: cognee:latest
    environment:
      - GRAPH_DB_PROVIDER=neo4j
      - NEO4J_URI=bolt://neo4j:7687
      - VECTOR_DB_PROVIDER=chromadb
      - CHROMADB_HOST=chromadb
    depends_on:
      - neo4j
      - chromadb
      - postgres

  neo4j:
    image: neo4j:latest
    ports:
      - "7687:7687"
      - "7474:7474"
    environment:
      - NEO4J_AUTH=neo4j/password

  chromadb:
    image: chromadb/chroma:latest
    ports:
      - "8000:8000"

  postgres:
    image: postgres:latest
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=cognee
```

## Database Migration

### Switching Databases
```python
import cognee

# Export data from current setup
data = await cognee.export_all()

# Change configuration
cognee.config.graph_db_provider = "neo4j"
cognee.config.vector_db_provider = "chromadb"

# Reinitialize
await cognee.init()

# Import data to new setup
await cognee.import_all(data)
```

### Running Alembic Migrations
```bash
# Create new migration
alembic revision --autogenerate -m "Add new table"

# Apply migrations
alembic upgrade head

# Rollback
alembic downgrade -1
```

## Performance Tuning

### Vector Database
```python
# Optimize embedding dimensions
config.embedding_dim = 768  # or 1536 for better accuracy

# Batch size for indexing
config.vector_batch_size = 1000

# Index type for similarity search
config.vector_index_type = "IVF"  # or "HNSW" for better recall
```

### Graph Database
```python
# Connection pool settings
config.graph_connection_pool_size = 10
config.graph_max_connection_lifetime = 3600

# Query optimization
config.graph_query_timeout = 30
config.graph_cache_enabled = True
```

### Relational Database
```python
# Connection pool
config.db_pool_size = 20
config.db_max_overflow = 40

# Query optimization  
config.db_echo = False  # Set True for debugging
config.db_pool_pre_ping = True
```

## Monitoring & Maintenance

### Health Checks
```python
from cognee.infrastructure.databases import (
    get_vector_database,
    get_graph_database, 
    get_relational_database
)

async def check_databases():
    # Check vector DB
    vector_db = await get_vector_database()
    vector_status = await vector_db.health_check()
    
    # Check graph DB
    graph_db = await get_graph_database()
    graph_status = await graph_db.health_check()
    
    # Check relational DB
    relational_db = await get_relational_database()
    relational_status = await relational_db.health_check()
    
    return {
        "vector": vector_status,
        "graph": graph_status,
        "relational": relational_status
    }
```

### Backup Strategies

#### Embedded Databases
```bash
# Backup Kuzu and LanceDB
tar -czf cognee_backup.tar.gz .cognee_system/databases/

# Restore
tar -xzf cognee_backup.tar.gz
```

#### Neo4j
```bash
# Backup
neo4j-admin backup --to=/backup/neo4j-backup

# Restore
neo4j-admin restore --from=/backup/neo4j-backup
```

#### PostgreSQL
```bash
# Backup
pg_dump -U postgres cognee > cognee_backup.sql

# Restore
psql -U postgres cognee < cognee_backup.sql
```

## Troubleshooting

### Common Issues

1. **Connection Refused**
```bash
# Check if services are running
docker ps
lsof -i :7687  # Neo4j
lsof -i :5432  # PostgreSQL
```

2. **Permission Denied**
```bash
# Fix file permissions for embedded DBs
chmod -R 755 .cognee_system/databases/
```

3. **Out of Memory**
```bash
# Increase Docker memory
docker run -m 4g neo4j:latest

# Or adjust batch sizes
VECTOR_BATCH_SIZE=100
GRAPH_BATCH_SIZE=500
```

4. **Slow Queries**
```python
# Enable query logging
config.db_echo = True
config.graph_query_logging = True

# Check indexes
await graph_db.create_indexes()
await vector_db.optimize_index()
```

## Best Practices

1. **Start Simple**: Use embedded databases for development
2. **Test Migrations**: Always test database switches with small datasets
3. **Monitor Performance**: Set up monitoring for production
4. **Regular Backups**: Implement automated backup strategies
5. **Security**: Use strong passwords and encryption for production
6. **Connection Pooling**: Configure appropriate pool sizes
7. **Index Optimization**: Regularly optimize indexes for performance