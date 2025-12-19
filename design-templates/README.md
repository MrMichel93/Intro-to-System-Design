# System Design Templates

Reusable frameworks and patterns for common system design scenarios.

## Reusable Frameworks

Start with these frameworks before diving into specific system designs:

### 1. [Problem Analysis Template](./problem-analysis-template.md)
**Use this first!** Systematically break down any system design problem
- Clarifying questions checklist
- Requirements definition
- Capacity estimation framework
- Use case analysis

### 2. [Architecture Diagram Guide](./architecture-diagram-guide.md)
Learn how to draw clear, effective system diagrams
- Tool recommendations (Excalidraw, draw.io, etc.)
- Standard symbols and notation
- Diagram types and best practices
- Interview tips

### 3. [Component Selection Matrix](./component-selection-matrix.md)
Decision frameworks for choosing the right technologies
- SQL vs NoSQL decision trees
- REST vs GraphQL vs gRPC vs WebSockets
- Load balancer types and algorithms
- Message queue comparison
- Storage options (Object, Block, File)

### 4. [Trade-off Analysis Framework](./trade-off-analysis-framework.md)
Structured approach to evaluating design decisions
- Performance vs Consistency
- Scalability vs Simplicity
- Availability vs Consistency (CAP)
- Build vs Buy
- Real-world examples with analysis

---

## System Design Templates

### 1. [URL Shortener Template](./url-shortener.md)
Learn: Hash generation, database design, caching, redirection

### 2. [Social Media Feed Template](./social-feed.md)
Learn: Timeline generation, fan-out strategies, caching

### 3. [Chat Application Template](./chat-app.md)
Learn: Real-time messaging, WebSockets, message storage

### 4. [E-commerce Platform Template](./ecommerce.md)
Learn: Product catalog, shopping cart, order processing, payments

### 5. [Video Streaming Template](./video-streaming.md)
Learn: Video encoding, CDN, adaptive bitrate, recommendations

### 6. [Search Engine Template](./search-engine.md)
Learn: Web crawling, indexing, ranking, distributed search

## How to Use Templates

### Recommended Workflow

1. **Start with [Problem Analysis](./problem-analysis-template.md)** - Break down the problem systematically
2. **Use [Component Selection Matrix](./component-selection-matrix.md)** - Choose appropriate technologies
3. **Draw diagrams with [Architecture Guide](./architecture-diagram-guide.md)** - Visualize your design
4. **Analyze decisions with [Trade-off Framework](./trade-off-analysis-framework.md)** - Evaluate options
5. **Reference system templates** - Learn from examples (URL Shortener, etc.)
6. **Adapt to your needs** - Customize based on scale and requirements

### General Principles

1. **Start with requirements** - Understand what to build
2. **Follow the template structure** - Proven patterns
3. **Adapt to your needs** - Customize based on scale
4. **Consider trade-offs** - Understand decisions
5. **Practice variations** - Try different constraints

## Template Structure

Each template includes:
- **Problem Statement**
- **Functional Requirements**
- **Non-Functional Requirements**
- **Capacity Estimation**
- **High-Level Architecture**
- **API Design**
- **Database Schema**
- **Key Components**
- **Trade-offs**
- **Variations**

Start with [URL Shortener](./url-shortener.md) as it covers many fundamental concepts!
