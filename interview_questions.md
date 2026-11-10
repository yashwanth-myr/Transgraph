# POC_MAF_X Interview Questions

## Project Overview
POC_MAF_X is a multi-agent system built with LangGraph that handles commodity market data analysis, user management, and document processing. The system uses intelligent routing to direct queries to specialized agents and supports data visualization and report generation.

---

## Technical Architecture Questions

### 1. System Architecture
**Question:** Can you explain the overall architecture of this multi-agent system and how the different components interact?

**Expected Answer:** The system uses LangGraph's StateGraph to orchestrate multiple specialized agents:
- **Supervisor Agent**: Routes queries to appropriate agents based on intent classification
- **MySQL Agent**: Handles user profiles and subscription data
- **MSSQL Agent**: Manages commodity pricing data and historical records
- **RAG Agent**: Processes user-uploaded documents (PDFs, DOCX, etc.)
- **VM Agent**: Provides permanent knowledge base for market research
- **Report Agent**: Generates structured reports and summaries
- **Charts Agent**: Creates data visualizations using matplotlib
- **Planner Agent**: Decomposes complex queries into multi-agent execution plans
- **Combiner Agent**: Merges results from multiple agents

The workflow uses conditional edges to route between agents based on query type and results.

### 2. State Management
**Question:** How does the system manage state across different agents and what information is shared between them?

**Expected Answer:** The system uses an `AgentState` TypedDict that includes:
- `query`: User's original question
- `route`: Which agent should handle the query
- `final_output`: The response to return to user
- `chat_history`: Conversation context
- `last_agent`: Previously used agent
- `db_results`: Database query results
- `chart_buffer`: Visualization data
- `context_data`: Data passed between agents
- `plan`: Multi-agent execution plan

State flows through the graph, with each agent potentially modifying relevant fields.

### 3. Routing Logic
**Question:** Explain the supervisor agent's routing logic and how it decides which agent to use for a given query.

**Expected Answer:** The supervisor uses a hierarchical rule-based approach:
1. **Rule 0**: If uploaded docs available and query relates to them → RAG agent
2. **Rule 1**: Report-only queries (no new data) → Report agent
3. **Rule 2**: Report with specific data → Multi-agent
4. **Rule 3**: Cross-functional queries → Multi-agent
5. **Rule 4**: Market analysis/research → VM agent
6. **Rule 5**: Raw commodity prices → MSSQL agent
7. **Rule 6**: User/subscription data → MySQL agent
8. **Rule 7**: "How do I" questions → RAG agent
9. **Rule 10**: Conversation references → Previous agent

---

## Database & Data Processing Questions

### 4. SQL Generation
**Question:** How do the MySQL and MSSQL agents generate and execute SQL queries safely?

**Expected Answer:** 
- Both agents use LangChain's SQLDatabase for connection management
- SQL generation uses structured prompts with strict rules:
  - MySQL: Standard MySQL syntax, no backticks
  - MSSQL: T-SQL with mandatory square brackets for all columns/tables
- Reserved keywords are automatically bracketed in MSSQL
- SQL cleaning removes markdown formatting and backticks
- Error handling provides user-friendly messages without exposing SQL details

### 5. Data Visualization
**Question:** How does the charts agent determine what type of chart to generate and what data is suitable for visualization?

**Expected Answer:** 
- Uses LLM to analyze text answers and extract structured data
- Determines chart type based on data characteristics:
  - **Line**: Time-series/trend data
  - **Bar**: Categorical comparisons
  - **Pie**: Distribution/proportion data
- Requires minimum 3 data points for visualization
- Uses matplotlib with custom styling (colors, labels, formatting)
- Returns PNG buffer as BytesIO object

### 6. PandasAI Integration
**Question:** How is PandasAI integrated into the SQL agents and what value does it provide?

**Expected Answer:** 
- After SQL execution, results are converted to pandas DataFrame
- PandasAI processes the DataFrame with the original query
- Provides enhanced analysis beyond basic SQL aggregation
- Results are combined with the original answer as "Enhanced Analysis"
- Handles column name extraction from SQL SELECT clause
- Gracefully handles parsing errors and falls back to original response

---

## RAG and Document Processing Questions

### 7. Document Processing Pipeline
**Question:** Describe the RAG pipeline for processing uploaded documents.

**Expected Answer:** 
- **Loader**: Supports PDF, DOCX, CSV, XLSX using langchain document loaders
- **Chunking**: RecursiveCharacterTextSplitter with 500 chunk size, 100 overlap
- **Metadata**: Adds chunk_id, ingest_time, doc_name to each chunk
- **Embeddings**: Uses sentence-transformers for vector embeddings
- **Vector Store**: ChromaDB for similarity search
- **Chain**: Custom RAG chain for retrieval and generation

### 8. Vector Store Management
**Question:** How does the system manage vector embeddings and similarity search?

**Expected Answer:** 
- ChromaDB as vector store backend
- Sentence-transformers for embedding generation
- Documents are chunked with metadata for context
- Similarity search retrieves relevant chunks
- Results are formatted for LLM consumption with source attribution

---

## Multi-Agent Coordination Questions

### 9. Planning and Execution
**Question:** How does the planner agent decompose complex queries into multi-agent execution plans?

**Expected Answer:** 
- Analyzes query complexity and data source requirements
- Creates JSON execution plan with sequential agent tasks
- Uses dependency injection with `{{prev_result}}` placeholders
- Considers database schemas for accurate task formulation
- Determines visualization needs based on expected output
- Handles schema introspection for dynamic table/column discovery

### 10. Result Combination
**Question:** How does the combiner agent merge results from multiple agents?

**Expected Answer:** 
- Receives results from all executed agents in sequence
- Routes to report agent if query mentions reports
- Routes to chart agent for visualization
- Preserves context and data relationships
- Maintains execution order for dependent queries

---

## Error Handling and Resilience Questions

### 11. LLM Retry Logic
**Question:** How does the system handle LLM failures and ensure reliability?

**Expected Answer:** 
- Uses `invoke_with_fallback` utility for LLM calls
- Implements retry logic with exponential backoff
- Graceful degradation when LLM calls fail
- Provides meaningful error messages to users
- Maintains system state even during failures

### 12. Database Error Handling
**Question:** How are database connection and query errors handled?

**Expected Answer:** 
- Connection testing during initialization
- Try-catch blocks around SQL execution
- User-friendly error messages without exposing SQL details
- Empty result handling with explanatory responses
- Fallback suggestions for query rephrasing

---

## Performance and Scalability Questions

### 13. Memory Management
**Question:** How does the system manage memory and state across conversations?

**Expected Answer:** 
- In-memory state management during session
- Chat history maintained for context
- Limited history (last 10 turns) to prevent memory bloat
- State passed between agents efficiently
- No persistent storage of conversation state

### 14. Concurrent Query Handling
**Question:** How would this system handle multiple concurrent users?

**Expected Answer:** 
- Current implementation is single-user (in-memory state)
- Would need session management for multi-user
- Database connections could be pooled
- Vector store would need user isolation
- Stateless agent design facilitates scaling

---

## Frontend Integration Questions

### 15. React Frontend
**Question:** How does the React frontend integrate with the backend agents?

**Expected Answer:** 
- Uses Vite for development and build tooling
- CopilotKit integration for AI-powered UI
- Axios for HTTP requests to backend
- React Router for navigation
- Chart display through image buffers from backend

---

## Advanced Technical Questions

### 16. LangGraph Workflow Design
**Question:** Why was LangGraph chosen over other orchestration frameworks and what are its advantages?

**Expected Answer:** 
- Native support for conditional routing
- State management built-in
- Visual workflow representation
- Easy debugging and tracing
- Integration with LangChain ecosystem
- Support for complex multi-agent patterns

### 17. Schema Management
**Question:** How does the system handle database schema changes and maintain compatibility?

**Expected Answer:** 
- Dynamic schema introspection using SQLAlchemy
- Schema passed to LLM prompts at runtime
- Column name extraction from queries
- Graceful handling of missing tables/columns
- Schema summaries for planner agent

### 18. Testing Strategy
**Question:** What testing strategies would you implement for this multi-agent system?

**Expected Answer:** 
- Unit tests for individual agents
- Integration tests for agent coordination
- Mock LLM responses for deterministic testing
- Database transaction testing
- End-to-end workflow testing
- Performance testing for concurrent queries

---

## Practical Coding Questions

### 19. Agent Implementation
**Question:** Implement a new agent that fetches weather data and integrates it with commodity pricing analysis.

**Expected Answer:** Should demonstrate:
- Creating new agent node function
- Adding to workflow graph
- State management for new data
- Integration with existing routing logic
- Error handling patterns

### 20. Performance Optimization
**Question:** Identify and optimize performance bottlenecks in the current implementation.

**Expected Answer:** Should identify:
- LLM call latency
- Database query optimization
- Vector store indexing
- Chart generation performance
- Memory usage patterns
- Caching opportunities

---

## System Design Questions

### 21. Scalability Design
**Question:** How would you redesign this system to handle enterprise-scale usage?

**Expected Answer:** Should cover:
- Microservices architecture
- Message queues for agent communication
- Distributed caching
- Load balancing
- Database sharding
- Monitoring and observability

### 22. Security Considerations
**Question:** What security measures should be implemented for production deployment?

**Expected Answer:** Should include:
- Input validation and sanitization
- SQL injection prevention
- Authentication and authorization
- Data encryption
- API rate limiting
- Audit logging

---

## Troubleshooting Questions

### 23. Debugging Multi-Agent Issues
**Question:** A user reports that their query is being routed to the wrong agent. How would you debug this?

**Expected Answer:** Should demonstrate:
- Checking supervisor routing logic
- Analyzing conversation history
- Verifying prompt engineering
- Testing edge cases
- Logging and monitoring

### 24. Performance Issues
**Question:** The system is slow when processing large documents. What would you investigate?

**Expected Answer:** Should cover:
- Document chunking strategy
- Vector store performance
- Embedding generation time
- Memory usage patterns
- Batch processing opportunities

---

This comprehensive question set covers the technical depth of the POC_MAF_X system, from basic architecture to advanced troubleshooting and scalability considerations.
