# Banking Experience API Design Pattern

## Problems

This design pattern addresses several key challenges in the current banking architecture:

1. **Journey Fragmentation**: Mobile and web applications invoke multiple channel APIs to fulfill a single user journey, creating unnecessary complexity and potential points of failure.

2. **Security Inconsistency**: Varied security controls across different channels lead to potential vulnerabilities and compliance issues.
3. //to do

4. **Business Logic Redundancy**: The same business logic is duplicated across different client channels, causing maintenance overhead and inconsistent implementations.

5. **Incorrect Orchestration Layer**: Business logic orchestration is happening in the channel layer rather than in the Process API layer where it belongs.

6. **Client Complexity**: Clients are responsible for managing multiple API calls and handling errors across varied journeys.

7. **Cross-Channel Consistency**: Difficulty in maintaining consistent user experiences across different client applications.

8. **Backend System Integration**: Complex integration with diverse core banking systems across international operations.

## Context

The bank operates internationally, serving retail and affluent banking segments across various domains including:

- Current Accounts and Savings Accounts (CASA)
- Fixed and Term Deposits
- Investment Products
- Personal and Business Loans
- Credit and Debit Cards
- Domestic and International Payments

The existing architecture has evolved over time, resulting in:

- Multiple client applications (mobile, web, branch terminals) with their own channel-specific APIs
- Diverse core banking systems in different regions, some legacy and some modern
- Siloed development teams responsible for different channels
- A mix of monolithic and microservice architectures
- Ongoing digital transformation efforts to modernize customer experiences

The bank has adopted an API-led connectivity approach with three distinct layers:
1. **Experience APIs** (EAPI): Client-facing APIs designed for specific user experiences
2. **Process APIs** (PAPI): Orchestrating business processes and logic
3. **System APIs** (SAPI): Abstracting underlying system complexities

## Scope

This design pattern focuses specifically on the Experience API layer and its interactions. It covers:

- The design principles and architecture for Experience APIs across banking domains
- Integration patterns between Experience APIs and Process APIs
- Security patterns for client authentication and authorization
- Response aggregation and transformation strategies
- Error handling and resilience patterns

The pattern does not cover:
- Specific implementation technologies or frameworks
- Detailed API specifications or schemas
- Internal Process API or System API designs (though it references them)
- DevOps practices for API deployment and management

## Solution

### Experience API Design Pattern: Journey-Oriented Facade

The proposed design pattern creates a facade that presents cohesive, journey-oriented endpoints to client applications while orchestrating the necessary Process API calls behind the scenes.

#### Architecture Overview

```
┌───────────────┐     ┌───────────────┐     ┌───────────────┐
│ Mobile Client │     │   Web Client  │     │ Other Clients │
└───────┬───────┘     └───────┬───────┘     └───────┬───────┘
        │                     │                     │
        └─────────────┬───────┴─────────────┬───────┘
                      │                     │
           ┌──────────▼─────────┐  ┌────────▼──────────┐
           │ Identity & Access  │  │ API Gateway Layer │
           │    Management     │  │   (Rate limiting,  │
           └──────────┬────────┘  │  Analytics, etc.)  │
                      │           └─────────┬──────────┘
                      │                     │
┌─────────────────────▼─────────────────────▼──────────────────────┐
│                                                                  │
│                         Experience API Layer                     │
│                                                                  │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐         │
│  │ Retail Banking│  │   Investment  │  │    Payment    │   ...   │
│  │  Journey APIs │  │  Journey APIs │  │  Journey APIs │         │
│  └───────┬───────┘  └───────┬───────┘  └───────┬───────┘         │
│          │                  │                  │                  │
├──────────┼──────────────────┼──────────────────┼──────────────────┤
│          │                  │                  │                  │
│  ┌───────▼──────┐   ┌───────▼──────┐   ┌───────▼──────┐           │
│  │Retail Process│   │Investment    │   │Payment       │    ...    │
│  │    APIs      │   │Process APIs  │   │Process APIs  │           │
│  └───────┬──────┘   └───────┬──────┘   └───────┬──────┘           │
│          │                  │                  │                  │
├──────────┼──────────────────┼──────────────────┼──────────────────┤
│          │                  │                  │                  │
│  ┌───────▼──────┐   ┌───────▼──────┐   ┌───────▼──────┐           │
│  │Core Banking  │   │Investment    │   │Payment       │    ...    │
│  │System APIs   │   │System APIs   │   │System APIs   │           │
│  └───────┬──────┘   └───────┬──────┘   └───────┬──────┘           │
│          │                  │                  │                  │
└──────────┼──────────────────┼──────────────────┼──────────────────┘
           │                  │                  │
  ┌────────▼──────┐  ┌────────▼──────┐  ┌────────▼──────┐
  │ Core Banking  │  │  Investment   │  │    Payment    │
  │    Systems    │  │    Systems    │  │    Systems    │
  └───────────────┘  └───────────────┘  └───────────────┘
```

#### Key Components

1. **Journey-Oriented APIs**: Experience APIs are designed around complete user journeys rather than around backend systems or data entities.

2. **Client-Specific Adapters**: Transformation layers that adapt common journeys to specific client needs (mobile, web, etc.) without duplicating business logic.

3. **Security Gateway**: Unified authentication and authorization layer enforcing consistent security policies.

4. **Orchestration Engine**: Coordinates Process API calls to fulfill complete user journeys.

5. **Response Aggregator**: Combines and transforms responses from multiple Process APIs into client-appropriate formats.

6. **Context Manager**: Maintains user context and state across multi-step journeys.

7. **Error Handler**: Provides client-friendly error responses while logging detailed information for troubleshooting.

### Example Implementation

For a "Fund Transfer" journey in retail banking:

```
┌─────────────────┐
│  Mobile Client  │
└────────┬────────┘
         │
         │ Single API call: POST /retail/fund-transfer
         │ {sourceAccount, destAccount, amount, ...}
         ▼
┌─────────────────────────────────────┐
│      Experience API Layer           │
│                                     │
│ 1. Authorize user for transaction   │
│ 2. Transform request                │
│ 3. Orchestrate Process API calls    │
│ 4. Aggregate responses              │
│ 5. Transform final response         │
└───────────────┬─────────────────────┘
                │
    ┌───────────┴────────────┐
    │                        │
    ▼                        ▼
┌─────────────┐      ┌───────────────┐
│ Account     │      │ Transaction   │
│ Process API │      │ Process API   │
└─────┬───────┘      └───────┬───────┘
      │                      │
      ▼                      ▼
┌─────────────┐      ┌───────────────┐
│ Account     │      │ Transaction   │
│ System API  │      │ System API    │
└─────────────┘      └───────────────┘
```

## Drivers

1. **Customer-Centricity**: Focus on delivering seamless customer journeys rather than exposing system architectures.

2. **Reduced Time-to-Market**: Enabling faster development of new features across channels by reducing redundant implementations.

3. **Consistency**: Ensuring consistent user experiences regardless of the client application used.

4. **Security**: Implementing uniform security controls across all client interactions.

5. **Scalability**: Creating an architecture that can scale with growing customer base and expanding geographical footprint.

6. **Technology Evolution**: Allowing for backend system modernization without disrupting client experiences.

7. **Omni-Channel Strategy**: Supporting the bank's strategy to provide seamless experiences across digital and physical channels.

8. **Regulatory Compliance**: Meeting financial regulations that may vary across different markets.

## Controls

1. **Authentication & Authorization**
   - OAuth 2.0/OpenID Connect implementation for consistent authentication
   - Role-Based Access Control (RBAC) with fine-grained permissions
   - Multi-factor authentication integration
   - Consent management for customer data usage

2. **Request Validation & Sanitization**
   - Schema validation for all incoming requests
   - Input sanitization to prevent injection attacks
   - Business rule validation before orchestration

3. **Traffic Management**
   - Rate limiting based on client application and user tiers
   - Circuit breakers to prevent cascade failures
   - Request prioritization for critical journeys
   - Throttling controls for high-volume operations

4. **Monitoring & Observability**
   - End-to-end transaction tracing
   - Performance metrics collection
   - Error rate monitoring
   - Journey completion analytics

5. **Data Controls**
   - Field-level encryption for sensitive data
   - Data minimization in responses
   - Masking of PII in logs and responses
   - Compliance with data residency requirements

## Instructions

### Design Principles for Experience APIs

1. **Journey-First Approach**
   - Design APIs around complete user journeys, not system functions
   - Name resources based on customer understanding, not internal systems
   - Example: `/retail/account-opening` instead of `/create-customer-record-and-account`

2. **Consistent Patterns**
   - Maintain consistent URL structures across domains
   - Standardize error responses
   - Use consistent versioning strategies
   - Example pattern: `/{segment}/{journey-name}/{resource}`

3. **Client-Appropriate Responses**
   - Return only the data needed by the client
   - Structure responses to minimize client-side processing
   - Include hypermedia links for journey continuation where appropriate

4. **Versioning Strategy**
   - Use URL versioning for major changes: `/v1/retail/accounts`
   - Use content negotiation for minor variations: `Accept: application/vnd.bank.account.v2+json`

### Implementation Guidelines

1. **Journey Mapping**
   - Document complete user journeys across channels
   - Identify touchpoints and data requirements
   - Map journeys to required Process APIs

2. **API Design**
   - Create OpenAPI specifications for each Experience API
   - Define clear request/response schemas
   - Document error scenarios and codes

3. **Security Implementation**
   - Integrate with the bank's IAM solution
   - Implement token validation and scope checking
   - Apply appropriate rate limiting

4. **Orchestration Logic**
   - Implement asynchronous processing for long-running journeys
   - Use correlation IDs for request tracing
   - Implement compensation logic for partial failures

5. **Testing Strategy**
   - Create journey-based test scenarios
   - Implement contract testing between Experience and Process APIs
   - Perform security testing and penetration testing

## Relevant Industry Best Practices and Regulations

1. **Open Banking Standards**
   - Align with Open Banking standards for common banking functions
   - Follow standard security profiles from the Financial-grade API (FAPI)
   - Reference: [Open Banking Implementation Entity](https://www.openbanking.org.uk/)

2. **PSD2 Compliance** (for European operations)
   - Implement Strong Customer Authentication (SCA)
   - Support Transaction Risk Analysis (TRA)
   - Maintain fraud monitoring capabilities

3. **API Design Best Practices**
   - Follow REST maturity model level 2 or 3
   - Implement HATEOAS for complex journeys
   - Use standard HTTP status codes appropriately

4. **Security Standards**
   - OWASP API Security Top 10 compliance
   - OAuth 2.0 and OpenID Connect implementations
   - Transport Layer Security (TLS 1.2+)

5. **Banking Industry Standards**
   - ISO 20022 for payment message formats
   - BIAN (Banking Industry Architecture Network) service domains
   - ISO 8583 for card transactions (where applicable)

## Pattern Control Domain

The Experience API pattern operates within the "Customer Experience Domain" of the banking architecture. This domain:

1. **Owns**:
   - Client-facing API contracts
   - User journey definitions
   - Channel-specific adaptations
   - API documentation for developers

2. **Coordinates with**:
   - Security and Identity domain
   - Business Process domain
   - Data Governance domain
   - Regulatory Compliance domain

3. **Governed by**:
   - API Governance Council
   - Digital Experience Committee
   - Enterprise Architecture team
   - Information Security Office

## Pattern Layer

This pattern applies specifically to the Experience API Layer of the three-tiered API-led connectivity approach:

1. **Position**: Top layer (client-facing)
2. **Interfaces with**:
   - Client applications (above)
   - Process API layer (below)
3. **Characteristics**:
   - High volatility (changes frequently with UI/UX)
   - Client-oriented semantics
   - Coarse-grained operations
   - Journey-focused resources

## Rationale

This Experience API design pattern is the optimal approach for the bank's modernization for several reasons:

1. **Decoupling of Concerns**: Separates client experience logic from business process logic, allowing teams to evolve independently.

2. **Reduced Complexity for Clients**: Client applications make fewer API calls, reducing development and maintenance costs.

3. **Simplified Evolution**: Backend systems can be modernized without requiring client application changes.

4. **Consistent Security**: Centralizes authentication and authorization, reducing the risk of security vulnerabilities.

5. **Improved Performance**: Reduces network round-trips between client and server by handling orchestration server-side.

6. **Future-Proofing**: Creates an abstraction layer that can accommodate future channels (voice, conversational UI, etc.).

7. **Specialized Optimization**: Allows for channel-specific optimizations without duplicating business logic.

8. **Cross-functional Alignment**: Creates a clear boundary between channel teams and backend system teams.

## Consequences

### Positive Consequences

1. **Streamlined User Journeys**: Customers experience smoother interactions across channels.

2. **Faster Time to Market**: New features can be developed and deployed more quickly.

3. **Consistent Experiences**: Users encounter the same behavior regardless of channel.

4. **Enhanced Security**: Security controls are applied consistently across all channels.

5. **Reduced Redundancy**: Elimination of duplicate business logic across channels.

6. **Better Analytics**: Journey-based APIs allow for better tracking of user behavior.

7. **Simplified Client Development**: Mobile and web developers work with simpler, more focused APIs.

### Negative Consequences

1. **Additional Architectural Layer**: Introduces another layer that needs to be maintained.

2. **Potential Performance Overhead**: Additional processing for orchestration and transformation.

3. **Governance Challenges**: Requires strong API governance to prevent drift and inconsistency.

4. **Development Complexity**: Teams need to understand journey-based design rather than system-based design.

5. **Initial Investment**: Requires upfront effort to design and implement the pattern across domains.

## Related Patterns

1. **API Gateway Pattern**: Complements the Experience API by providing cross-cutting concerns like security, monitoring, and traffic management.

2. **Backend for Frontend (BFF)**: A specialized variation where Experience APIs are tailored specifically for individual client types.

3. **Aggregator Pattern**: Used within Experience APIs to combine data from multiple Process APIs.

4. **Command Query Responsibility Segregation (CQRS)**: Often used in Process APIs that are orchestrated by Experience APIs.

5. **Strangler Pattern**: Used when gradually migrating from legacy direct channel APIs to the new Experience API pattern.

6. **Circuit Breaker Pattern**: Implemented within Experience APIs to handle failures in downstream Process APIs.

7. **Saga Pattern**: Used for implementing complex, multi-step journeys that require transaction management.

8. **Event-Driven Architecture**: Complementary pattern for real-time updates and notifications through Experience APIs.

## Responsibilities of Experience API

### 1. Authorization

The Experience API is responsible for ensuring the right resources are accessed by the right users:

- **Token Validation**: Validate OAuth tokens or session tokens
- **Permission Checking**: Verify user has appropriate permissions for the requested journey
- **Contextual Authorization**: Consider factors like device, location, and risk score
- **Delegation of Authority**: Handle scenarios where one user acts on behalf of another
- **Consent Management**: Enforce user consent choices for data usage

**Implementation Strategy**:
- Integrate with the bank's Identity and Access Management (IAM) system
- Extract user claims from authentication tokens
- Apply fine-grained authorization rules based on user segments and journey sensitivity
- Maintain authorization decision audit logs

### 2. Request Transformation

Experience APIs transform client requests to match Process API requirements:

- **Format Conversion**: Transform request formats from client-friendly to process-oriented
- **Data Enrichment**: Add context information not provided by the client
- **Validation Enhancement**: Apply additional validation rules beyond basic schema validation
- **Terminology Translation**: Convert client terminology to internal business terminology
- **Channel-Specific Processing**: Apply channel-specific business rules

**Implementation Strategy**:
- Define clear mapping rules between Experience API and Process API schemas
- Use transformation templates for consistent processing
- Implement adapter patterns for different client types
- Generate transformation code from schema definitions where possible

### 3. Orchestration

Experience APIs orchestrate multiple Process API calls to fulfill complete journeys:

- **Process Flow Management**: Determine the sequence of Process API calls
- **Conditional Processing**: Execute different flows based on request data or user context
- **Parallel Execution**: Call multiple Process APIs simultaneously when possible
- **Error Handling**: Manage errors from individual Process APIs
- **Compensation Logic**: Handle rollback of partial transactions when needed
- **State Management**: Maintain state for multi-step journeys

**Implementation Strategy**:
- Implement workflow patterns using orchestration engines or state machines
- Use correlation IDs to track requests across Process APIs
- Apply circuit breaker patterns for resilience
- Implement idempotency for retry scenarios

### 4. Response Aggregation

Experience APIs aggregate and transform responses from multiple Process APIs:

- **Data Consolidation**: Combine data from multiple sources into a cohesive response
- **Response Filtering**: Remove unnecessary fields based on client needs
- **Format Transformation**: Convert internal data formats to client-friendly formats
- **Data Enrichment**: Add derived or calculated fields
- **Hypermedia Addition**: Add links to related resources or next steps in the journey

**Implementation Strategy**:
- Define clear response schemas for each journey
- Implement intelligent field filtering based on client context
- Use response templates for consistent formatting
- Apply response caching where appropriate

### 5. Attribute Minimization

Experience APIs should return only the required attributes:

- **Data Minimization**: Return only the data needed by the client application
- **Privacy Protection**: Limit exposure of sensitive personal information
- **Performance Optimization**: Reduce payload sizes for better network performance
- **Field Selection**: Support client-driven field selection when appropriate
- **Tiered Information**: Provide basic information with links to detailed resources

**Implementation Strategy**:
- Define "view models" specific to client needs
- Implement field filtering based on authorization context
- Support query parameters for field selection
- Create journey-specific response projections
