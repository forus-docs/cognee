# Cognee Pipeline Architecture

## Overview

Cognee's pipeline system is a flexible, async-first architecture for processing data through a series of composable tasks. The system follows the ECL (Extract-Cognify-Load) pattern with support for parallel execution, error handling, and progress tracking.

## Core Concepts

### Pipeline
A sequence of tasks that process data from input to output. Each pipeline has:
- Unique ID for tracking
- Status (pending, processing, completed, failed)
- Progress metrics
- Error handling

### Task
An atomic unit of work that:
- Takes input data
- Performs processing
- Produces output for next task
- Can run async or sync
- Supports batching

### Pipeline Run
An execution instance of a pipeline with:
- Run ID for tracking
- Dataset association
- Task execution history
- Telemetry data

## Pipeline Types

### 1. Cognify Pipeline
Main pipeline for transforming text into knowledge graphs.

```python
# Pipeline stages
cognify_pipeline = [
    classify_documents,
    check_permissions_on_dataset,
    extract_chunks_from_documents,
    extract_graph_from_data,
    summarize_text,
    add_data_points
]
```

### 2. Code Graph Pipeline
Specialized pipeline for analyzing source code.

```python
# Pipeline stages
code_pipeline = [
    get_repo_file_dependencies,
    extract_code_parts,
    build_dependency_graph,
    add_data_points
]
```

### 3. Custom Pipelines
User-defined pipelines for specific workflows.

```python
from cognee.pipelines import Pipeline, Task

custom_pipeline = Pipeline([
    Task(preprocess_data),
    Task(custom_extraction),
    Task(post_process)
])
```

## Task Architecture

### Task Types

#### 1. Coroutine Tasks
Single async operations:
```python
async def extract_entities(data):
    # Process data
    return entities
```

#### 2. Generator Tasks
Streaming/batched operations:
```python
async def process_chunks(data):
    for chunk in data:
        processed = await process(chunk)
        yield processed
```

#### 3. Parallel Tasks
Concurrent execution:
```python
from cognee.pipelines import run_parallel

tasks = [task1, task2, task3]
results = await run_parallel(tasks, data)
```

### Task Interface
```python
class Task:
    async def execute(self, data, context):
        """Execute task logic"""
        pass
    
    async def validate(self, data):
        """Validate input data"""
        pass
    
    async def rollback(self, error):
        """Handle failures"""
        pass
```

## Pipeline Execution Flow

### 1. Initialization
```python
# Create pipeline run
pipeline_run = PipelineRun(
    id=generate_uuid(),
    pipeline_id=pipeline.id,
    dataset_id=dataset.id,
    status=PipelineStatus.PENDING
)
```

### 2. Task Execution
```python
# Sequential execution
for task in pipeline.tasks:
    result = await task.execute(data, context)
    data = result  # Pass to next task
    
# Parallel execution
results = await run_parallel(
    parallel_tasks,
    data,
    max_workers=4
)
```

### 3. Progress Tracking
```python
# Update progress
await log_pipeline_progress(
    pipeline_run_id,
    current_task="extracting entities",
    progress=0.5
)
```

### 4. Error Handling
```python
try:
    result = await task.execute(data)
except Exception as e:
    await log_pipeline_error(pipeline_run_id, e)
    await task.rollback(e)
    raise
```

## Key Pipeline Tasks

### Document Processing

#### classify_documents
```python
async def classify_documents(documents):
    """Classify document types (PDF, text, image, etc.)"""
    classifications = []
    for doc in documents:
        doc_type = await identify_type(doc)
        classifications.append({
            "document": doc,
            "type": doc_type,
            "loader": get_loader(doc_type)
        })
    return classifications
```

#### extract_chunks_from_documents
```python
async def extract_chunks_from_documents(documents):
    """Chunk documents into semantic units"""
    chunks = []
    for doc in documents:
        text = await extract_text(doc)
        doc_chunks = await chunk_text(
            text,
            method="paragraph",  # or sentence, word
            size=512,
            overlap=50
        )
        chunks.extend(doc_chunks)
    return chunks
```

### Graph Building

#### extract_graph_from_data
```python
async def extract_graph_from_data(chunks):
    """Extract entities and relationships"""
    graph = Graph()
    
    for chunk in chunks:
        # Extract entities
        entities = await llm_extract_entities(chunk)
        graph.add_nodes(entities)
        
        # Infer relationships
        relationships = await llm_infer_relationships(
            chunk, entities
        )
        graph.add_edges(relationships)
    
    return graph
```

#### infer_data_ontology
```python
async def infer_data_ontology(graph):
    """Generate ontology from graph structure"""
    ontology = Ontology()
    
    # Analyze entity types
    entity_types = graph.get_unique_node_types()
    ontology.add_classes(entity_types)
    
    # Analyze relationship types
    rel_types = graph.get_unique_edge_types()
    ontology.add_properties(rel_types)
    
    return ontology
```

### Data Storage

#### add_data_points
```python
async def add_data_points(data):
    """Store processed data in databases"""
    # Store embeddings in vector DB
    vector_db = await get_vector_database()
    await vector_db.add_embeddings(
        data.embeddings,
        metadata=data.metadata
    )
    
    # Store graph in graph DB
    graph_db = await get_graph_database()
    await graph_db.add_nodes(data.nodes)
    await graph_db.add_edges(data.edges)
    
    # Store metadata in relational DB
    relational_db = await get_relational_database()
    await relational_db.save_metadata(data.metadata)
```

## Pipeline Configuration

### Environment Variables
```bash
# Pipeline settings
PIPELINE_MAX_WORKERS=4
PIPELINE_BATCH_SIZE=100
PIPELINE_TIMEOUT=3600
PIPELINE_RETRY_COUNT=3
PIPELINE_RETRY_DELAY=5
```

### Python Configuration
```python
from cognee.config import PipelineConfig

config = PipelineConfig(
    max_workers=4,
    batch_size=100,
    timeout=3600,
    enable_telemetry=True,
    enable_caching=True
)
```

## Custom Pipeline Creation

### Basic Custom Pipeline
```python
from cognee.pipelines import Pipeline, Task

# Define custom tasks
async def custom_preprocessing(data):
    # Your preprocessing logic
    return processed_data

async def custom_analysis(data):
    # Your analysis logic
    return analyzed_data

# Create pipeline
my_pipeline = Pipeline([
    Task(custom_preprocessing),
    Task(custom_analysis)
])

# Execute pipeline
result = await my_pipeline.run(input_data)
```

### Advanced Pipeline with Branching
```python
from cognee.pipelines import Pipeline, ConditionalTask

async def route_by_type(data):
    if data.type == "text":
        return "text_branch"
    elif data.type == "code":
        return "code_branch"
    else:
        return "default_branch"

pipeline = Pipeline([
    Task(classify_data),
    ConditionalTask(
        router=route_by_type,
        branches={
            "text_branch": [process_text, summarize],
            "code_branch": [parse_code, analyze_deps],
            "default_branch": [generic_process]
        }
    ),
    Task(store_results)
])
```

## Pipeline Monitoring

### Status Tracking
```python
async def get_pipeline_status(pipeline_run_id):
    """Get current pipeline status"""
    run = await get_pipeline_run(pipeline_run_id)
    return {
        "id": run.id,
        "status": run.status,
        "progress": run.progress,
        "current_task": run.current_task,
        "started_at": run.started_at,
        "completed_at": run.completed_at,
        "error": run.error
    }
```

### Progress Updates
```python
# Real-time progress via WebSocket
async def pipeline_progress_stream(pipeline_run_id):
    async for update in watch_pipeline(pipeline_run_id):
        yield {
            "type": "progress",
            "progress": update.progress,
            "message": update.message
        }
```

### Metrics Collection
```python
async def get_pipeline_metrics(pipeline_run_id):
    """Get pipeline execution metrics"""
    metrics = await calculate_metrics(pipeline_run_id)
    return {
        "total_time": metrics.total_time,
        "task_times": metrics.task_times,
        "data_processed": metrics.data_volume,
        "errors": metrics.error_count,
        "retries": metrics.retry_count
    }
```

## Error Handling & Recovery

### Retry Logic
```python
from cognee.pipelines import RetryPolicy

retry_policy = RetryPolicy(
    max_attempts=3,
    backoff_factor=2,
    max_delay=60,
    retryable_errors=[TimeoutError, ConnectionError]
)

pipeline = Pipeline(tasks, retry_policy=retry_policy)
```

### Checkpointing
```python
# Save pipeline state for recovery
async def checkpoint_pipeline(pipeline_run, data):
    await save_checkpoint({
        "pipeline_run_id": pipeline_run.id,
        "current_task": pipeline_run.current_task,
        "data": data,
        "timestamp": datetime.now()
    })

# Resume from checkpoint
async def resume_pipeline(pipeline_run_id):
    checkpoint = await load_checkpoint(pipeline_run_id)
    return await pipeline.resume(
        checkpoint.current_task,
        checkpoint.data
    )
```

### Error Recovery
```python
async def handle_pipeline_error(pipeline_run, error):
    """Handle pipeline failures"""
    # Log error
    await log_error(pipeline_run.id, error)
    
    # Attempt recovery
    if isinstance(error, RecoverableError):
        await retry_task(pipeline_run.current_task)
    else:
        # Rollback changes
        await rollback_pipeline(pipeline_run)
        
        # Notify user
        await notify_failure(pipeline_run, error)
```

## Performance Optimization

### Batching
```python
# Configure batch processing
async def process_in_batches(data, batch_size=100):
    for i in range(0, len(data), batch_size):
        batch = data[i:i+batch_size]
        yield await process_batch(batch)
```

### Caching
```python
from functools import lru_cache

@lru_cache(maxsize=1000)
async def cached_extraction(text_hash):
    """Cache expensive extractions"""
    return await extract_entities(text_hash)
```

### Parallel Processing
```python
# Use asyncio for I/O-bound tasks
async def parallel_io_tasks(tasks):
    return await asyncio.gather(*tasks)

# Use multiprocessing for CPU-bound tasks
from concurrent.futures import ProcessPoolExecutor

async def parallel_cpu_tasks(tasks):
    with ProcessPoolExecutor() as executor:
        loop = asyncio.get_event_loop()
        futures = [
            loop.run_in_executor(executor, task)
            for task in tasks
        ]
        return await asyncio.gather(*futures)
```

## Pipeline Testing

### Unit Testing Tasks
```python
import pytest

@pytest.mark.asyncio
async def test_extraction_task():
    # Arrange
    test_data = "Sample text for testing"
    
    # Act
    result = await extract_entities(test_data)
    
    # Assert
    assert len(result) > 0
    assert all(e.type in ["Person", "Organization"] 
              for e in result)
```

### Integration Testing Pipelines
```python
@pytest.mark.asyncio
async def test_full_pipeline():
    # Create test pipeline
    pipeline = create_test_pipeline()
    
    # Run with test data
    result = await pipeline.run(test_documents)
    
    # Verify results
    assert result.status == "completed"
    assert result.graph.node_count > 0
    assert result.graph.edge_count > 0
```

## Best Practices

1. **Keep Tasks Atomic**: Each task should do one thing well
2. **Use Async/Await**: Leverage async for I/O operations
3. **Implement Validation**: Validate data between tasks
4. **Add Telemetry**: Track execution metrics
5. **Handle Errors Gracefully**: Implement proper error recovery
6. **Optimize Batching**: Balance memory vs performance
7. **Cache Expensive Operations**: Avoid redundant processing
8. **Test Thoroughly**: Unit and integration test pipelines
9. **Monitor Performance**: Track bottlenecks and optimize
10. **Document Custom Pipelines**: Clear documentation for maintenance