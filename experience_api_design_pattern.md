
# Experience API Design Pattern for Modernized Banking Architecture

## 1. Problems

The Experience API design pattern addresses the following challenges in a modern international banking architecture:

- **Redundant Business Logic**: Business logic is duplicated across multiple client channels (mobile/web), increasing complexity and maintenance overhead.
- **Channel-Centric Orchestration**: Each client manages its own orchestration, resulting in inconsistent implementations and duplicated efforts.
- **Multiple API Invocations**: User journeys often require invoking multiple backend APIs individually, impacting performance and UX.
- **Security Gaps**: Inconsistent implementation of authorization and access control across different channels.
- **Lack of Abstraction**: Clients are tightly coupled with process/system APIs, increasing the risk of breaking changes and reducing flexibility.
- **Scalability and Performance**: Repetitive orchestration at the channel level results in latency and scalability issues.

## 2. Context

The bank operates globally across multiple markets and serves both **retail and affluent** customers across domains like **CASA, term deposits, investments, loans, cards, and payments**. The core banking backend is heterogeneous and varies across regions and product lines. The bank is undergoing modernization and intends to **align with API-led connectivity principles** (Experience, Process, and System APIs).

//todo

## 3. Scope

This pattern applies to the **Experience API layer** within an API-led architecture. It focuses on:

- **Channel-agnostic orchestration** of user journeys.
- **Abstracting client interfaces** from process and system APIs.
- **Serving mobile and web channels** with unified APIs.
- **Improving maintainability and agility** of client applications.

This pattern does not cover:

- Design of underlying system APIs.
- Non-RESTful interfaces like event-driven APIs or gRPC.

## 4. Solution

Introduce a centralized **Experience API (EAPI)** layer to:

- Expose **unified, channel-agnostic APIs** to clients.
- **Transform requests** to suit multiple process APIs.
- **Orchestrate** calls across domain APIs for complex journeys (e.g., Apply for a credit card + CASA account).
- **Aggregate and shape responses** tailored for client consumption.
- **Implement caching and filtering** to improve performance and UX.

### Responsibilities of EAPI

1. **Authorization**: Ensures the right user accesses the right resource using token-based mechanisms.
2. **Request Transformation**: Adapts client input formats to match process API requirements.
3. **Orchestration**: Sequences multiple process APIs to complete a user journey.
4. **Response Aggregation**: Combines results from multiple process APIs into a single cohesive response.
5. **Attribute Filtering**: Filters out unnecessary fields to return only required attributes to clients.
6. **User-Level Distributed Caching**: Caches personalized data per user with TTL to reduce latency.

## 5. Drivers

- Reduce development effort and code duplication across clients.
- Improve consistency, maintainability, and scalability of APIs.
- Accelerate time-to-market for new user journeys.
- Enhance security and governance in client communication.
- Enable channel-specific flexibility (e.g., shaping data for mobile vs. web).

## 6. Controls

- **API Gateway Integration**: Enforce authentication, rate-limiting, IP whitelisting.
- **Role-based Access Control (RBAC)**: Apply OAuth 2.0 scopes per user or role.
- **Caching Strategy**: TTL-based per-user cache to reduce backend dependency for frequently accessed data.
- **Response Contracts**: Define strict schemas using OpenAPI/Swagger to standardize response formats.
- **Telemetry & Monitoring**: Real-time monitoring, logging, and tracing for observability.
- **Service Mesh Policies**: Secure service-to-service communication.

## 7. Instructions

- Design each EAPI around **user journeys**, not backend capabilities.
- Use OpenAPI spec for documenting EAPI contracts.
- Keep EAPI stateless where possible; leverage cache where needed.
- Align naming conventions and URI standards with internal API governance.
- Integrate feature toggles and market-specific configuration.

## 8. Relevant Industry Best Practices

- **BIAN (Banking Industry Architecture Network)** service domains for aligning functional boundaries.
- **PSD2 / Open Banking** for consent and access standards.
- **OWASP API Security Top 10** for protecting APIs.
- **RESTful API Design Guidelines** (Microsoft, Google, etc.) for endpoint consistency.

## 9. Pattern Control Domain

This pattern operates within the **API Management and Orchestration** control domain, aligning client interfaces with internal capabilities.

## 10. Pattern Layer

- Applies to the **Experience API layer**, sitting between clients and process APIs.
- Acts as a consumer-facing orchestration layer.

## 11. Rationale

This pattern ensures channel-agnostic, scalable, and secure delivery of user journeys. By abstracting backend complexity and concentrating orchestration logic, the bank can improve agility and customer experience while simplifying client app development.

## 12. Consequences

**Pros**:

- Centralized orchestration logic.
- Better user experience and performance.
- Streamlined governance and monitoring.

**Cons**:

- Adds an additional layer of complexity.
- Requires well-defined process APIs underneath.
- Potential performance impact if caching is not managed properly.

## 13. Related Patterns

- **Process API Pattern**: Handles business logic and workflow-level orchestration.
- **System API Pattern**: Direct integration with core banking systems.
- **API Gateway Pattern**: Enforces policies and authentication at the edge.
- **Façade Pattern**: Abstracts and simplifies client-facing interfaces.

---

**End of Document**
