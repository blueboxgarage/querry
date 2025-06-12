# PostgreSQL Query Generator API - Rust Implementation

**Claude Code Instructions for Building a Production-Ready Natural Language to SQL API Service**

## 🎯 Project Overview

Build a high-performance, production-scalable REST API service in Rust using Axum that converts natural language descriptions into PostgreSQL queries using a CSV-based field mapping system. This service will allow non-technical users to query databases using plain English.

## 📋 Core Requirements

### **Primary Functionality**
- Parse CSV file containing database field mappings
- Accept natural language descriptions via HTTP POST
- Generate PostgreSQL queries using fuzzy field matching
- Return structured JSON responses with generated SQL and confidence scores
- Support multiple field mapping systems (system_a, system_b, default)
- Production-ready with proper error handling, logging, and metrics

### **CSV Field Mapping Format**
```csv
column_name,table_name,system_a_fieldmap,system_b_fieldmap,field_description,field_type
user_id,users,uid,user_identifier,Unique identifier for user,INTEGER
email,users,email_addr,user_email,User email address,VARCHAR
created_at,users,create_date,registration_date,Account creation timestamp,TIMESTAMP
order_total,orders,amount,total_cost,Total order value in cents,INTEGER
order_status,orders,status,order_state,Current status of order,VARCHAR
```

## 🔧 Technical Specifications

### **Tech Stack**
- **Language**: Rust (latest stable)
- **Web Framework**: Axum 0.7+ for high-performance HTTP handling
- **Async Runtime**: Tokio for production-grade concurrency
- **Serialization**: Serde for JSON handling
- **CSV Parsing**: csv crate for robust CSV processing
- **String Matching**: fuzzy-matcher or similar for field matching
- **Logging**: tracing + tracing-subscriber for structured logging
- **Database**: Optional sqlx for PostgreSQL integration

### **Key Dependencies (Cargo.toml)**
```toml
[dependencies]
tokio = { version = "1.0", features = ["full"] }
axum = "0.7"
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
csv = "1.3"
tower = "0.4"
tower-http = { version = "0.5", features = ["cors", "trace"] }
fuzzy-matcher = "0.3"
anyhow = "1.0"
thiserror = "1.0"
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter"] }
uuid = { version = "1.0", features = ["v4"] }
```

### **API Endpoints**
1. **POST /api/v1/generate-query** - Main query generation endpoint
2. **GET /health** - Health check endpoint  
3. **GET /api/v1/fields** - List available field mappings
4. **GET /metrics** - Prometheus metrics (optional)

### **Request/Response Format**

**POST /api/v1/generate-query**
```json
{
  "description": "get all active users with email addresses from last 30 days",
  "system": "system_a",
  "limit": 100,
  "request_id": "optional-trace-id"
}
```

**Response**
```json
{
  "query": "SELECT uid, email_addr, create_date\nFROM users\nWHERE status = 'active' AND create_date >= CURRENT_DATE - INTERVAL '30 days'\nLIMIT 100;",
  "matched_fields": [
    {
      "column_name": "uid",
      "table_name": "users", 
      "field_description": "Unique identifier for user",
      "field_type": "INTEGER",
      "match_score": 85.5,
      "matched_text": "user identifier"
    }
  ],
  "confidence": 0.855,
  "request_id": "abc-123",
  "processing_time_ms": 23
}
```

## 🏗 Architecture Requirements

### **Project Structure**
```
query-generator-api/
├── Cargo.toml
├── src/
│   ├── main.rs               # Entry point, server setup
│   ├── lib.rs                # Library exports
│   ├── config.rs             # Configuration management
│   ├── error.rs              # Error types and handling
│   ├── handlers/             # HTTP request handlers
│   │   ├── mod.rs
│   │   ├── health.rs
│   │   ├── query.rs
│   │   └── fields.rs
│   ├── models/               # Data structures
│   │   ├── mod.rs
│   │   ├── field_mapping.rs
│   │   ├── query_request.rs
│   │   └── query_response.rs
│   ├── services/             # Business logic
│   │   ├── mod.rs
│   │   ├── field_matcher.rs
│   │   ├── nlp_processor.rs
│   │   └── query_generator.rs
│   └── utils/                # Utilities
│       ├── mod.rs
│       └── csv_loader.rs
├── field_mappings.csv        # Sample field mapping data
├── tests/                    # Integration tests
│   ├── common/
│   └── integration_tests.rs
└── README.md
```

### **Core Modules to Implement**

#### **1. Configuration (`config.rs`)**
```rust
#[derive(Debug, Clone)]
pub struct Config {
    pub server_port: u16,
    pub csv_file_path: String,
    pub max_request_size: usize,
    pub fuzzy_match_threshold: f64,
    pub max_matched_fields: usize,
    pub log_level: String,
}
```

#### **2. Error Handling (`error.rs`)**
```rust
#[derive(thiserror::Error, Debug)]
pub enum ApiError {
    #[error("CSV parsing error: {0}")]
    CsvError(#[from] csv::Error),
    #[error("No relevant fields found")]
    NoFieldsFound,
    #[error("Invalid request: {0}")]
    InvalidRequest(String),
    #[error("Internal server error: {0}")]
    Internal(#[from] anyhow::Error),
}
```

#### **3. Field Matcher (`services/field_matcher.rs`)**
- Load field mappings from CSV using csv crate
- Implement fuzzy string matching using fuzzy-matcher
- Extract keywords from natural language input
- Score and rank field matches with confidence scores
- Support system-specific field name resolution
- Cache parsed mappings in memory

#### **4. NLP Processor (`services/nlp_processor.rs`)**
- Extract keywords with regex and stop word filtering
- Detect query intent (SELECT, COUNT, GROUP BY, etc.)
- Identify temporal patterns with regex ("last 30 days", "recent")
- Recognize filter patterns ("active", "status", conditions)
- Implement confidence scoring algorithm

#### **5. Query Generator (`services/query_generator.rs`)**
- Generate SELECT queries for general data retrieval
- Generate COUNT/aggregate queries for "how many" questions
- Generate GROUP BY queries for categorization
- Build WHERE clauses based on detected patterns
- Handle LIMIT clauses and suggest JOINs
- SQL injection prevention with parameterized queries

#### **6. HTTP Handlers (`handlers/`)**
- Async request handlers using Axum extractors
- JSON request/response serialization with Serde
- Proper HTTP status codes and error responses
- Request tracing and logging
- Input validation and sanitization

## 🎯 Implementation Priorities

### **Phase 1: Foundation (Start Here)**
1. Set up Cargo.toml with all required dependencies
2. Implement basic Axum server with health check endpoint
3. Create configuration management with environment variables
4. Set up structured logging with tracing
5. Define core data structures and error types

### **Phase 2: CSV and Basic Matching**
1. Implement CSV parser with robust error handling
2. Create field mapping storage and retrieval
3. Add basic string matching (contains/substring)
4. Create `/api/v1/fields` endpoint
5. Add basic `/api/v1/generate-query` with hardcoded responses

### **Phase 3: Intelligence and Query Generation**
1. Implement fuzzy string matching with scoring
2. Add keyword extraction and NLP processing
3. Create query generation for SELECT statements
4. Add intent detection for different query types
5. Implement confidence scoring system

### **Phase 4: Production Features**
1. Add comprehensive error handling and validation
2. Implement proper logging and metrics
3. Add request tracing and performance monitoring
4. Optimize memory usage and response times
5. Add integration tests and documentation

## 🧠 Algorithm Guidelines

### **Fuzzy Matching with fuzzy-matcher**
```rust
use fuzzy_matcher::FuzzyMatcher;
use fuzzy_matcher::skim::SkimMatcherV2;

let matcher = SkimMatcherV2::default();
if let Some((score, _)) = matcher.fuzzy_match(field_description, query) {
    // score is i64, normalize to 0-100 range
}
```

### **Intent Detection Patterns**
- **COUNT**: regex for "how many", "count", "total number"
- **GROUP BY**: "by category", "group by", "breakdown by"
- **SELECT**: Default for general queries
- **AGGREGATE**: "sum", "average", "max", "min"
- **FILTER**: temporal and status patterns

### **Query Generation Strategy**
```rust
pub enum QueryIntent {
    Select { columns: Vec<String> },
    Count { conditions: Vec<String> },
    GroupBy { group_column: String },
    Aggregate { function: String, column: String },
}
```

## 📊 Performance Requirements

- **Startup Time**: < 200ms (CSV loading + server start)
- **Query Generation**: < 50ms per request (95th percentile)
- **Memory Usage**: < 100MB for typical field mapping files
- **Throughput**: 1000+ requests/second under load
- **Concurrent Connections**: Support 10,000+ concurrent connections

## 🔒 Production Requirements

### **Error Handling**
- Graceful error responses with proper HTTP status codes
- Request ID tracing for debugging
- Structured error logging
- Input validation and sanitization

### **Observability**
- Structured logging with request tracing
- Metrics collection (request count, latency, errors)
- Health checks with dependency validation
- Performance monitoring

### **Security**
- Input sanitization to prevent injection attacks
- Rate limiting (implement with tower-governor)
- CORS configuration for web clients
- Request size limits

## 🧪 Testing Strategy

### **Unit Tests**
```rust
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_csv_parsing() { /* ... */ }
    
    #[test]
    fn test_field_matching() { /* ... */ }
    
    #[test]
    fn test_query_generation() { /* ... */ }
}
```

### **Integration Tests**
- Full HTTP endpoint testing with test client
- CSV loading with various file formats
- Error handling scenarios
- Performance benchmarks

## 🔍 Example Use Cases to Support

1. **Simple Selection**: "get user emails" → `SELECT email FROM users;`
2. **Filtered Data**: "active users from last week" → `SELECT * FROM users WHERE status = 'active' AND created_at >= CURRENT_DATE - INTERVAL '7 days';`
3. **Counting**: "how many orders were placed" → `SELECT COUNT(*) FROM orders;`
4. **Grouping**: "orders by status" → `SELECT order_status, COUNT(*) FROM orders GROUP BY order_status;`
5. **Aggregation**: "average order value" → `SELECT AVG(order_total) FROM orders;`
6. **Multi-system**: Use system_a field mappings when system="system_a"

## 🚀 Getting Started Commands

```bash
# Create new Rust project
cargo new query-generator-api
cd query-generator-api

# Add dependencies to Cargo.toml
# (Use the dependencies list from above)

# Run in development
cargo run

# Run tests
cargo test

# Run with logging
RUST_LOG=debug cargo run

# Build for production
cargo build --release
```

## 💡 Implementation Notes

### **Axum Best Practices**
- Use `State` for shared application data
- Implement proper extractors for request validation
- Use middleware for cross-cutting concerns (logging, CORS)
- Handle errors with custom error types that implement `IntoResponse`

### **Memory Management**
- Use `Arc` for shared immutable data (field mappings)
- Consider `RwLock` if mappings need dynamic updates
- Use streaming for large CSV files if needed
- Implement connection pooling for database access

### **Configuration**
```rust
// Environment-based configuration
pub fn load_config() -> Result<Config> {
    Ok(Config {
        server_port: env::var("PORT")?.parse().unwrap_or(3000),
        csv_file_path: env::var("CSV_FILE_PATH")
            .unwrap_or_else(|_| "field_mappings.csv".to_string()),
        // ... other config
    })
}
```

## 🎯 Success Criteria

The implementation is successful when:
- ✅ Loads CSV field mappings on startup with error handling
- ✅ Serves HTTP requests with proper async handling via Axum
- ✅ Generates valid PostgreSQL queries from natural language
- ✅ Returns structured JSON with confidence scores and timing
- ✅ Handles errors gracefully with appropriate HTTP status codes
- ✅ Supports multiple field mapping systems
- ✅ Processes 95% of requests in under 50ms
- ✅ Handles 1000+ concurrent requests
- ✅ Has comprehensive logging and observability
- ✅ Includes integration tests and documentation

## 🔧 Deployment Considerations

### **Docker Support**
```dockerfile
FROM rust:1.75 as builder
WORKDIR /app
COPY . .
RUN cargo build --release

FROM debian:bookworm-slim
RUN apt-get update && apt-get install -y ca-certificates
COPY --from=builder /app/target/release/query-generator-api /usr/local/bin/
EXPOSE 3000
CMD ["query-generator-api"]
```

### **Environment Variables**
```bash
PORT=3000
CSV_FILE_PATH=/app/field_mappings.csv
RUST_LOG=info
MAX_REQUEST_SIZE=1048576
FUZZY_THRESHOLD=30.0
```

**Start with the Axum foundation and build incrementally. Focus on getting the HTTP server and basic CSV loading working first, then add intelligence layer by layer. Axum's excellent async performance will handle production traffic much better than the Zig std.http implementation.**
