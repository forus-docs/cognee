# Cognee Prune Tool Documentation

## Overview

The prune tool in Cognee provides functionality to completely reset the knowledge graph system by removing all stored data and clearing all databases. This is a powerful tool for starting fresh or cleaning up after testing.

## Architecture

The prune functionality is split into two main components:

1. **prune_data()** - Removes all stored data files
2. **prune_system()** - Clears all database systems

## Components

### 1. Main API Interface (`cognee/api/v1/prune/prune.py`)

The main prune class provides static methods for data and system pruning:

```python
class prune:
    @staticmethod
    async def prune_data():
        """Remove all stored data files"""
        await _prune_data()
    
    @staticmethod
    async def prune_system(graph=True, vector=True, metadata=False):
        """Clear system databases"""
        await _prune_system(graph, vector, metadata)
```

### 2. Data Pruning (`cognee/modules/data/deletion/prune_data.py`)

The data pruning function removes all files from the configured storage location:

```python
async def prune_data():
    storage_config = get_storage_config()
    data_root_directory = storage_config["data_root_directory"]
    await get_file_storage(data_root_directory).remove_all()
```

**What it does:**
- Gets the storage configuration
- Identifies the data root directory
- Removes all files in that directory
- Works with both local and S3 storage backends

### 3. System Pruning (`cognee/modules/data/deletion/prune_system.py`)

The system pruning function clears the three database systems:

```python
async def prune_system(graph=True, vector=True, metadata=False):
    if graph:
        graph_engine = await get_graph_engine()
        await graph_engine.delete_graph()
    
    if vector:
        vector_engine = get_vector_engine()
        await vector_engine.prune()
    
    if metadata:
        db_engine = get_relational_engine()
        await db_engine.delete_database()
```

**Parameters:**
- `graph` (bool, default=True): Clear the graph database
- `vector` (bool, default=True): Clear the vector database
- `metadata` (bool, default=False): Clear the relational metadata database

## Usage

### Python API

```python
import cognee
from cognee.api.v1.prune import prune

# Complete reset - remove all data and clear all databases
await prune.prune_data()
await prune.prune_system(metadata=True)

# Selective pruning - only clear specific databases
await prune.prune_system(graph=True, vector=False, metadata=False)
```

### MCP Tool

```python
# Using the MCP prune tool
await mcp.prune()
# This executes both prune_data() and prune_system() internally
```

### API Endpoint

```http
POST /api/v1/prune
Content-Type: application/json

{
  "confirm": true
}
```

## What Gets Deleted

### When running `prune_data()`:
- All uploaded files
- All text data
- All processed documents
- All temporary files
- Complete contents of data storage directory

### When running `prune_system()`:

#### Graph Database (if graph=True):
- All nodes (entities)
- All edges (relationships)
- All graph metadata
- Ontology information
- Entity classifications

#### Vector Database (if vector=True):
- All embeddings
- All text chunks
- All vector indices
- Similarity search data
- Document vectors

#### Relational Database (if metadata=False by default):
- User data (if metadata=True)
- Dataset information (if metadata=True)
- Pipeline run history (if metadata=True)
- System configuration (if metadata=True)
- Permission settings (if metadata=True)

## Warning and Best Practices

### ⚠️ CRITICAL WARNINGS

1. **Irreversible Operation**: Prune operations cannot be undone
2. **Complete Data Loss**: All knowledge graph data will be permanently deleted
3. **No Backup**: Ensure you have backups before pruning if data is valuable
4. **System-Wide Impact**: Affects all users in multi-tenant setups

### Best Practices

1. **Development Environment**:
   ```python
   # Safe for development - complete reset
   await prune.prune_data()
   await prune.prune_system(metadata=True)
   ```

2. **Testing**:
   ```python
   # Between test runs - clear data but keep config
   await prune.prune_data()
   await prune.prune_system(graph=True, vector=True, metadata=False)
   ```

3. **Production** (NOT RECOMMENDED):
   ```python
   # DANGER: Only with explicit user confirmation
   if user_confirmed and admin_approved:
       await prune.prune_data()
       await prune.prune_system()
   ```

4. **Selective Cleanup**:
   ```python
   # Clear only vector embeddings
   await prune.prune_system(graph=False, vector=True, metadata=False)
   
   # Clear only graph relationships
   await prune.prune_system(graph=True, vector=False, metadata=False)
   ```

## Common Use Cases

### 1. Fresh Start After Configuration Change
```python
# Changed database providers
await prune.prune_data()
await prune.prune_system(metadata=True)
# Reconfigure
cognee.config.vector_db_provider = "chromadb"
# Start fresh
await cognee.add(data)
await cognee.cognify()
```

### 2. Clear Test Data
```python
# After running tests
await prune.prune_data()
await prune.prune_system()
```

### 3. Reset After Failed Processing
```python
# If cognify fails with corrupted data
await prune.prune_data()
await prune.prune_system(graph=True, vector=True)
# Retry with clean state
```

### 4. Memory Management
```python
# When running out of disk space
await prune.prune_data()  # Clear files
await prune.prune_system(vector=True)  # Clear embeddings
```

## Implementation Details

### Storage Backends

The prune tool works with different storage backends:

1. **Local Storage**:
   - Deletes files from local filesystem
   - Path: `.cognee_system/files/`

2. **S3 Storage**:
   - Removes objects from S3 bucket
   - Uses batch delete operations
   - Respects bucket policies

### Database Backends

Different database providers implement pruning differently:

1. **Kuzu (Graph)**:
   - Deletes database directory
   - Recreates empty database

2. **Neo4j (Graph)**:
   - Executes `MATCH (n) DETACH DELETE n`
   - Clears all nodes and relationships

3. **LanceDB (Vector)**:
   - Drops all tables
   - Clears index files

4. **ChromaDB (Vector)**:
   - Deletes collection
   - Recreates empty collection

5. **SQLite (Relational)**:
   - Drops all tables
   - Recreates schema

6. **PostgreSQL (Relational)**:
   - Truncates all tables
   - Resets sequences

## Error Handling

The prune operations handle various error scenarios:

```python
try:
    await prune.prune_data()
except StorageError as e:
    # Handle storage access issues
    logger.error(f"Failed to prune data: {e}")

try:
    await prune.prune_system()
except DatabaseError as e:
    # Handle database connection issues
    logger.error(f"Failed to prune system: {e}")
```

## Related Tools and Commands

- `cognee.delete()` - Delete specific data items
- `cognee.list_data()` - List data before pruning
- Database-specific cleanup commands
- Backup utilities before pruning

## Troubleshooting

### Common Issues

1. **Permission Denied**:
   - Check file system permissions
   - Verify database user privileges

2. **Database Lock**:
   - Ensure no active connections
   - Close all cognify processes

3. **Partial Pruning**:
   - If prune fails midway, run again
   - Check logs for specific errors

4. **Storage Not Cleared**:
   - Verify storage configuration
   - Check S3 bucket permissions

## Summary

The prune tool is a powerful utility for completely resetting Cognee's knowledge graph system. It provides granular control over what to delete, supports multiple storage and database backends, and is essential for testing and development workflows. However, it should be used with extreme caution in production environments as all operations are irreversible.