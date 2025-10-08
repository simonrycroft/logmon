# Local Log Monitor & Metrics Dashboard

## What is it?
The aim is to build a local-only Go app to watch a folder of *.log files, tail them concurrently, parse events, aggregate metrics over sliding windows, and expose results via a small HTTP API (and optional TUI).

Designed to practice goroutines/channels with clean architecture and comprehensive testing.

## Why?
Real-world use of:
- goroutines/channels
- fan-in/out
- worker pools
- backpressure
- cancellation
- graceful shutdown

## Primary tech
- Go 1.25+
- fsnotify (or similar)
- stdlib HTTP
- context
- testing/quick fuzz
- race detector

## Glossary
- Tail: Continuously read new lines appended to a file. 
- Sliding window: Metrics computed over a moving time range (e.g., last 1/5/15 minutes). 
- Fan-out / Fan-in: Broadcast one input to many workers (out) and merge many outputs to one stream (in). 
- Backpressure: Mechanism to avoid overwhelming consumers (bounded channels, drop strategies). 
- Dead letter: Invalid/parsing-failed messages collected for inspection/metrics.

## Project Brief

### Goal
Robust, cancellable worker system with backpressure and fault isolation, mirroring real log pipelines but running locally.

### Non-Functional Requirements
- Concurrency-safe; no goroutine/file-descriptor leaks (prove with tests + -race). 
- Deterministic core logic via injected clock. 
- Config via env/YAML with sensible defaults. 
- Clean architecture boundaries; high test coverage on domain/usecases. 
- Offline; zero external services.

### Success Criteria

Start binary → watches a folder; dropping logs increases counters; /metrics returns expected JSON; SIGINT stops cleanly.

95%+ branch coverage in domain/usecase; integration tests pass on Linux/macOS.

## Architecture
```
/internal
  /domain # pure entities & logic (LogLine, Window, Alert)
  /usecase # orchestrators (pipeline, aggregator, shutdown)
  /adapters # fs watcher, tailer, parsers, http ui
  /interfaces # boundaries: Watcher, Tailer, Parser, MetricsStore
/cmd/logmon # composition root (DI, config)
/pkg/clock # real/fake clock
/pkg/log # logger wrapper
```

## Channel Topology (high-level)
```
[Watcher] -> fileEvents -> [Tailer per file] -> rawLines (bounded)
-> fan-out to parser workers (N) -> parsedLines (fan-in)
-> [Aggregator + Rules] -> metricsOut, alertsOut -> [HTTP/TUI]
```
