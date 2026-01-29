# Chapter 12: Capstone Project - Implementation & Presentation

## Learning Objectives
By the end of this chapter, you will be able to:
1. **Write** production-quality code with proper error handling, logging, and documentation
2. **Conduct** effective code reviews that improve code quality while maintaining team morale
3. **Debug** distributed systems using logs, metrics, and traces across multiple services
4. **Present** technical work to diverse audiences with clear explanations and live demonstrations

## Introduction

You've designed your data platform. Infrastructure is provisioned. Now comes the hard part: making it actually work.

Not "work on your laptop." Work in production. At scale. With real data. With things failing. With teammates depending on you.

This chapter is about the difference between code that runs and code you're proud to show at a job interview.

## Implementation Best Practices

### Code Quality Standards

**Every file should have:**

1. **Clear purpose** (docstring at top)
2. **Error handling** (specific exceptions)
3. **Logging** (info, warning, error levels)
4. **Tests** (unit tests minimum)
5. **Type hints** (Python 3.10+)

**Example: Production-quality consumer**

```python
"""
Kafka clickstream consumer that ingests user events to BigQuery.

Handles:
- Message validation and schema enforcement
- Dead letter queue for failed messages
- Graceful shutdown on termination signals
- Backpressure when BigQuery is slow
"""

import logging
import signal
import sys
from typing import Dict, List, Optional
from kafka import KafkaConsumer
from google.cloud import bigquery
from google.api_core import exceptions as gcp_exceptions
import json

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

class ClickstreamConsumer:
    """Consumes clickstream events from Kafka and loads to BigQuery."""

    def __init__(
        self,
        kafka_servers: List[str],
        topic: str,
        bigquery_table: str,
        dlq_table: str
    ):
        """
        Initialize consumer.

        Args:
            kafka_servers: List of Kafka bootstrap servers
            topic: Kafka topic to consume from
            bigquery_table: Target BigQuery table (project.dataset.table)
            dlq_table: Dead letter queue table for failed messages
        """
        self.consumer = KafkaConsumer(
            topic,
            bootstrap_servers=kafka_servers,
            group_id='clickstream-to-bigquery',
            auto_offset_reset='latest',
            enable_auto_commit=False,
            value_deserializer=lambda m: json.loads(m.decode('utf-8'))
        )
        self.bq_client = bigquery.Client()
        self.table_id = bigquery_table
        self.dlq_table_id = dlq_table
        self.running = True

        # Graceful shutdown on SIGTERM
        signal.signal(signal.SIGTERM, self._shutdown)
        signal.signal(signal.SIGINT, self._shutdown)

    def _shutdown(self, signum, frame):
        """Handle shutdown signal."""
        logger.info(f"Received signal {signum}, shutting down gracefully...")
        self.running = False

    def validate_event(self, event: Dict) -> Optional[str]:
        """
        Validate event schema.

        Args:
            event: Event dictionary from Kafka

        Returns:
            Error message if invalid, None if valid
        """
        required_fields = ['event_id', 'event_timestamp', 'event_type', 'user_id']

        for field in required_fields:
            if field not in event:
                return f"Missing required field: {field}"

        # Validate event type
        valid_types = ['page_view', 'add_to_cart', 'purchase', 'search']
        if event['event_type'] not in valid_types:
            return f"Invalid event_type: {event['event_type']}"

        return None  # Valid

    def transform_event(self, event: Dict) -> Dict:
        """
        Transform event to BigQuery schema.

        Args:
            event: Raw event from Kafka

        Returns:
            Transformed event ready for BigQuery
        """
        return {
            'event_id': event['event_id'],
            'event_timestamp': event['event_timestamp'],
            'user_id': event['user_id'],
            'event_type': event['event_type'],
            'page_url': event.get('page_url'),  # Optional
            'product_id': event.get('product_id'),  # Optional
            'ingested_at': datetime.utcnow().isoformat()
        }

    def send_to_dlq(self, event: Dict, error: str):
        """
        Send failed event to dead letter queue.

        Args:
            event: Original event that failed
            error: Error message
        """
        dlq_row = {
            'original_event': json.dumps(event),
            'error': error,
            'failed_at': datetime.utcnow().isoformat(),
            'consumer': 'clickstream-to-bigquery'
        }

        try:
            errors = self.bq_client.insert_rows_json(self.dlq_table_id, [dlq_row])
            if errors:
                logger.error(f"Failed to insert to DLQ: {errors}")
        except gcp_exceptions.GoogleAPIError as e:
            logger.error(f"DLQ insert failed: {e}")

    def process_batch(self, messages: List) -> int:
        """
        Process a batch of messages.

        Args:
            messages: List of Kafka messages

        Returns:
            Number of successfully processed messages
        """
        valid_events = []
        failed_count = 0

        for message in messages:
            event = message.value

            # Validate
            validation_error = self.validate_event(event)
            if validation_error:
                logger.warning(f"Invalid event: {validation_error}")
                self.send_to_dlq(event, validation_error)
                failed_count += 1
                continue

            # Transform
            try:
                transformed = self.transform_event(event)
                valid_events.append(transformed)
            except Exception as e:
                logger.error(f"Transform failed: {e}", exc_info=True)
                self.send_to_dlq(event, str(e))
                failed_count += 1

        # Load to BigQuery
        if valid_events:
            try:
                errors = self.bq_client.insert_rows_json(self.table_id, valid_events)
                if errors:
                    logger.error(f"BigQuery insert errors: {errors}")
                    for i, error in enumerate(errors):
                        self.send_to_dlq(valid_events[i], str(error))
                        failed_count += 1
                else:
                    logger.info(f"Loaded {len(valid_events)} events to BigQuery")
            except gcp_exceptions.GoogleAPIError as e:
                logger.error(f"BigQuery insert failed: {e}", exc_info=True)
                # All events failed, send to DLQ
                for event in valid_events:
                    self.send_to_dlq(event, str(e))
                failed_count += len(valid_events)

        success_count = len(messages) - failed_count
        return success_count

    def run(self, batch_size: int = 100, batch_timeout: float = 5.0):
        """
        Main processing loop.

        Args:
            batch_size: Number of messages to batch before inserting
            batch_timeout: Max seconds to wait for batch to fill
        """
        logger.info("Starting clickstream consumer...")
        batch = []
        last_commit = time.time()

        try:
            for message in self.consumer:
                if not self.running:
                    break

                batch.append(message)

                # Process batch when full or timeout
                should_process = (
                    len(batch) >= batch_size or
                    time.time() - last_commit > batch_timeout
                )

                if should_process:
                    success_count = self.process_batch(batch)
                    logger.info(f"Processed {success_count}/{len(batch)} messages")

                    # Commit offsets
                    self.consumer.commit()
                    last_commit = time.time()
                    batch = []

        except Exception as e:
            logger.error(f"Fatal error in consumer: {e}", exc_info=True)
            raise
        finally:
            logger.info("Shutting down consumer...")
            self.consumer.close()

if __name__ == "__main__":
    consumer = ClickstreamConsumer(
        kafka_servers=['kafka1:9092', 'kafka2:9092'],
        topic='clickstream',
        bigquery_table='ecommerce_prod.fact_events',
        dlq_table='ecommerce_prod.events_dlq'
    )
    consumer.run()
```

**Why this is production-quality:**
- ✅ Full docstrings with type hints
- ✅ Specific error handling (not bare `except:`)
- ✅ Structured logging with levels
- ✅ Graceful shutdown handling
- ✅ Dead letter queue for failures
- ✅ Batch processing for efficiency
- ✅ Validation before transformation
- ✅ Clear variable names and logic flow

### Testing Strategies

**Unit tests (test individual functions):**

```python
# tests/unit/test_consumer.py
import pytest
from streaming.clickstream_consumer import ClickstreamConsumer

def test_validate_event_success():
    """Valid event passes validation"""
    consumer = ClickstreamConsumer([], 'topic', 'table', 'dlq')

    event = {
        'event_id': '123',
        'event_timestamp': '2026-01-29T10:00:00Z',
        'event_type': 'page_view',
        'user_id': 'user-456'
    }

    error = consumer.validate_event(event)
    assert error is None

def test_validate_event_missing_field():
    """Event missing required field fails validation"""
    consumer = ClickstreamConsumer([], 'topic', 'table', 'dlq')

    event = {
        'event_id': '123',
        # Missing event_timestamp
        'event_type': 'page_view',
        'user_id': 'user-456'
    }

    error = consumer.validate_event(event)
    assert 'event_timestamp' in error

def test_transform_event():
    """Event transformation works correctly"""
    consumer = ClickstreamConsumer([], 'topic', 'table', 'dlq')

    event = {
        'event_id': '123',
        'event_timestamp': '2026-01-29T10:00:00Z',
        'event_type': 'purchase',
        'user_id': 'user-456',
        'product_id': 'prod-789'
    }

    result = consumer.transform_event(event)

    assert result['event_id'] == '123'
    assert result['product_id'] == 'prod-789'
    assert 'ingested_at' in result
```

**Integration tests (test full flow):**

```python
# tests/integration/test_consumer_to_bigquery.py
def test_consumer_end_to_end(kafka_producer, bigquery_client):
    """Test event flows from Kafka to BigQuery"""
    # Publish test event
    test_event = {
        'event_id': f'test-{uuid.uuid4()}',
        'event_timestamp': datetime.utcnow().isoformat(),
        'event_type': 'page_view',
        'user_id': 'test-user'
    }
    kafka_producer.send('clickstream', test_event)

    # Start consumer (runs in background thread)
    consumer = ClickstreamConsumer(
        kafka_servers=['localhost:9092'],
        topic='clickstream',
        bigquery_table='test_dataset.events',
        dlq_table='test_dataset.events_dlq'
    )
    consumer_thread = threading.Thread(target=consumer.run)
    consumer_thread.start()

    # Wait for processing
    time.sleep(5)

    # Verify in BigQuery
    query = f"""
        SELECT * FROM test_dataset.events
        WHERE event_id = '{test_event['event_id']}'
    """
    results = list(bigquery_client.query(query).result())

    assert len(results) == 1
    assert results[0].event_type == 'page_view'

    # Cleanup
    consumer.running = False
    consumer_thread.join(timeout=10)
```

## Code Review Process

### Writing Good Pull Requests

**Bad PR:**
```
Title: Fix
Description: (empty)
Files changed: 47 files, +2,847 lines
```

**Good PR:**
```
Title: Add Kafka consumer with DLQ and validation

Description:
Implements clickstream consumer that:
- Reads from Kafka topic 'clickstream'
- Validates event schema (required fields, valid types)
- Transforms to BigQuery schema
- Batches inserts for efficiency
- Sends failed events to DLQ

Testing:
- Unit tests for validation/transformation logic
- Integration test with test Kafka + BigQuery
- Manually tested with 10K events, all processed successfully

Closes #42
```

**PR best practices:**
- Keep it small (<500 lines if possible)
- Clear title and description
- Link to issue/ticket
- Include testing notes
- Add screenshots for UI changes

### Conducting Code Reviews

**What to look for:**

1. **Correctness:** Does it work? Are there bugs?
2. **Tests:** Are there tests? Do they cover edge cases?
3. **Readability:** Can you understand it? Is it documented?
4. **Performance:** Are there obvious bottlenecks?
5. **Security:** Any secrets hardcoded? SQL injection risks?

**How to give feedback:**

**Bad review comment:**
```
This is wrong.
```

**Good review comment:**
```
This could cause a memory leak if the consumer runs for days.

The `batch` list grows unbounded when BigQuery is slow.
Consider adding a max batch size or timeout.

Suggestion:
if len(batch) >= max_batch_size:
    process_batch(batch)
    batch = []
```

**Review etiquette:**
- Assume good intent
- Ask questions, don't demand changes
- Praise good code
- Suggest, don't command
- Explain the "why"

**Example review:**

```
Overall: Nice work! The error handling is solid and the tests are thorough.

src/consumer.py:
Line 45: Consider adding a type hint for `event`
Line 67: What happens if BigQuery is down for >1 hour? Will Kafka consumer lag forever?
  Suggestion: Add a circuit breaker or max retry limit
Line 89: 🎉 Love the structured logging here

tests/test_consumer.py:
Line 23: Could we add a test for invalid event_type? (e.g., 'invalid_type')

Approved pending discussion of circuit breaker for BigQuery failures.
```

## Debugging Distributed Systems

### When Things Go Wrong

**Scenario:** Events aren't appearing in BigQuery

**Debugging process:**

1. **Check the consumer logs**
```bash
docker logs clickstream-consumer --tail 100

# Look for:
# - Connection errors
# - Validation failures
# - BigQuery insert errors
```

2. **Check Kafka consumer lag**
```bash
kafka-consumer-groups --bootstrap-server kafka:9092 \
    --describe --group clickstream-to-bigquery

# Is lag growing? Consumer is falling behind
# Is lag stable? Consumer is keeping up but maybe not processing
```

3. **Check BigQuery streaming inserts**
```sql
SELECT
    DATE(insertion_timestamp) AS date,
    COUNT(*) AS records_inserted
FROM `ecommerce_prod.fact_events`
WHERE insertion_timestamp >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR)
GROUP BY date
ORDER BY date DESC;
```

4. **Check dead letter queue**
```sql
SELECT
    error,
    COUNT(*) AS count
FROM `ecommerce_prod.events_dlq`
WHERE failed_at >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR)
GROUP BY error
ORDER BY count DESC;
```

5. **Check Cloud Monitoring metrics**
- Consumer CPU/memory usage
- Network throughput
- BigQuery quota usage

### Common Issues & Solutions

| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| Consumer lag growing | Can't keep up with event rate | Scale consumers horizontally |
| Events in DLQ | Schema validation failing | Check DLQ errors, fix upstream |
| BigQuery insert errors | Quota exceeded or schema mismatch | Check quotas, validate schema |
| Consumer crashing | OOM or unhandled exception | Check logs, add memory limits |

## Technical Presentations

### Structure (15 minutes)

**Slide 1: Title (30 sec)**
- Project name
- Your name
- Date

**Slide 2: The Problem (2 min)**
- What business need does this solve?
- Why does it matter?
- One concrete example

**Slide 3-4: Architecture (3 min)**
- Show diagram
- Explain data flow left to right
- Highlight key components
- Mention scale (events/sec, data volume)

**Slide 5-7: Live Demo (5 min)**
- Show working system
- Publish test event → appears in dashboard
- Show monitoring (lag, quality metrics)
- Demonstrate graceful failure (DLQ)

**Slide 8-9: Challenges (3 min)**
- What went wrong during development?
- How did you fix it?
- What did you learn?

**Slide 10: What's Next (1 min)**
- Improvements you'd make
- Production readiness gaps
- Lessons for next project

**Slide 11: Questions (1 min)**
- Thank audience
- Open for questions

### Demo Tips

**Before presenting:**
- ✅ Test demo 3 times
- ✅ Have backup screenshots if live demo fails
- ✅ Clear browser history/tabs
- ✅ Zoom in on terminal text (font size 18+)
- ✅ Mute Slack/email notifications

**During demo:**
- Narrate what you're doing ("I'm going to publish an event...")
- Go slow enough for audience to follow
- If something breaks, have Plan B ready

**If demo fails:**
- Don't panic
- Switch to screenshots
- Explain what would have happened
- Continue confidently

## Capstone Rubric

### Technical (60 points)

**Architecture (15 pts)**
- Clear diagram showing all components
- Reasonable technology choices
- Justification for decisions

**Implementation (25 pts)**
- Code works end-to-end
- Streaming latency <5 seconds
- Data quality >99%
- Proper error handling
- Tests passing

**Infrastructure (10 pts)**
- Fully automated with Terraform
- One-command deployment
- All secrets in environment variables

**Monitoring (10 pts)**
- Dashboard showing pipeline health
- Alerts configured
- Runbook documented

### Professional (40 points)

**Collaboration (15 pts)**
- PR process followed
- Code reviews conducted
- Git commits are clear
- Documentation is complete

**Presentation (15 pts)**
- Clear problem statement
- Live demo works (or good backup)
- Technical depth appropriate
- Handles questions well

**Code Quality (10 pts)**
- Readable and documented
- Follows conventions
- No obvious bugs
- Production-ready

## Congratulations!

You've completed the Data Engineering Bootcamp. You've learned:
- Python & SQL at scale
- Git & Docker workflows
- Database design & warehousing
- ETL orchestration with Airflow
- Data quality & error handling
- Distributed processing with Spark
- Real-time streaming with Kafka
- Cloud platforms (GCP)
- Infrastructure as Code
- CI/CD for data pipelines

Most importantly, you've learned from 30+ war stories about production disasters. Now go build systems that don't create new ones.

Welcome to data engineering. Try not to take down production on your first day.

(But when you do—and you will—you'll know how to fix it.)
