# Distributed Log System

Secure multi-client distributed log aggregation built with low-level Python sockets and a Flask dashboard.

## Problem Statement

This project collects logs from multiple distributed clients, sends them over the network using TLS-protected sockets, orders them by event time, stores them in SQLite, and displays recent system activity on a dashboard.

## Architecture

```text
+------------------+      TLS over TCP      +---------------------------+
| Log Client(s)    | ---------------------> | Secure Socket Receiver    |
| generate logs    |                        | threaded accept loop      |
+------------------+                        +-------------+-------------+
                                                          |
                                                          v
                                                +---------+---------+
                                                | Shared Ingestion  |
                                                | Queue             |
                                                +---------+---------+
                                                          |
                                                          v
                                                +---------+---------+
                                                | Worker Threads    |
                                                | parse log lines   |
                                                +---------+---------+
                                                          |
                                                          v
                                                +---------+---------+
                                                | Time Ordered      |
                                                | Buffer            |
                                                +---------+---------+
                                                          |
                                                          v
                                                +---------+---------+
                                                | SQLite Storage    |
                                                | logs + metrics    |
                                                +---------+---------+
                                                          |
                                                          v
                                                +---------+---------+
                                                | Flask Dashboard   |
                                                +-------------------+
```

## Protocol Design

- Transport: TCP sockets wrapped with TLS 1.2+
- Message format: `timestamp|server_name|level|message\n`
- Supported levels: `INFO`, `WARNING`, `ERROR`
- One TLS connection can carry many log messages

## Features

- Explicit low-level socket creation, bind, listen, accept, connect, send, receive
- TLS-secured client/server communication
- Multiple concurrent clients using threaded connection handling
- Worker-thread pipeline for parsing and ordering
- Queue overflow protection and parse-error tracking
- SQLite persistence for logs and live metrics
- Dashboard for recent logs and system health
- Load test script for concurrent client evaluation

## Project Structure

- `client/`: secure log generator and client-side configuration
- `server/`: TLS receiver, workers, processor, launcher
- `ordering/`: event-time ordering buffer
- `queue_system/`: bounded ingestion queue
- `storage/`: SQLite database layer and schema
- `metrics/`: throughput and error counters
- `dashboard/`: Flask UI
- `tests/`: concurrent client load test
- `certs/`: self-signed certificate for demo use

## Setup

1. Create and activate a Python virtual environment.
2. Install dependencies:

```bash
pip install -r requirements.txt
```

3. Start the full system:

```bash
python run_project.py
```

4. Open the dashboard at `http://127.0.0.1:5000`

## Running Components Individually

```bash
python server/server_main.py
python client/log_client.py
python dashboard/app.py
```

## Performance Evaluation

Run the included load test while the server is running:

```bash
python tests/load_test.py
```

What to record for submission:

- received logs per second
- processed logs per second
- queue size under load
- dropped logs
- parse errors
- active client count
- TLS handshake failures

Reference measurements already captured in this repo are available in `docs/PERFORMANCE_EVALUATION.md`.

## Security Notes

- The repo includes a self-signed certificate for classroom/demo use.
- Production deployments should replace `certs/server.crt` and `certs/server.key` with properly issued certificates.
- All client/server log traffic is protected with TLS.

## Edge Cases Handled

- abrupt client disconnects
- queue overflow
- malformed log lines
- TLS handshake failures
- partial line reads over TCP
- graceful shutdown with remaining buffered logs flushed

## Submission Checklist

- TLS-based network communication implemented
- Multiple concurrent clients supported
- Low-level sockets used directly
- Performance/load test available
- Documentation, setup, usage, and architecture included
