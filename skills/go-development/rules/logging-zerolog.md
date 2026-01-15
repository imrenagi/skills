---
title: Use Zerolog for Structured Logging
impact: HIGH
impactDescription: Provides high-performance, structured, and context-aware logging
tags: logging, zerolog
---

## Use Zerolog for Structured Logging

**Impact: HIGH**

Use `rs/zerolog` for logging. It is faster than the standard library and provides structured JSON output which is essential for observability. Always pass `context` to loggers when available to carry request-scoped metadata.

**Incorrect (fmt.Println or standard log):**

```go
import "log"

func process(userID string) {
    // Unstructured, hard to parse
    log.Printf("Processing user %s", userID)

    if err := doWork(); err != nil {
        log.Printf("Error: %v", err)
    }
}
```

**Correct (Zerolog with context fields):**

```go
import (
    "github.com/rs/zerolog/log"
)

func process(ctx context.Context, userID string) {
    // Contextual logging
    logger := log.With().Str("user_id", userID).Logger()

    logger.Info().Msg("starting processing")

    if err := doWork(); err != nil {
        // Include error object
        logger.Error().Err(err).Msg("failed to process work")
        return
    }

    logger.Info().Int("items_processed", 10).Msg("processing complete")
}
```

Reference: [Zerolog Documentation](https://github.com/rs/zerolog)
