# Local Log Monitor & Metrics Dashboard

## What is it?
A learning project. Runs locally to watch a folder of *.log files, tails them concurrently, parses events, aggregates metrics in sliding windows, and exposes results via a small HTTP API (and optional TUI).

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

## Non-Functional Requirements

- Concurrency safe, no goroutine leaks (prove with tests + -race). 
- Deterministic core logic (injectable clock for tests). 
- Config via env or YAML; sensible defaults. 
- Clean architecture boundaries; high test coverage on domain/usecases. 
- Runs offline; zero external services.

## Success Criteria

- Start binary → begins watching a folder
- dropping logs increases counters
- `/metrics` returns expected JSON
- stopping via SIGINT shuts down cleanly

## Testing
- 95%+ branch coverage in domain/usecase
- integration tests pass on Linux/macOS.